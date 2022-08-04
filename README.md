# Frontend CICD Setup 

## AWS Setup
1. Creation of S3 Bucket
2. Creation of Cloud Front 
3. Creation of IAM deployer Role

## Node JS
4. Creation of Node JS scripts

## Github
5. Creation of github work flow

<br> 


# AWS
## S3 Bucket
Create the new bucket (click blue rectangle): 
![Create bucket image](/images/s3/create_bucket.png)

Fill in the `Bucket name` and `AWS Region`:
![Name and region image](/images/s3/name_and_region.png)

Select your `Object Ownership` and set your `Block Public Access settings for this bucket`

If ACLs are disabled then only resources from this account can contribute to this bucket. If you are expecting external resources to directly contribute to the bucket then we would want to enable ACLs. Since we only want one `Deployer` user to be the one contributing to this bucket (i.e. pushing up the built react projects via github actions) we want to select ACLs `disabled`.

Since we are going to be hosting this site from CloudFront, we don't want to have any public access to our S3 bucekt so we will select `Block all public access`

![Object ownership image](/images/s3/object_ownership_acls.png)

For the rest of the settings you can leave the default preset or change them as you see fit. But for this example we will click `Create bucket` with the default set values. 
![Final create s3 button](/images/s3/final_create_s3_button.png)

You should be able to verify that your new created S3 bucket has been added to the list:
![S3 Confirm](/images/s3/confirm.png)

<br>

## Cloudfront
Let's navigate over to the `Cloudfront` service using the top search bar:

![cloudfront navigation](/images/cloudfront/cloudfront_navigation.png)


Create a new `Distribution`: 
![create new distribution](/images/cloudfront/create_cloudfront.png)

Select the S3 bucket created from the previous step as the `Origin domain`:
![select domain](/images/cloudfront/select_s3_bucket.png)

Set up the OAI settings as shown below and click `Create new OAI` or use an existing identity. In the example we will create a new OAI:
![OAI setup](/images/cloudfront/select_OIA.png)

Create new OAI and name it as you please. We will leave the default name in this example: 
![OAI Creation](/images/cloudfront/create_OIA.png)

There are a ton of different settings available, but we will leave all the default settings except for the `Default root object` field. We will fill it with `index.html` or whatever file the build is planning to use as the root. For our example and most Javascript frameworks the finalized code will have an index.html be an entry point. The click `Create distribution` to continue.
![Distribution finalization](/images/cloudfront/finish_distribution.png)

We can verify that these changes took effect by navigating to the S3 bucket created and viewing the updates to it's policy:

Navigate back to S3 via the top search bar
![verify 1](/images/cloudfront/verify1.png)

Select the s3 bucket we created earlier
![verify 2](/images/cloudfront/verify2.png)

Click on `Permissions`
![verify 3](/images/cloudfront/verify3.png)

Scroll down and verify that a similar policy has been put in place on the S3 bucket
![verify 4](/images/cloudfront/verify4.png)

## IAM deployer role

Navigate to the `IAM` service.
![IAM navigation](/images/iam/iam_navigation.png)

Select `Users`
![IAM users navigation](/images/iam/iam_users.png)

Select `Add users`
![IAM add users](/images/iam/iam_add_user.png)

Follow the flow of creation of a `deployer` user as demonstrated in the images below: 

Create a user name
![Deployer setup 1](/images/iam/deployer1.png)

Leave the permissions blank. Click `Next`
![Deployer setup 2](/images/iam/deployer2.png)

Click `Next`
![Deployer setup 3](/images/iam/deployer3.png)

Click `Create user`
![Deployer setup 4](/images/iam/deployer4.png)

*Remember we want this user to have as little permissions available as possible, so we only give it the permissions that is needed per AWS best practices*

Download the CSV file for these credentials. Or copy and paste it. Be sure to keep these on hand. They will be very important in the deployment process. 
![Deployer setup 5](/images/iam/deployer5.png)

Select your newly created User
![Deployer setup 6](/images/iam/deployer6.png)

In the permissions tab click `Add inline policy`
![Deployer setup 7](/images/iam/real_inline.png)

Make sure you are in the `JSON` tab view
![Deployer setup 9](/images/iam/deployer9.png)

Copy and paste the following policy below *WITH YOUR BUCKET ARN* into the json text field `"arn:aws:s3:::interact-example-bucket"` and `"arn:aws:s3:::interact-example-bucket/*"`. Then click `Next`. Instructions finding the Bucket ARN are down below.
![Deployer setup 11](/images/iam/deployer11.png)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::interact-example-bucket",
                "arn:aws:s3:::interact-example-bucket/*"
            ]
        }
    ]
}
```

*Navigate to S3 > Select your bucket > Properties and you will see the Bucket ARN*

![Deployer setup 10](/images/iam/deployer10.png)
(The ARN location in the current example)

ARN stands for `Amazon Resource Name` and this is how services in the account identify each other. Here this policy states that the deployer user we created will have PUT permissions for the bucket created. This is what we want because the deployer user credentials (CSV file we downloaded before) will be used to place updated builds into the bucket hosting our application


Keep clicking `Next` 
![Deployer setup 12](/images/iam/deployer12.png)

Fill out the name and descriptions as you please then click `Create policy`
![Deployer setup 13](/images/iam/deployer13.png)

Upon completion you should see your policy right here. 
![Deployer setup 14](/images/iam/deployer14.png)

Let's add the same policy to our S3 bucket so it is aware that our deployer has the PUT rights and can add objects to the bucket. In it's current state only the `CloudFront` Distribution we created earlier has READ access to it's content. Follow the images below to complete this final step. 

Navigate to S3 and select the bucket we created earlier
![S3 IAM setup 1](/images/s3/iam_s3_1.png)

Select `Permissions`
![S3 IAM setup 2](/images/s3/iam_s3_2.png)

Scroll down and select `Edit` on the `Bucket policy`
![S3 IAM setup 3](/images/s3/iam_s3_3.png)

Copy and paste the JSON below *WITH YOUR ARN* in the `Principal` attribute (i.e. replace `arn:aws:iam::908064770691:user/example-deployer` with your ARN). Instructions on finding your deployer ARN will be below the JSON.
![S3 IAM setup 4](/images/s3/iam_s3_4.png)

*Don't forget to properly format the JSON (adding commas, etc). There will be red X's if there are any syntax errors*

```
	{
		    "Sid": "2",
            "Effect": "Allow",
            "Principal": {"AWS":"arn:aws:iam::908064770691:user/example-deployer"},
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::interact-example-bucket",
                "arn:aws:s3:::interact-example-bucket/*"
            ]
        }
```

*Finding your user ARN: Navigate to IAM > Users > Select your user > You should see the ARN like the image above*
![S3 IAM setup 5](/images/s3/iam_s3_5.png)

Now click `Save changes` and we are done with AWS setup.
![S3 IAM setup 6](/images/s3/iam_s3_6.png)


# NodeJS
## Setup of the scripts 

We assume the git repo is already apart of the Interact Suite of projects and there is an admin access to the AWS account (similar to the CSV we downloaded but not just for the deployer, but for the whole account).

We only need to modify one part of the code to enable this deployment: 

![node script](/images/node/scripts.png)

We should add these two scripts to our `package.json` which will be used in the `Github` actions workflow. 

```
"deploy": "aws s3 sync build/ s3://interact-example-bucket"
```
*This script uses the AWS CLI tool and basically will take the content of the `build` folder and sync it up with the content of the s3 bucket provided. You would simply use the name of the bucket we created earlier. Replace `interact-example-bucket` with the name of your bucket and add it to the `scripts` object.*

```
"invalidate": "aws cloudfront create-invalidation --distribution-id E2E0C1XE77EVJ8 --paths /index.html"
```
*The same can be said here for the distribution. Basically we want to make sure the Distribution gets our latest changes so we invalidate the root `/index.html` which should have the bulk of our changes. This can be modified/catered to specific applications dealing with images, videos, etc. Also note that the `distribution-id` should match the new Distribution we created earlier (view image for reference)*

![dist-id](/images/node/distribution-id.png)


# Github workflow
## Setup ENV variables in secrets

Go to `Settings`
![Settings](/images/node/settings.png)

Go to `Secrets`
![Secrets](/images/node/secrets.png)

Click actions
![Actions](/images/node/actions.png)

Click `New repository secret`
![new_secret](/images/node/new_repo_secrets.png)

Create a new Secret for your Deployer *AND* your admin account. There needs to be one for the `Access Key Id` and one for the `Secret Access Key`. A total of 4 keys should be created. 2 for deployer and 2 for the admin account. Note the admin account can be reused to fire off the distribution cache invalidation call. Example image below

![deployer_secrets](/images/node/deployer_secrets.png)

## Build the Action

Navigate back to `Actions`
![back to actions](/images/node/back_to_Actions.png)

Click new workflow
![new work flow](/images/node/new_workflow.png)

Select the node base
![select Node](/images/node/select_node.png)

Copy and paste the script below and get rid of all the code in the free text. Change the secret values to your specific secret values and name the action whatever suits the project (prod vs dev etc). 
![secret values](/images/node/copy_paste.png)

```
name: Example Interact CICD

on:
   workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:

  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build
    - name: Set AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with: 
         aws-access-key-id: ${{ secrets.EXAMPLE_DEPLOYER_ID }}
         aws-secret-access-key: ${{ secrets.EXAMPLE_DEPLOYER_SECRET }}
         aws-region: us-west-1
    - run: npm run deploy
    - name: Set AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with: 
         aws-access-key-id: ${{ secrets.AWS_INVALIDATE_KEY_ID }}
         aws-secret-access-key: ${{ secrets.AWS_INVALIDATE_ACCESS_KEY }}
         aws-region: us-west-1
    - run: npm run invalidate
```
Commit to the code and you should be able to test and verify your changes via your distribution URL:

![commit](/images/node/commit_new.png)

Where to find the distribution URL:
![commit](/images/node/dist-url.png)


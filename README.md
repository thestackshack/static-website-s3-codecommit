# Static Website, S3, SSL, CodeCommit

The template: [cloudformation.json](cloudformation.json)

## What AWS resources does this template use?
* S3 (File storage & static website)
* CloudFront (HTTPS & caching)
* Certificate Manager (SSL Cert)
* Route53 (DNS)
* CodeBuild (Continuous deployment)
* CodePipeline (Continuous deployment)
* CodeCommit (GIT repo)
* CloudFormation (Infrastructure as Code)
* IAM (AWS permissions & users)

## Validate domain ownership
When AWS Certificate Manager creates your SSL certificate it sends and email to the domain name administrator (You).  You have to get that email and click the confirmation link.  Once this step has been completed the stack can continue being built.  More information can be found here:  http://docs.aws.amazon.com/acm/latest/userguide/gs-acm-validate.html

## What do I do after my stack has been created?
Use your CodeCommit repo and your CodeCommit credentials to start checking in your static site.

CodePipeline and CodeBuild are setup to continuously deploy your site as you push changes to your repo.  

Branches and CodePipeline actions:
**master** -> Pushes changes to APEX <example.com> (yes ssl)
**develop** -> Pushes changes to dev subdomain (no ssl)

## What should the structure of my project be?
Great question.  You need to add a buildspec.yml so CodeBuid knows how to build your project.  

* /www/assets/ -> Put all js, css, images, etc... in this folder.
* /www/index.html -> Put all html files in this folder.
* buildspec.yml -> CodeBuild uses this to build your project.

buildspec.yml
```
version: 0.1
phases:
  install:
    commands:
  pre_build:
    commands:
  build:
    commands:
      - aws s3 sync www "s3://${BUCKET_NAME}" --acl bucket-owner-full-control --acl public-read --delete --cache-control "max-age=1" --exclude www/assets
      - aws s3 sync www/assets "s3://${BUCKET_NAME}/assets" --acl bucket-owner-full-control --acl public-read --delete --cache-control "max-age=31536000"
  post_build:
    commands:
```

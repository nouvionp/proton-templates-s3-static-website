# Proton templates for creating and maintaining S3 static websites

This template is meant to help quickly create, configure and deploy a static website on AWS S3 using AWS Proton.
This code is heavily inspired from the static website CloudFormation template from: https://www.coletiv.com/blog/how-to-use-aws-cloud-formation-to-setup-the-infrastructure-for-a-static-website/

The environment template will be responsible for creating the S3 bucket with the configuration required (IAM roles, recommended security settings, Etc).

The service template will be responsible for creating the pipeline that will deploy your code build output to the S3 bucket.

## TODOs

1. Add CloudFront setup
2. Add custom domain name as an option

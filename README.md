# Proton templates for creating and maintaining S3 static websites

This template is meant to help quickly create, configure and deploy a static website on AWS S3 using AWS Proton.
This code is heavily inspired from the static website CloudFormation template from: https://www.coletiv.com/blog/how-to-use-aws-cloud-formation-to-setup-the-infrastructure-for-a-static-website/

The environment template will be responsible for creating the S3 bucket with the configuration required (IAM roles, recommended security settings, Etc).

The service template will be responsible for creating the pipeline that will deploy your code build output to the S3 bucket.

## TODOs

1. Add CloudFront setup
2. Add custom domain name as an option

## CLI Helpers

### Environment template

Having to create a new template version for testing is time consuming, template sync is nice but when building from scratch is still a bit slow to quickly debug and iterate. Below is a set of CLI commands aiming to make updating a version of a template easier/faster.

#### Setup

First let's set some vars to customize the CLI for your project.

```bash
export PROTON_S3_BUCKET=yokailabs-proton-templates
export PROTON_TEMPLATE_NAME=ssw-environment
```

```bash
# CD into template directory
cd ~/Projects/proton-templates-s3-static-website

# Create Archive to be uploaded to S3
tar -czvf $PROTON_TEMPLATE_NAME.tar.gz $PROTON_TEMPLATE_NAME/v1

# Upload to S3
aws s3 cp $PROTON_TEMPLATE_NAME.tar.gz s3://$PROTON_S3_BUCKET/$PROTON_TEMPLATE_NAME.tar.gz

# Delete version 1.0 as you are iterating and developing your template ...
aws proton delete-environment-template-version \
    --template-name "$PROTON_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0"

# Update and publish new proton environment template minor version
aws proton create-environment-template-version \
    --template-name "$PROTON_TEMPLATE_NAME" \
    --source s3="{bucket=$PROTON_S3_BUCKET, key=$PROTON_TEMPLATE_NAME.tar.gz}"

aws proton update-environment-template-version \
    --template-name "$PROTON_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0" \
    --status "PUBLISHED"

# Cleanup step - Delete the archive
rm -f $PROTON_TEMPLATE_NAME.tar.gz
```
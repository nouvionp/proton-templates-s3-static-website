# Proton templates for creating and maintaining S3 static websites

This template is meant to help quickly create, configure and deploy a static website on AWS S3 using AWS Proton.
This code is heavily inspired from the static website CloudFormation template from: https://www.coletiv.com/blog/how-to-use-aws-cloud-formation-to-setup-the-infrastructure-for-a-static-website/

## TODOs

1. Add custom domain name as an option

## CLI Helpers

Having to create a new template version for testing is time consuming, template sync is nice but when building from scratch is still a bit slow to quickly debug and iterate. Below are a set of instructions and CLI commands aiming to make updating/testing a version of a template easier/faster.

### Global setup

Define your template bucket and other vars once per session or save it in your shell config (.bashrc, .zshrc) or maybe use a .env file.

```bash
export PROTON_S3_BUCKET=yokailabs-proton-templates
export PROTON_ENV_TEMPLATE_NAME=ssw-environment
export PROTON_SVC_TEMPLATE_NAME=ssw-service
```

Then change directory to your template directory

```bash
cd ~/Projects/proton-templates-s3-static-website
```

### Environment template

```bash
# Create Archive to be uploaded to S3
tar -czvf $PROTON_ENV_TEMPLATE_NAME.tar.gz $PROTON_ENV_TEMPLATE_NAME/v1

# Upload to S3
aws s3 cp $PROTON_ENV_TEMPLATE_NAME.tar.gz s3://$PROTON_S3_BUCKET/$PROTON_ENV_TEMPLATE_NAME.tar.gz

# Delete version 1.0 as you are iterating and developing your template ...
aws proton delete-environment-template-version \
    --template-name "$PROTON_ENV_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0"

# Update and publish new proton environment template minor version
aws proton create-environment-template-version \
    --template-name "$PROTON_ENV_TEMPLATE_NAME" \
    --source s3="{bucket=$PROTON_S3_BUCKET, key=$PROTON_ENV_TEMPLATE_NAME.tar.gz}"

aws proton update-environment-template-version \
    --template-name "$PROTON_ENV_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0" \
    --status "PUBLISHED"

# Cleanup step - Delete the archive
rm -f $PROTON_ENV_TEMPLATE_NAME.tar.gz
```

### Service template

```bash
rm -f $PROTON_SVC_TEMPLATE_NAME.tar.gz
tar -czvf $PROTON_SVC_TEMPLATE_NAME.tar.gz $PROTON_SVC_TEMPLATE_NAME/v1
aws s3 cp $PROTON_SVC_TEMPLATE_NAME.tar.gz s3://$PROTON_S3_BUCKET/$PROTON_SVC_TEMPLATE_NAME.tar.gz

aws proton delete-service-template-version \
    --template-name "$PROTON_SVC_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0"

aws proton create-service-template-version \
    --template-name "$PROTON_SVC_TEMPLATE_NAME" \
    --compatible-environment-templates '[{"templateName":"'$PROTON_ENV_TEMPLATE_NAME'","majorVersion":"1" }]' \
    --source s3="{bucket=$PROTON_S3_BUCKET, key=$PROTON_SVC_TEMPLATE_NAME.tar.gz}"

aws proton update-service-template-version \
    --template-name "$PROTON_SVC_TEMPLATE_NAME" \
    --major-version "1" \
    --minor-version "0" \
    --status "PUBLISHED"
```

### Security - Post setup steps.

* After you've created your service instances (websites) make sure to edit the S3 bucket and block all public access, we're leaving public by default in the CloudFormation template as otherwise we would not be able to apply the S3 policy.
* See: 'CloudFormation s3:PutBucketPolicy Access Denied'

### Notes

You will have to wait until the pipeline has completed its first build and until cloudfront was fully deployed before being able to load your site using the CloudfrontUrl.
i,e: 'CloudfrontUrl	d1geeahx8q8jj3.cloudfront.net/index.html'

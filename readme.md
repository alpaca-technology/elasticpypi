elastic pypi
------------

A fully functional, self-hosted  simple pypi service running on AWS.

# Caveats

**Cannot browse with a browser**

This is a limitation of current browsers. They have removed basic authentication for remote urls via the url (e.g. x:y@z). `WWW-Authenticate` responses do not work with AWS Lambda.

# Setup

1. Edit `serverless.yml`

## Configuration

### serverless.yml

```
service: elasticpypi

provider:
  name: aws
  runtime: python3.6
  memorySize: 128
  stage: dev
  # profile: "some-local-aws-config-profile"
  # region: us-east-1

  environment:
    SERVICE: ${self:service}          # See above. Defaults to elasticpypi
    STAGE: "/${self:provider.stage}"  # See above. Defaults to dev
    BUCKET: "elasticpypi"             # CHANGE ME
    TABLE: "elasticpypi"              # You can change me if you want, but do you?
    USERNAME: "elasticpypi"           # You can change me if you want, but do you?
    PASSWORD: "something-secretive"   # CHANGE ME
    OVERWRITE: false                  # Allow uploads to overwrite already existing packages
```

# Deploy

`npm`/`yarn` and `pip` are required to install the necessary packages to deploy.

1. `yarn` or `npm install`
1. `sls deploy`


**Note** that when deploying do not have the virtualenv activated. The `wsgi` plugin for serverless will automatically fetch the python requirements.

# Using

Based on the output of the deploy command or via the AWS console add the url to your pip conf.

The url should be something like `https://blah.execute-api.region.amazonaws.com/dev/simple`.

Make sure you add a trailing slash as required in the PEP.

Make sure you add your basic authentication credentials to your url.

## Throttling

AWS resources maybe throttled. As such, if you are intending to dump a bunch of packages into the S3 bucket check your
service and account limits. Additionally, changing the read and write capacity of dynamodb may help. It is currently set
to the lowest possible unit (1).

# Testing

## Requirements

1. Install testing requirements from `test-requirements.txt`
1. Run `python -m pytest`

## Using Docker

The example below runs the full test suite. To debug, add `/bin/bash` to the end of the command.

    $ sudo docker build -t elasticpypi-test .
    $ sudo docker run -it \
        -v $(pwd):/code \
        elasticpypi-test

# Changelog

* 2017-12-27 Uploads work. Manually tested with `python setup.py upload` and `twine upload`

* *2017-12-22* Use Python 3, downloads go through the API Gateway so pip's caching now works

* *2017-03-24* The configuration has moved from `./elasticpypi/config.json` to `./serverless.yml` and is consumed by elasticpypi as environment variables. If you are upgrading from an older version, you may need to migrate your configuration to serverless.yml.


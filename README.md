# Rotate Credentials Lambda
This lambda script sets up a IAM user access key rotation schedule.  This
script is not designed to actually auto rotate keys. It's designed to make the
IAM user rotate their own keys because there is no mechanism to auto rotate
keys on behalf of another user in AWS.

Specifically, it does two things:
1. Notify users when their key is about to expire and ask them to rotate their
   key themselves.
2. Upon key expiration it will automatically deactivate keys and notify users
   that they will need to rotate their expired keys.

## Requirements
Run `pipenv install --dev` to install both production and development
requirements, and `pipenv shell` to activate the virtual environment. For more
information see the [pipenv docs](https://pipenv.pypa.io/en/latest/).

After activating the virtual environment, run `pre-commit install` to install
the [pre-commit](https://pre-commit.com/) git hook.

## Create a local build

```shell script
$ sam build --use-container
```

## Deployment

### Build

```shell script
sam build
```

## Deploy Lambda to S3
This requires the correct permissions to upload to bucket
`bootstrap-awss3cloudformationbucket-19qromfd235z9` and
`essentials-awss3lambdaartifactsbucket-x29ftznj6pqw`

```shell script
sam package --template-file .aws-sam/build/template.yaml \
  --s3-bucket essentials-awss3lambdaartifactsbucket-x29ftznj6pqw \
  --output-template-file .aws-sam/build/rotate-credentials.yaml

aws s3 cp .aws-sam/build/rotate-credentials.yaml s3://bootstrap-awss3cloudformationbucket-19qromfd235z9/rotate-credentials/master/
```

## Install Lambda into AWS
Create the following [sceptre](https://github.com/Sceptre/sceptre) file

config/prod/rotate-credentials.yaml
```yaml
template_path: "remote/rotate-credentials.yaml"
stack_name: "rotate-credentials"
stack_tags:
  Department: "Platform"
  Project: "Infrastructure"
  OwnerEmail: "it@sagebase.org"
hooks:
  before_launch:
    - !cmd "curl https://s3.amazonaws.com/bootstrap-awss3cloudformationbucket-19qromfd235z9/rotate-credentials/master/rotate-credentials.yaml --create-dirs -o templates/remote/rotate-credentials.yaml"
```

Install the lambda using sceptre:
```shell script
sceptre --var "profile=my-profile" --var "region=us-east-1" launch prod/rotate-credentials.yaml
```

## Author

Your Name Here.
FIXME: I don't know who the author is

Here is a guidline for naming your S3 bucket.




## Create AWS S3 bucket
Create a [Amazon S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html)

One of the option for creating a bucket is by using [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/create-bucket.html)
```
aws s3api create-bucket \
    --bucket my-bucket \
    --region us-east-1
```

Follow the [guideline for naming](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html) your S3 bucket.
```
Bucket Name:  mybucket.mydomain.com
AWS Region: US West (N. California) us-west-1
Object Ownership: ACLs disabled
Block Public Access settings for this bucket: Off
Bucket Versioning: Disable
Default encryption: Disable
```

Open a terminal and verify that the bucket was created

```
aws s3 ls s3://<BUCKET-NAME>

aws s3 ls
```

## How to upload a test file to AWS S3 bucket using the CLI
To upload a file to S3, you’ll need to provide two arguments (source and destination) to the aws s3 cp command.
```
aws s3 cp test.txt s3://<BUCKET-NAME>
```


## How to upload multiple zip files to AWS S3 bucket using the CLI
Open a terminal and go to the directory where the the zip files are located

Uploads the zip files in the current directory to your AWS bucket
```
for f in $(ls *) ; do aws s3 cp $f s3://ext.yourbucketsuffix.edu; done
```

## Authenticate with Google Cloud Container Registry
Authenticate for Linux or [follow the instruction for your environment](https://cloud.google.com/container-registry/docs/advanced-authentication)

Add the user that you use to run Docker commands to the Docker security group
```
sudo usermod -a -G docker ${USER}
```
Log in to gcloud as the user that will run Docker commands.
```
gcloud auth login
```
Configure Docker with the following command
```
gcloud auth configure-docker
```

Your credentials are saved in your user home directory
```
$HOME/.docker/config.json
```


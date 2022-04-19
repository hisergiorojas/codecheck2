## Create production configuration file 
Generate the application secret and store it at production configuration file
```
echo "play.http.secret.key=\"$(head -c 32 /dev/urandom | base64)\"" > conf/production.conf
```

```Here is a guidline for naming your S3 bucket.


echo "com.horstmann.codecheck.comrun.remote=\"http://host.docker.internal:8080/api/upload\"" >> conf/production.conf
```

Update conf/production.conf


```
play.http.secret.key="see above"
com.horstmann.codecheck.comrun.remote="the URL of the comrun service"
com.horstmann.codecheck.s3.accessKey="your AWS credentials"
com.horstmann.codecheck.s3.secretKey=""
com.horstmann.codecheck.s3bucketsuffix="mybucket.mydomain.com"
com.horstmann.codecheck.s3.region=your AWS region such as "us-west-1"
com.horstmann.codecheck.repo.ext=""
```



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


## Error cgroups: cgroup mountpoint does not exist: unknown
A temporary fix
```
sudo mkdir /sys/fs/cgroup/systemd
sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```
## Unknown Host Exception: host.docker.internal
Add this to production.conf
```
com.horstmann.codecheck.storeLocation=
```

## Deployment failed      
## ERROR: (gcloud.run.deploy) 
Cloud Run error: Container failed to start. Failed to start and then listen on the port defined by the PORT environment variable. Logs for this revision might contain more information.

```
com.horstmann.codecheck.storeLocation=""
```
## System Error: java.util.NoSuchElementException: submissionrun1/_run
Update the production.conf file with /api/upload append at the end of the comrun remote url. 
```
com.horstmann.codecheck.comrun.remote="https://comrun-url/api/upload"

```

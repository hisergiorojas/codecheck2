Docker Deployment
-----------------


## Build and run the comrun service Docker container
From the root directory of the repository, build the comrun service Docker container
```
sudo docker build --tag codecheck:1.0-SNAPSHOT comrun
```
From the root directory of the repository, run the comrun service Docker container
```
sudo docker run -p 8080:8080 -it codecheck:1.0-SNAPSHOT
```
To verify that it works
```
/opt/codecheck/codecheck -l samples/java/example1 &
```


## Build and run the play-codecheck Docker container
From the root directory of the repository, build the Docker image
```
sudo sbt docker:publishLocal 
```
From the root directory of the repository, run the Docker container
```
sudo docker run -p 9090:9000 -it --add-host host.docker.internal:host-gateway play-codecheck:1.0-SNAPSHOT
```
To verify that it works visit the url and upload a problem.
```
http://localhost:9090/assets/uploadProblem.html
```




# Cloud Deployment



conf/production.conf


```
play.http.secret.key="see above"
com.horstmann.codecheck.comrun.remote="https://comrun-url/api/upload" google cloud run service
com.horstmann.codecheck.s3.accessKey="your AWS credentials"
com.horstmann.codecheck.s3.secretKey=""
com.horstmann.codecheck.s3bucketsuffix="mybucket.mydomain.com"
com.horstmann.codecheck.s3.region=your AWS region such as "us-west-1"
com.horstmann.codecheck.repo.ext=""
com.horstmann.codecheck.storeLocation=""
```

Comrun Service Deployment {#service-deployment}
-------------------------

There are two parts to the CodeCheck server. We\'ll take them up one at
a time. The `comrun` service compiles and runs student programs,
isolated from the web app and separately scalable.

We would be deploying the `comrun` service to [Google Cloud](https://cloud.google.com/).

Create a [Google Cloud Run](https://console.cloud.google.com/run?project) project and a define a service for `comrun`.

* Click on Create Service
* For Deploy one revision from an existing container image: Click on Test With a Sample Container
* Change the Service name to Comrun
* Don't change the Region default value
* CPU allocation and pricing should be CPU is only allocated during request processing
* Authentication should be Allow unauthenticated invocations



After creating a project look for the project id 
```
export PROJECT=your Google project id
```

Deploy the Comrun service
```
docker tag codecheck:1.0-SNAPSHOT gcr.io/$PROJECT/comrun
docker push gcr.io/$PROJECT/comrun

gcloud run deploy comrun \
  --image gcr.io/$PROJECT/comrun \
  --port 8080 \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --min-instances=1 \
  --max-instances=50 \
  --memory=512Mi \
  --concurrency=40
```

You should get a URL for the service. Make a note of it---it won't change, and you need it in the next steps. To test that the service is properly deployed, do this:



To verify that the service is properly deployed, You should get a report that was obtained by sending the compile and run jobs to your remote service.
```
export REMOTE_URL=the URL of the comrun service
cd path to/codecheck2 
/opt/codecheck/codecheck -rt samples/java/example1
```


Alternatively, you can test with the locally running web app. In
`conf/production.conf`, you need to add

    com.horstmann.codecheck.comrun.remote= the URL of the comrun service
    
 
To check the health of the comrun service, visit this url

```
https://service url/api/health
```


## How to scale your comrun service
Following the [guideline from Google Cloud](https://cloud.google.com/run/docs/about-instance-autoscaling)

Go to your [google cloud Run](https://console.cloud.google.com/run)

* Click on a comrun service
* Click on Edit and Deploy New Revision
* Under Capacity you can change the Memory and CPU settings
* Under Capacity you can change the Maximum requests per container (Concurrency)
* Under Autoscaling you can change the Minimum and Maximum instances
    

Play Server Deployment {#server-deployment}
----------------------

In Amazon S3, create a bucket whose name starts with the four characters `ext.` and an arbitrary suffix, such as `ext.mydomain.com` to hold
the uploaded CodeCheck problems. Set the ACL so that the bucket owner has all access rights and nobody else has any.

We would be deploying the play server to [Google Cloud](https://cloud.google.com/).

Create a [Google Cloud Run](https://console.cloud.google.com/run?project) project and a define a service for `play-codecheck`.

Add the following to `conf/production.conf`:

    play.http.secret.key= see above
    com.horstmann.codecheck.comrun.remote=comrun host URL/api/upload
    com.horstmann.codecheck.s3.accessKey= your AWS credentials
    com.horstmann.codecheck.s3.secretKey=
    com.horstmann.codecheck.s3bucketsuffix="mydomain.com"
    com.horstmann.codecheck.s3.region=your AWS region such as "us-west-1"
    com.horstmann.codecheck.repo.ext=""
    com.horstmann.codecheck.storeLocation=""

After creating a project look for the project id 
```
export PROJECT=your Google project id
```

Deploy the `play-codecheck` service:

    docker tag play-codecheck:1.0-SNAPSHOT gcr.io/$PROJECT/play-codecheck
    docker push gcr.io/$PROJECT/play-codecheck

    gcloud run deploy play-codecheck \
      --image gcr.io/$PROJECT/play-codecheck \
      --port 9000 \
      --platform managed \
      --region us-central1 \
      --allow-unauthenticated \
      --min-instances=1
      

You will get a URL for the service.

To verify that it works visit this url and upload a problem
```
https://service url/assets/uploadProblem.html
```

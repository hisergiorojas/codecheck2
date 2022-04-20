CodeCheck^®^ Build Instructions
===============================

## Supported Platforms
Codecheck is currently supported on Linux platforms (Ubuntu 20.04 LTS).

Program Structure
-----------------

CodeCheck has two parts:

-   A web application that manages submission and assignments. It is
    called `play-codecheck` because it is based on the Play framework.
    It is convenient, but not necessary, to run it in Docker.
-   A Dockerized service that compiles and runs programs, called
    `comrun`.

The `play-codecheck` program has a number of responsibilities:

-   Display problems, collect submissions from students, and check them
-   Manage problems from instructors
-   Manage assignments (consisting of multiple problems)
-   Interface with learning management systems through the LTI protocol

You can find a listing of the supported REST services in the
`app/conf/routes` file.

The `comrun` service is extremely simple. You can find a basic
description in the `comrun/bin/comrun` script.

For local testing of problems, there is also a handy command-line tool.
This tool uses only the part of `play-codecheck` that deals with
checking a problem (in the `com.horstmann.codecheck` package). The tool
is called `codecheck`. It is created by the `cli/build.xml` Ant script.




## Download Codecheck codebase
Download the codecheck source code using git to clone the repository
```
git clone https://github.com/cayhorstmann/codecheck2
```

## (Automatically) Install Codecheck dependencies
Go to the root directory of codecheck2
```
cd codecheck2
```
### For Debian based Linux

Make codecheck_deb_install script executable
```
chmod +x codecheck_deb_install.sh
```
Run the script
```
./codecheck_deb_install.sh
```

### For Rehat based Linux
Make codecheck_rpm_install script executable
```
chmod +x codecheck_rpm_install.sh
```
Run the script
```
./codecheck_rpm_install.sh
```





## Setup Codecheck environment
Create a /opt/codecheck directory and a subdirectory ext that you own
```
sudo mkdir -p /opt/codecheck/ext
export ME=$(whoami) ; sudo -E chown $ME /opt/codecheck /opt/codecheck/ext
```

## Build the command-line tool
Install jackson-core, jackson-annotations, and jackson-databind jar files

From the root directory of the repository, go to cli directory and make a lib directory
```
cd cli
mkdir lib
```

Install the jar files in the lib directory
```
cd lib
curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-core/2.6.4/jackson-core-2.6.4.jar
curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-annotations/2.6.4/jackson-annotations-2.6.4.jar
curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-databind/2.6.4/jackson-databind-2.6.4.jar
```
Install checkstyle, hamcrest-core, and junit jar files

From the root directory of the repository, go to comrun directory and next go to bin directory, then make a lib directory
```
cd comrun/bin
mkdir lib
```
Install the jar files in the lib directory
```
cd lib
curl -LOs https://repo1.maven.org/maven2/com/puppycrawl/tools/checkstyle/8.42/checkstyle-8.42.jar
curl -LOs https://repo1.maven.org/maven2/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar
curl -LOs https://repo1.maven.org/maven2/junit/junit/4.13.2/junit-4.13.2.jar
```
From the root directory of the repository, build the command-line tool
```
ant -f cli/build.xml
```
To verify that it works
```
/opt/codecheck/codecheck -t samples/java/example1
```
If you omit the `-t`, you get a report with your default browser instead
of the text report.



    

Debugging the Command Line Tool
-------------------------------

If you are making changes to the part of CodeCheck that does the actual
code checking, such as adding a new language, and you need to run a
debugger, it is easiest to debug the command line tool.

Make directories for the submission and problem files, and populate them
with samples.

In your debug configuration, set:

-   The main class to

        com.horstmann.codecheck.Main

-   Program arguments to

        /path/to/submissiondir /path/to/problemdir

-   VM arguments to

        -Duser.language=en
        -Duser.country=US
        -Dcom.horstmann.codecheck.comrun.local=/opt/codecheck/comrun
        -Dcom.horstmann.codecheck.report=HTML
        -Dcom.horstmann.codecheck.debug

-   The environment variable `COMRUN_USER` to your username

To debug on Windows or MacOS, you have to use the Docker container for
compilation and execution.

    docker build --tag comrun:1.0-SNAPSHOT comrun
    docker run -p 8080:8080 -it comrun:1.0-SNAPSHOT

Point your browser to <http://localhost:8080/api/health> to check that
the container is running.

When debugging, add the VM argument

    -Dcom.horstmann.codecheck.comrun.remote=http://localhost:8080/api/upload

## Run the web application locally

From the root directory of the repository, run the play-codecheck server    
```
sbt run
```



To verify that it works visit this url and upload a problem
```
http://localhost:9000/assets/uploadProblem.html
```
Note: The problem files will be located inside the `/opt/codecheck/ext`
directory.

Debugging the Server
--------------------

Install, following the instructions

-   [Eclipse](https://www.eclipse.org/eclipseide/)

Import the project into Eclipse. Run

    sbt eclipse

Then open Eclipse and import the created project.

Make a debugger configuration. Select Run → Debug Configurations,
right-click on Remote Java Application, and select New Configuration.
Specify:

-   Project: `play-codecheck`
-   Connection type: Standard
-   Host: `localhost`
-   Port: 9999

Run the `play-codecheck` server in debug mode:

    COMRUN_USER=$(whoami) sbt -jvm-debug 9999 run

In Eclipse, select Run → Debug Configurations, select the configuration
you created, and select Debug. Point the browser to a URL such as
<http://localhost:9090/assets/uploadProblem.html>. Set breakpoints as
needed.

Docker Deployment
-----------------

Create a file `conf/production.conf` holding an [application
secret](https://www.playframework.com/documentation/2.8.x/ApplicationSecret):

    echo "play.http.secret.key=\"$(head -c 32 /dev/urandom | base64)\"" > conf/production.conf
    echo "com.horstmann.codecheck.comrun.remote=\"http://host.docker.internal:8080/api/upload\"" >> conf/production.conf

Do not check this file into version control!

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

### Shutdown both containers
Open a terminal and shutdown both containers
```
docker container kill $(docker ps -q)
```


# Cloud Deployment

## Update production configuration file 
Generate the application secret and store it at production configuration file
```
echo "play.http.secret.key=\"$(head -c 32 /dev/urandom | base64)\""
```


conf/production.conf


```
play.http.secret.key="see above"
com.horstmann.codecheck.comrun.remote="the URL of the comrun service"
com.horstmann.codecheck.s3.accessKey="your AWS credentials"
com.horstmann.codecheck.s3.secretKey=""
com.horstmann.codecheck.s3bucketsuffix="mybucket.mydomain.com"
com.horstmann.codecheck.s3.region=your AWS region such as "us-west-1"
com.horstmann.codecheck.repo.ext=""
```

Unknown Host Exception: host.docker.internal
Add this to production.conf
```
com.horstmann.codecheck.storeLocation=
```


Deployment failed      
ERROR: (gcloud.run.deploy) 
Cloud Run error: Container failed to start. Failed to start and then listen on the port defined by the PORT environment variable. Logs for this revision might contain more information.

```
com.horstmann.codecheck.storeLocation=""
```

System Error: java.util.NoSuchElementException: submissionrun1/_run
Update the production.conf file with /api/upload append at the end of the comrun remote url. 
```
com.horstmann.codecheck.comrun.remote="https://comrun-url/api/upload"

```

Comrun Service Deployment {#service-deployment}
-------------------------

There are two parts to the CodeCheck server. We\'ll take them up one at
a time. The `comrun` service compiles and runs student programs,
isolated from the web app and separately scalable.

We would be deploying the `comrun` service to [Google Cloud](https://cloud.google.com/).

Create a [Google Cloud Run](https://console.cloud.google.com/run?project) project and a define a service for `comrun`.


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

## Check if the comrun service is running

You will get a URL for the service.

To verify that the service is properly deployed, You should get a report that was obtained by sending the compile and run jobs to your remote service.
```
export REMOTE_URL=the URL of the comrun service
cd path to/codecheck2 
/opt/codecheck/codecheck -rt samples/java/example1
```


Alternatively, you can test with the locally running web app. In
`conf/production.conf`, you need to add

    com.horstmann.codecheck.comrun.remote= the URL of the comrun service
    
 
To verify that the comrun service is running, visit this url

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

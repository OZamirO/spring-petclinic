# My PetClinic Assignment

## Practicing Jenkins pipeline 

<br>

This repo was created as part of an assignment, with the following guidelines:

1. Use Spring pet-clinic (https://github.com/spring-projects/spring-petclinic) as your project source code

2. Build a Jenkins pipeline with the following steps:
    1. Compile the code
    2. Run the tests
    3. Package the project as a runnable Docker image
    4. Publish the image to JFrog Artifactory in your pipeline

3. Make sure all dependencies are resolved from Maven Central

<br>
Deliverables:

1. GitHub link to the repo including
    1. Jenkins file within that repo
    2. Docker file within that repo
    3. readme.md file explaining the work and how to run the project

2. Command to obtain and run the docker image

---

<br>

## Running the project locally

For this practice I used an Amazon Linux 2 machine.
The following packages are required:
- Docker 
- Jenkins
- JDK 17 (11 or newer)
- git


<br>

> Note: In my case, I had to modify the jenkins user permissions in order to allow it to run docker commands.   
>```  
>sudo usermod -aG docker jenkins  
>```  
><br>
<br>

### Make sure all dependencies are resolved from Maven Central

In order to make sure all dependencies are resolved from Maven Central, I edited the pom.xml file

```diff
<repositories>
+     <repository>
+       <id>central</id>
+        <url>https://repo.maven.apache.org/maven2</url>
+        <releases>
+            <enabled>true</enabled>
+        </releases>
+    </repository>
!   <!-- <repository>
!      <id>spring-snapshots</id>
!      <name>Spring Snapshots</name>
!      <url>https://repo.spring.io/snapshot</url>
!      <snapshots>
!        <enabled>true</enabled>
!      </snapshots>
!    </repository>
!    <repository>
!      <id>spring-milestones</id>
!      <name>Spring Milestones</name>
!      <url>https://repo.spring.io/milestone</url>
!      <snapshots>
!        <enabled>false</enabled>
!      </snapshots>
!    </repository> -->
  </repositories>

  <pluginRepositories>
+    <pluginRepository>
+      <id>central</id>
+        <url>https://repo.maven.apache.org/maven2</url>
+        <releases>
+            <enabled>true</enabled>
+        </releases>
+    </pluginRepository>
!    <!--<pluginRepository>
!      <id>spring-snapshots</id>
!      <name>Spring Snapshots</name>
!      <url>https://repo.spring.io/snapshot</url>
!      <snapshots>
!        <enabled>true</enabled>
!      </snapshots>
!    </pluginRepository>
!    <pluginRepository>
!      <id>spring-milestones</id>
!      <name>Spring Milestones</name>
!      <url>https://repo.spring.io/milestone</url>
!      <snapshots>
!        <enabled>false</enabled>
!      </snapshots>
!    </pluginRepository>-->
  </pluginRepositories>

```

<br>




### To compile and run the project
```
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw package
java -jar target/*.jar
```
<br>

You can then access petclinic here: http://<YOUR_MACHINE_IP>:8080/

<br>

> NOTE:`./mvnw package` includes both compilation and testing of the code.   
> I need to separate it to different stages later in my Jenkinsfile.

> NOTE: Jenkins also uses tcp port 8080, depanding on your environment, you may need to stop its process

<br>

In order to separate the process into separate stages I used the following:

```
./mvnw -U -Dmaven.test.skip=true -T 4 clean install
./mvnw -U test

```

---
<br><br>

## Running the project as a Docker container

As part of the assignment I was asked to create and use a Dockerfile in order to pakage the app as a Docker Image.

<br>
Here is the Dockerfile content:

```
FROM openjdk:17
COPY . /PetClinic
WORKDIR /PetClinic
RUN ./mvnw -U -Dmaven.test.skip=true -T 4 clean install
WORKDIR target
CMD ["java", "-jar", "spring-petclinic-3.0.0-SNAPSHOT.jar" ]
```
<br>
The next step would be building the Docker Image.

```
docker build . -t "ImageName"
```
<br>
Now let's test it 

```
docker run -p 80:8080 ImageName
```

> NOTE: PetClinic is using tcp port 8080 by default.  
> Jenkins is running on my machine and uses tcp port 8080.   
> I used `-p 80:8080` to map tcp port 80 of my machine to tcp port 8080 of the container
---

<br><br>

## Manualy pushing the Image to Artifactory

Let's push the image.
> Note:  For this step you will need a Jfrog Artifactory server. I used the cloud version of Jfrog Artifactory.

> Note:  Make sure you have the necessary permissions to upload files to Jfrog Artifactory.

In my environment login is required.
```
docker login myRTtenant.jfrog.io
```
Enter you credentials.

Once you get `Login Succeeded`, you can continue with the Push command.

Before that, you must tag the Image

```
docker tag ImageName myRTtenant.jfrog.io/MyDockerRepo/ImageName:latest
```

<br>
Now let's push it

```
docker push myRTtenant.jfrog.io/MyDockerRepo/ImageName:latest
```
<br>

---

## Cleanup 

Before the last manual step (pulling and running the image), let's cleanup everything.

```
docker container prune 
docker images prune -a 

```


<br>

---
## Pull the image and make it a running container

Let's pull the image from our Jfrom Artifactory repo and run it.

```
docker run -d -p 80:8080 myRTtenant.jfrog.io/MyDockerRepo/ImageName:latest
```

> Note: Since we purged all the images previously created, The image will be automatically pulled.

<br>

Verify the container is up 

```
docker ps
```

You can now  access petclinic here: http://<YOUR_MACHINE_IP>

<br>

---

## Working with the Jenkinsfile

<br>

The Jenkinsfile is made out of two major section, Environment and Stages.

<br>

### Environment

<br>

The Environment section contains all the necesary variables which will be used in the following stages

Some variables are clear text, which can be directly edited according to your environment. Others are secrets that must be configured as Jenkins credentials.

```
RT_SRV =        'myRTtenant.jfrog.io'
RT_USER =       credentials('rtuser')
RT_PASS =       credentials('rtpass')
RT_TRG_REPO =   'MyDockerRepo'
IMG_NAME =      'ImageName'
```

### Stages

The stages part defines the stages in our pipeline and uses the variables defined in the Environment section.

My pipeline contains the following stages:

## Compile

Will compile the code while using the most updated releases and snapshots on remote repositories, will skip any tests and remove previously compiled code

<br>

## Test

Will test the compiled code.

<br>

## Package

This stage will build the Docker Image (Using the Dockerfile described above)

<br>

## Publish

Will tag the Docker Image and push is to Artifactory.

<br>

## Cleanup

In this stage we are removing all unused docker images from the local device.

---




<br><br><br><br><br><br> 



### References:
[mvn Commands and Options](https://www.digitalocean.com/community/tutorials/maven-commands-options-cheat-sheet)

[Using a Jenkinsfile](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/)


[Jfrog Pushing and Pulling Images](https://www.jfrog.com/confluence/display/JFROG/Docker+Registry#:~:text=4.9.0%22%2C%20%22targetTag%22%20%3A%20%22latest%22%7D%27-,Pushing%20and%20Pulling%20Images,-Set%20Me%20Up)

[Dockerfile Commands and Variables](https://docs.docker.com/engine/reference/builder/)

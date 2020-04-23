
# Running Java Application within Docker Container for absolute beginners
## Practical Docker Commands for running a Java Console Application for Chatting
This markdown document is list of practical docker commands for an absolute beginner 
who wish to run first java application in docker container.


## Step-1: Setup sample application code: 

For this demonstration, download the sample chat application from [this link](https://www.codejava.net/java-se/networking/how-to-create-a-chat-console-application-in-java-using-socket). 
We will first build the executable jars from source within the openjdk container to keep the compatibility and get familiarized with docker. 


Setup the code as per below directory structure: 

```
C:\JAVACHATCONSOLE
│   dockerfile
│
├───client
│       ChatClient.java
│       ChatClient.mf
│       ReadThread.java
│       WriteThread.java
│
└───server
        ChatServer.java
        ChatServer.mf
        UserThread.java

```

## Step-2: Pull the docker image for OpenJDK:

For this demo, we will use [`openjdk:8-jdk-alpine`](https://hub.docker.com/_/openjdk). 
Use below command to pull it: 

```docker pull openjdk:8-jdk-alpine```

## Step-3: Run a docker container using openjdk:
We assume that we have the java source code for the `ChatServer` and `ChatClient` and we need to build executable jar files.
For this, we will have to run the container mounting the local folder and then use `javac` and `jar` to compile and archive into executable jar. 

Use below command run the container based on `openjdk:8-jdk-alpine` image: 

```docker run -it -d --rm  --name MyJavaApp openjdk:8-jdk-alpine```

Verify the running container using: 

``` docker ps   ```

Note down the container id. Now, to get SSH access into the running container based on `alpine`:

``` docker exec -it <container-d> sh```

Inside container, verify the `java` by running commands:

``` 
java -version 
javac -version

``` 

## Step-4: Copying source files into docker container:
Create a directory within container using:

```
mkdir /rswarnka
cd /rswarnka
mkdir server
mkdir client
```

In another `cmd` window, use below commands to copy the source files from Host to container:

```
# copy the chat server sources
docker cp c:/JavaChatConsole/server/ChatServer.java <container-id>:/rswarnka/server/ChatServer.java
docker cp c:/JavaChatConsole/server/UserThread.java <container-id>:/rswarnka/server/UserThread.java
docker cp c:/JavaChatConsole/server/ChatServer.mf <container-id>:/rswarnka/server/ChatServer.mf

# copy the chat client sources
docker cp c:/JavaChatConsole/client/ChatClient.java <container-id>:/rswarnka/client/ChatClient.java
docker cp c:/JavaChatConsole/client/ReadThread.java <container-id>:/rswarnka/client/ReadThread.java
docker cp c:/JavaChatConsole/client/WriteThread.java <container-id>:/rswarnka/client/WriteThread.java
docker cp c:/JavaChatConsole/client/ChatClient.mf <container-id>:/rswarnka/client/ChatClient.mf
```

## Step-5: Compile `ChatServer.java` and create jar into docker container:

Switch to SSH, and run command to compile the server code: 

```
# compile the chat server sources

# Below command yeilds compiled .class files
javac ChatServer.java

# Use manifest to make executable jar file
jar -cvfm ChatServer.jar ChatServer.mf *.class

```

Verify the executable jar by running: 

```
java -jar ChatServer.jar 8989
# Chat Server is listening on port 8989. Ctrl+C to exit. 
```

Remove chat server source files from container (optional):
```
cd /rswarnka/server/
rm *.java
rm *.class
``` 

Copy the executable jar file from container to host: 
```
docker cp <container-id>:/rswarnka/ChatServer.jar c:/JavaChatConsole/server/ChatServer.jar
```




## Step-6: Compile `ChatClient.java`:

Switch to SSH, and run command to compile the chat client code: 

```
# compile the chat client sources

# Below command yeilds compiled .class files
javac ChatClient.java

# Use manifest to make executable jar file
jar -cvfm ChatClient.jar ChatClient.mf *.class

```

Verify the executable jar by running: 

```
java -jar ChatClient.jar localhost 8989
# Server not found. Ctrl+C to exit. 
```

Remove chat client source files from container (optional):
```
cd /rswarnka/client/
rm *.java
rm *.class
``` 

Copy the executable jar file from container to host: 
```
docker cp <container-id>:/rswarnka/client/ChatClient.jar c:/JavaChatConsole/client/ChatClient.jar
```

Exit the docker container and stop the container. 
```
docker stop <container-id>
```


## Step-7: Manually building docker image for Chat Server:

We will see how to __manually__ build docker container using `docker commit` command (unlike using a *dockerfile*). 

Spawn a new docker container using `openjdk:8-jdk-alpine` image: 

```docker run -it -d --rm  --name JavaChatServer openjdk:8-jdk-alpine```

Verify the running container using & get container id: 

``` docker ps  ```

Note down the container id. Now, to get SSH access into the running container based on `alpine`:

``` docker exec -it <container-d> sh```


Inside container, create a new directory and copy the `ChatServer.jar` from host to container: 
```
mkdir /rswarnka
docker cp c:/JavaChatConsole/server/ChatServer.jar <container-id>:/rswarnka/ChatServer.jar 
```


Exit from container, and commit the modified container into image  using `docker commit` : 
```
docker commit <container-id> <image-name>:<image-tag>

#example: 

docker commit <container-id> javachatserver:v1.01

```

This will run commmit the container into a nwe image from which customized container can be launched. 


## Step-8: Run custom docker image for Chat Server:

Use below command run the customised container `javachatserver:v1.01` based on `openjdk:8-jdk-alpine` image with the port mapping: 


```
docker run -it -d --rm -p 8989:8989 --name MyJavaChatServer javachatserver:v1.01 sh -c  "cd /rswarnka; /usr/bin/java -jar ChatServer.jar 8989"
```

Verify the running container using: 

``` docker ps  ```

That's it for running a custom java chat server. Now let's run two chat client consoles to connect to chat server and do some chatting ! :)

## Step-9: Run Chat clients and have fun:

Run chat client executable jar in two `cmd`s. (You must have JRE on your system installed to run jar applications.):

```
cd c:/JavaChatConsole/client/ChatClient.jar
java -jar ChatClient.jar
```

## Step-10: Happy Chatting with Docker!

## Excercise: 
Write a `dockerfile` to automate above steps.

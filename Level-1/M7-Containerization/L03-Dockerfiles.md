# DOCKERFILE

A Docker Image is a read-only file with a bunch of instructions. When these instructions are executed, it creates a Docker container. Dockerfile is a simple text file that consists of instructions to build Docker images.

## About Dockerfile & Use Cases.
Docker can build images automatically by reading the instructions from a Dockerfile.
  
build command is used to create an image from the Dockerfile.

        $ docker build 

You can name your image as well.

        $ docker build -t my-image 

If your Dockerfile is placed in another path,

        $ docker build -f /path/to/a/Dockerfile. 

![](Images/docker5.png)

## DockerFile Format : INSTRUCTION arguments

Dockerfile Commands
- FROM - specifies the base(parent) image. Alpine version is the minimal docker image based on Alpine Linux which is only 5mb in size.
- RUN - runs a Linux command. Used to install packages into container, create folders, etc
- ENV - sets environment variables. We can have multiple variables in a single dockerfile.
- COPY - copies files and directories to the container.
- EXPOSE - expose ports
- ENTRYPOINT - provides command and arguments for an executing container.
- CMD - provides a command and arguments for an executing container. There can be only one CMD.
- VOLUME - create a directory mount point to access and store persistent data.
- WORKDIR - sets the working directory for the instructions that follow.
- LABEL - provides metada like maintainer.
- ADD - Copies files and directories to the container. Can unpack compressed files.
- ARG - Define build-time variable.

There are a few commands which are a little confusing. Let's have a look at them.

### COPY vs. ADD
Both commands serve a similar purpose, to copy files into the image.
- COPY - let you copy files and directories from the host.
- ADD - does the same. Additionally it lets you use URL location and unzip files into images.
Docker documentation recommends using the COPY command.

### ENTRYPOINT vs. CMD
- CMD - allows you to set a default command which will be executed only when you run a container without specifying a command. If a Docker container runs with a command, the default command will be ignored.
- ENTRYPOINT - allows you to configure a container that will run as an executable. ENTRYPOINT command and parameters are not ignored when Docker container runs with command line parameters.

### VOLUME
You declare VOLUME  in your Dockerfile to denote where your container will write application data. When you run your container using -v   you can specify its mounting point.

## MULTISTAGE DOCKERFILE:	
A multistage build allows you to use multiple images to build a final product. In a multistage build, you have a single Dockerfile, but can define multiple images inside it to help build the final image. In this post, you’ll learn about the core concepts of multistage builds in Docker and how they help to create production-grade images.

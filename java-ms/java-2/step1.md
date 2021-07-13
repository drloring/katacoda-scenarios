In this step, we're going to take the web service we created in the previous lesson and package it into a Docker image.

First, we're going to collect the executable jar we made in the last lesson `curl -o rest-service.jar https://raw.githubusercontent.com/drloring/katacoda-resources/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}}.

Run `ls -lha`{{execute}} to confirm the jar file transferred.  

Now create an empty Dockerfile `touch Dockerfile`{{execute}}

Open the `Dockerfile`{{open}} in the editor 

Next, we define the base image that we're going to be basing our image on.  For this, we're using ubuntu linux, but there are several to choose from.  Add <pre class="file" data-filename="Dockerfile" data-target="prepend">FROM ubuntu</pre> to the Dockerfile

Let's look at the images that are already installed on the system using `docker images`{{execute}} command.  Notice that several images are already available in our local docker repository.  To see if any are running, type `docker ps -a`{{execute}}.  None should be running, but that will change in a moment.

We're going to build our Docker image now and tag it with the `java-ws` tag using `docker build -t java-ws .`{{execute}}.  Notice the -t option to tag it and the period at the end just means we're using our local filesystem to look for the Dockerfile.  Dockerfile is the default, but you can name it anything and refer to it in the docker build command-line args.

Verify the docker image was build by running `docker images | grep java-ws`{{execute}}. 

Now we can run the docker image and pass a few arguments to it so that we can interact with the running container.  Run `docker run --name jws --rm -it java-ws bash`{{execute}} and observe that the command prompt has changed.  What that last command did was to execute a bash shell within the docker image named `java-ws`.  Passing in `-it` provides the interactive terminal and `--rm` removes the container when it exits.

type `ls -lha`{{execute}} from within the container and navigate through the bare ubuntu image we used as our base image.  When complete, type `exit`{{execute}} or Ctrl-D to quit the container



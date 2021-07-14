Next we're going to add our executable jar file to the Docker image.

If the Dockerfile is not already open, open the `Dockerfile`{{open}} in the editor 

Let's define the working directory <pre class="file" data-filename="Dockerfile" data-target="append">WORKDIR /root</pre> in the Dockerfile

Now, let's add the executable jar to the Dockerfile <pre class="file" data-filename="Dockerfile" data-target="append">COPY rest-service.jar svc.jar</pre>.  Note that the first parameter is the FROM and the second is the TO, you don't have to change names, but it is optional.

Now, let's build this version of the docker image with `docker build -t java-ws .`{{execute}}.  You'll see a couple more layers were added to the docker image this time, and when you view the image with `docker images | grep java-ws`{{execute}}, you'll see that the image has grown in size due to the added jar file.

Run `docker run --name jws --rm -it java-ws bash`{{execute}} and navigate to `/root` to see the svc.jar file in the docker image.  Attempt to run the `java` command in the container.  Notice that there is no java runtime installed.  

We'll have to change the base image to run our java service in the container with <pre class="file" data-filename="Dockerfile" data-target="insert" data-marker="FROM ubuntu">FROM openjdk:15</pre>.

Now we can build and run the java web service in the container.  Add  <pre class="file" data-filename="Dockerfile" data-target="append">CMD java -jar svc.jar</pre> to the Dockerfile and build it again.

One final thing we'll have to do is to expose the port from our container <pre class="file" data-filename="Dockerfile" data-target="append">EXPOSE 8080</pre> so that we can access the port from outside of the container.

Build the final image with `docker build -t java-ws .`{{execute}}, then run with the following command `docker run --name jws --rm -d -p80:8080 java-ws`{{execute}}.  Notice that we're running is in a detatched `-d` state so that we can interact with it from the command line.  Also notice that we've exposed the container port 8080 to port 80 at runtime.  This helps with deconflicting ports on the target system.

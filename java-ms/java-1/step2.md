Now that we have a running Spring Boot based microservice, let's make a docker image for it.

First, you have to kill the currently running application `pkill -9 java`{{execute}}.  NOTE: if you had any other java processes running, that command will kill those as well.

Let's start with a clean Dockerfile by Running `touch Dockerfile`{{execute}}.  Find the Dockerfile in the editor's tree and open it to display the empty contents.

Now we have to create an executable jar file to include in the Docker image.  Run `gradle bootJar`{{execute}} to generate an executable jar of our project.

The jar file is located under ./build/libs, `ls ./build/libs`{{execute}} to see the file.  To confirm that it built correctly, run `java -jar ./build/libs/rest-service-0.0.1-SNAPSHOT.jar`{{execute}}.

You should see the output of the web application running and the line "Tomcat started on port(s): 8080"... Kill the application with `^C`{{execute ctrl-seq}}.



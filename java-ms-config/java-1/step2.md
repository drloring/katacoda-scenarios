Now that we have a (somewhat) externally configured Spring Boot based microservice, let's change the parameters a couple of different ways.

But first, we have to kill the running application `pkill -9 java`{{execute}}.  NOTE: if you had any other java processes running, that command will kill those as well.

The first (and most obvious way) we can change the values are via the application.properties file.  Switch to the `java-1-step-1/src/main/resources/application.properties`{{open}} file in the editor and change the template string to <pre class="file" data-filename="java-1-step-1/src/main/resources/application.properties" data-target="insert" data-marker="template=Hello, %s!">template=Hola, %s!</pre>

and change the port to <pre class="file" data-filename="java-1-step-1/src/main/resources/application.properties" data-target="insert" data-marker="server.port=9090">server.port=9190</pre>

Now when we run `mvn clean install`{{execute}}  and `java -jar target/rest-service-0.0.1-SNAPSHOT.jar &`{{execute}}, we can verify the parameters took with the following command `curl localhost:9190/greeting`{{execute}}

Note: It would take a bit more effort if we wanted to convert the `defaultValue`, `name` and context path in `GreetingController.java`, like we did in the original scenario.

Now, let's pass the properties in on the command line to see how that works.  First, kill the process `pkill -9 java`{{execute}}, then execute the following command 

Then we'll add the override to the command line `java -jar target/rest-service-0.0.1-SNAPSHOT.jar --server.port=9999 &`{{execute}}

And test it out with `curl localhost:9999/greeting`{{execute}} 

Now, that we have demonstrated that we can override the application properties at runtime, next we'll see how we achieve this with docker.

To clean up, let's kill the process with `pkill -9 java`{{execute}}.






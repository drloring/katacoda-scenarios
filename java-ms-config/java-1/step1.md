Welcome to the Java Web Services with external configurations and port binding example project.  This is a continuation of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms/java-1).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

First, let's download the solution `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-1-step-1.zip && unzip java-ms-config-java-1-step-1.zip && cd java-1-step-1`{{execute}}

Spring Boot configuration files are extremely flexible and allow you to provide values from application.properties or environment variables.  Inspecting the `src/main/resources/application.properties`{{open}} file, we see
<pre>
server.port=9090
template=Hello, %s!
</pre>
Which we will use to override the message and port that our web server is running on.

If we look at `src/main/java/com/example/restservice/GreetingController.java`{{open}} We see that we've added a `@Value` annotation to inject the property.
<pre>
@Value("${template}")
private static final String template = "Hello, %s!";
</pre>

What about the server.port value?  That is set behind the scenes by Spring and does not need to be manually injected into our application

Since we're using a minikube environment for this course, we need to export our JAVA_HOME environment variable `export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}, then we can `mvn clean install`{{execute}}.

And, again, run ` java -jar target/rest-service-0.0.1-SNAPSHOT.jar &`{{execute}} to run our web service in the background.

Now, to verify it picked up the `server.port=9090`, run `curl http://localhost:9090/greeting`{{execute}} to display the Hello World message in a json formatted string.

Now that we have an couple of application properties exposed to our runtime, in the next step we'll see how we can manipulate those at runtime.


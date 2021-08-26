Welcome to the Java Web Services with external configurations example project.  This is a continuation of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms/java-1).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

Just like before, we clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{execute}}

CD to the `gs-rest-service/complete` directory `cd gs-rest-service/complete`{{execute}}

Spring Boot configuration files are extremely flexible and allow you to provide values from application.properties or environment variables.  We're going to start with creating an `application.properties` file. Run `mkdir src/main/resources`{{execute}}, then `touch src/main/resources/application.properties`{{execute}}.

Now open the application.properties file in the editor `gs-rest-service/complete/src/main/resources/application.properties`{{open}}.

Edit the application.properties to add a couple properties.  Add the default port for the REST server (recall the default is 8080), add  <pre class="file" data-filename="gs-rest-service/complete/src/main/resources/application.properties" data-target="prepend"># Server port
server.port=9080</pre> 

Then add the template string to application.properties <pre class="file" data-filename="gs-rest-service/complete/src/main/resources/application.properties" data-target="append">template=Hello, %s!</pre>

Now we need to change the Controller to look for these values.  First, we add the Value to the imports <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="import java.util.concurrent.atomic.AtomicLong;">import java.util.concurrent.atomic.AtomicLong;
import org.springframework.beans.factory.annotation.Value;
</pre>

Now we can add the annotation for the template string <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="	private static final String template = "Hello, %s!";">	private static final String template = "Hello, %s!";
</pre>

So, reviewing the changes, we've replaced the inline template string with a `@Value` annotation which is provided in the application.properties file.  What about the server.port value?  That is set behind the scenes by Spring.
 

Before we can compile, we need to update the gradle version `./gradlew wrapper --gradle-version 7.0`{{execute}}

Once you see `Build Successful`, you can continue with the following commands

Run`./gradlew bootRun &`{{execute}} in the background.

Run the following command to verify that the spring boot application is running `curl http://localhost:9080/greeting`{{execute}} to display the Hello World message in a json formatted string.



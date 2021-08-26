Now that we have a (somewhat) externally configured Spring Boot based microservice, let's change the parameters a couple of different ways.

But first, we have to kill the running application `pkill -9 java`{{execute}}.  NOTE: if you had any other java processes running, that command will kill those as well.

The first (and most obvious way) we can change the values are via the application.properties file.  Change the template string to <pre class="file" data-filename="gs-rest-service/complete/src/main/resources/application.properties" data-target="insert" data-marker="template=Hello, %s!">template=Hola, %s!</pre>

and change the port to <pre class="file" data-filename="gs-rest-service/complete/src/main/resources/application.properties" data-target="insert" data-marker="server.port=9080">server.port=9090</pre>

Now when we run `./gradlew bootRun &`{{execute}} we can verify the parameters took with the following command `curl http://localhost:9090/greeting`{{execute}}

Note: It would take a bit more effort if we wanted to convert the `defaultValue`, `name` and context path in `GreetingController.java`, like we did in the original scenario.

Now, let's pass the properties in on the command line to see how that works.  First, kill the process `pkill -9 java`{{execute}}, then execute the following command 

Now, let's rebuild and re-run the web application to check our changes.

`./gradlew bootRun &`{{execute}}

And test is with our new Spanish API with `curl http://localhost:8080/saludo`{{execute}} and `curl http://localhost:8080/saludo?nombre=David`{{execute}}






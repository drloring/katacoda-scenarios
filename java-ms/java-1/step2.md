Now that we have a running Spring Boot based microservice, let's look at what makes it work by looking at the application code.

If you drill into the directory structure in the editor to <pre>gs-rest-service/complete/src/main/java/com/example</pre>, you'll see there's only three files.

Spring boot hides a lot of the complexity of building applications from the developer, GreetingController.java is responsible for listening and responding to REST requests.

Open `/src/main/java/com/example/GreetingController.java`{{open}} in the editor and make the following change

<pre class="file" data-filename="/src/main/java/com/example/GreetingController.java" data-target="insert" data-marker="Hello">Hola</pre>

and <pre class="file" data-filename="/src/main/java/com/example/GreetingController.java" data-target="insert" data-marker="World">Mundo</pre>

and finally <pre class="file" data-filename="/src/main/java/com/example/GreetingController.java" data-target="insert" data-marker="greeting">saludo</pre>

Now, let's rebuild and re-run the web application to check our changes.




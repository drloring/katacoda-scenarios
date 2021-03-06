Now that we have a running Spring Boot based microservice, let's look at what makes it work by looking at the application code.

But first, we have to kill the running application `pkill -9 java`{{execute}}.  NOTE: if you had any other java processes running, that command will kill those as well.

If you drill into the directory structure in the editor to `gs-rest-service/complete/src/main/java/com/example`, you'll see there's only three files.

Spring boot hides a lot of the complexity of building applications from the developer, GreetingController.java is responsible for listening and responding to REST requests.  The method annotated with `@GetMapping("/greeting")` tells you that method is responsible for responding to HTTP GET Requests, and that it listens for the context of `/greeting`.  
The other moving part in this method is in the parameters of the method.  See `@RequestParam(value = "name", defaultValue = "World") String name`. that takes the request parameter called name from the url and maps it to a local environment variable.  That's how you add variability to RESTful APIs.

We're going to make the Spanish version of the API.  Open `gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java`{{open}} in the editor and make the following changes:

Change `Hello` to <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="Hello">Hola</pre> and `World` to <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="World">Mundo</pre> and `greeting` to  <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="greeting">saludo</pre> and finally `name` to  <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="name">nombre</pre>

Now, let's rebuild and re-run the web application to check our changes.

`mvn clean install`{{execute}} then `java -jar target/rest-service-complete-0.0.1-SNAPSHOT.jar &`{{execute}}

And test is with our new Spanish API with `curl http://localhost:8080/saludo`{{execute}} and `curl http://localhost:8080/saludo?nombre=User`{{execute}}






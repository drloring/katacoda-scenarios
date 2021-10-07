Welcome to the stateless Java Web Services with backing service project.  This is a continuation of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms-config/java-1).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

Just like before, we clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{execute}}

CD to the `gs-rest-service/complete` directory `cd gs-rest-service/complete`{{execute}}

We can view this project is an example of a stateful service.  If we open `gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java`{{open}}, we'll see `private final AtomicLong counter = new AtomicLong();` in the code.  This works fine as a counter for a single web application, but if we want to be able to scale this service out to multiple instances, we couldn't rely on the counter since each service would have it's own counter incrementing independent of the other services.

To correct this, we're going to introduce a Redis NoSQL database to store the counter for all of the services and not rely on the in-memory state of the service.

To get started, we'll add a couple dependencies for Spring Data and Redis to the Gradle Build file `gs-rest-service/complete/build.gradle`{{open}}.  <pre class="file" data-filename="gs-rest-service/complete/build.gradle" data-target="insert" data-marker="	implementation 'org.springframework.boot:spring-boot-starter-web'">  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.data:spring-data-redis'
  implementation 'io.lettuce:lettuce-core'
</pre>

Then we'll make some changes to `gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java`{{open}} by adding the imports <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="import java.util.concurrent.atomic.AtomicLong;">import java.util.concurrent.atomic.AtomicLong;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
</pre>

Then, we'll change the message so that we see our counter when we `curl` it.  <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="	private static final String template = "Hello, %s!";">	private static final String template = "Hello, %s! Current Count is %d";</pre>

Then, we'll create a method to perform the lookup and save to the redis database <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="		private final AtomicLong counter = new AtomicLong();">	private final AtomicLong counter = new AtomicLong();
    private static String USER_KEY = "User";

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    
    public int updateCount(String userId) {
    	String currentCount = (String) redisTemplate.opsForHash().get(USER_KEY, userId);
    	int cc = 1;
    	if (currentCount != null) {
    		cc = Integer.parseInt(currentCount) + 1;
    	}
    	redisTemplate.opsForHash().put(USER_KEY, userId, Integer.toString(cc));
    	return cc;
    }
    
</pre>

And finally, we'll create a method to perform the lookup and save to the redis database <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/GreetingController.java" data-target="insert" data-marker="				return new Greeting(counter.incrementAndGet(), String.format(template, name));">		int count = updateCount(name);
		return new Greeting(counter.incrementAndGet(), String.format(template, name, count));    
</pre>

Since we changed the output of the Hello World string, we'll need to modify the tests to account for the change.  Open `gs-rest-service/complete/src/test/java/com/example/restservice/GreetingControllerTests.java`{{open}}.  We'll relax the test to allow any string to pass <pre class="file" data-filename="gs-rest-service/complete/src/test/java/com/example/restservice/GreetingControllerTests.java" data-target="insert" data-marker="				.andExpect(jsonPath("$.content").value("Hello, World!"));">				.andExpect(jsonPath("$.content").isString());</pre> and here <pre class="file" data-filename="gs-rest-service/complete/src/test/java/com/example/restservice/GreetingControllerTests.java" data-target="insert" data-marker="				.andExpect(jsonPath("$.content").value("Hello, Spring Community!"));">				.andExpect(jsonPath("$.content").isString());</pre>

Since we're using a different VM than previously, we need to export our JAVA_HOME environment variable `export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}.

Before we can compile, we need to update the gradle version `./gradlew wrapper --gradle-version 7.0`{{execute}}

Once you see `Build Successful`, you can continue with the following commands

Start a redis server in the background with the docker command `docker run --name myredis -d --rm -p 6379:6379 redis`{{execute}}
	
Run`./gradlew bootRun &`{{execute}} in the background.

Run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message with an increasing counter.  You can stop and start the web application and the count will be saved in redis until redis is restarted.
	
So, we cheated by running gradle bootRun before running gradle build, if we run `./gradlew build`{{execute}} before we have the redis server running, we'll have test failures.  We don't want our unit tests to depend on externally running services, so in the next course, we'll learn how to use Spring application.properies to swap out redis backing services based on where we are running the web service.



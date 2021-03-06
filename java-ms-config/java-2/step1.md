Welcome to the stateless Java Web Services with backing service project.  This is a continuation of the previous katacoda course [here](https://www.katacoda.com/ng-dloring/courses/java-ms-config/java-1).  If you haven't reviewed that one yet, you may want to go through it now and return to this scenario afterwards.

Instead of cloning the original repo `https://github.com/spring-guides/gs-rest-service.git`, I've created a solution with the final implementation and we'll highlight the changes from the baseline.  If you'd prefer, you can download the original project and make the changes to get some hands-on experience.

First,  `wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-1.zip &&unzip java-ms-config-java-2-step-1.zip && cd step-1`{{execute}}

We can view this project is an example of a stateful service.  If we open `step-1/src/main/java/com/example/restservice/GreetingController.java`{{open}}, we'll see `private final AtomicLong counter = new AtomicLong();` in the code.  This works fine as a counter for a single web application, but if we want to be able to scale this service out to multiple instances, we couldn't rely on the counter since each service would have it's own counter incrementing independent of the other services.  We'll see that in-action in the next steps.

To correct this, we're going to introduce a Redis NoSQL database to store the counter for all of the services and not rely on the in-memory state of the service.

To get started, we'll add a couple dependencies for Spring Data and Redis to the Maven POM file `step-1/pom.xml`{{open}}.  Notice we added 

	<dependency>
		<groupId>org.springframework.data</groupId>
		<artifactId>spring-data-redis</artifactId>
	</dependency>
	<dependency>
		<groupId>io.lettuce</groupId>
		<artifactId>lettuce-core</artifactId>
	</dependency>

to the maven pom file, this will pull in the spring data redis and lettuce connectors into the project.

Then we made some changes to `step-1/src/main/java/com/example/restservice/GreetingController.java`{{open}} to add the externalized cache. Notice we changed 
	private static String USER_KEY = &#x22;User&#x22;;

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
to use the redis database to store and retrieve the counter.

Also, notice that we updated the message to include the counter 
	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = &#x22;name&#x22;, defaultValue = &#x22;World&#x22;) String name) {
		int count = updateCount(name);
		return new Greeting(counter.incrementAndGet(), String.format(template, name, counter.get()));
	}

We added `step-1/src/main/java/com/example/restservice/ApplicationConfig.java`{{open}} to connect to the redis server, adding the following content 
	@Configuration
	class ApplicationConfig {

		@Bean
		public LettuceConnectionFactory redisConnectionFactory(RedisProperties redisProperties) {

			return new LettuceConnectionFactory(
					new RedisStandaloneConfiguration(redisProperties.getRedisHost(), redisProperties.getRedisPort()));
		}
	}

Since we're using a different VM than previously, we need to export our JAVA_HOME environment variable `export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64`{{execute}}.
Now run `mvn clean install -DskipTests`{{execute}} to make the executable.  Notice that we are skipping tests for now, we'll address that later.

Once you see `Build Successful`, you can continue with the following commands

Start a redis server in the background with the docker command `docker run --name myredis -p 6379:6379 -e ALLOW_EMPTY_PASSWORD=yes bitnami/redis:latest &`{{execute}}

Run`java -jar target/rest-service-0.0.1-SNAPSHOT.jar &`{{execute}} in the background.

Run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message with an increasing counter.  You can stop and start the web application and the count will be saved in redis until redis is restarted.
	
So, in this example, we skipped the tests on compilation.  If we kill the redis server with `docker stop myredis && docker rm myredis`{{execute}} and our web service with `pkill -9 java`{{execute}} and then  run `mvn test`{{execute}}, we'll have test failures.  We don't want our unit tests to depend on externally running services, so in the next step, we'll learn how to use Spring application.properies to swap out redis backing services based on where we are running the web service.



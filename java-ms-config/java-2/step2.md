In this step, we're going to implement an in-memory Redis database to use for unit testing.  Since we don't want to recompile each time we change environments, we're going to externalize the database configuration using Spring Boot's application properties, like we did in the last course [https://www.katacoda/ng-dloring/java-ms-config/java-1].

There's another version of the project for this step `cd .. && wget https://raw.githubusercontent.com/drloring/katacoda-resources/main/java-ms-config-java-2-step-2.zip && unzip java-ms-config-java-2-step-2.zip && cd step-2`{{execute}}.

The first thing we did was modify our maven pom file to pull in the embedded redis server in `step-2/pom.xml`{{open}} by adding 
`		<dependency>
			<groupId>it.ozimov</groupId>
			<artifactId>embedded-redis</artifactId>
			<version>0.7.2</version>
		</dependency>`

Next, we created a couple new files to externalize our application properties for test and production `step-2/src/test/resources/application.properties`{{open}} and `step-2/src/main/resources/application.properties`{{open}}.  

We also created a RedisProperties class `step-2/src/main/java/com/example/restservice/RedisProperties.java`{{open}}, which loads the properties defined in `application.properties` into a class using the `@Value` annotation.

Then we added the embedded redis server to our unit tests `step-2/src/main/test/com/example/restservice/GreetingControllerTests.java`{{open}}.
Notice this section <pre>
	@PostConstruct
	public  void postConstruct() {
		redisServer = new RedisServer(redisProperties.getRedisPort());
	}

	@BeforeEach
	public void start() {
		redisServer.start();
	}

	@AfterEach
	public  void stop() {
		redisServer.stop();
	}

	private RedisServer redisServer;
</pre>
where we start and stop the embedded redis server before and after each test.

Now if we run `mvn clean install`{{execute}}, we'll see that the unit tests don't fail and aren't reliant on a redis container running.

If you previously killed the redis container, then run `docker run --name myredis -d --rm -p 6379:6379 redis`{{execute}} again, then run `pkill -9 java`{{execute}} then `java -jar target/rest-service-0.0.1-SNAPSHOT.jar &`{{execute}} to run it in the background.

Run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message with an increasing counter.  Notice that we added a Local Cound and a Cached count to differentiate between local state and state stored in the redis database.  You won't see any difference unless you restart the application.
	
In the next step, we'll scale out the microservice in a kubernetes cluster and verify the operations in the cluster and observe the load-balancing aspect of clustered services.

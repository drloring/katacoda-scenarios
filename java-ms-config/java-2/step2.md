In this step, we're going to implement an in-memory Redis database to use for unit testing.  Since we don't want to recompile each time we change environments, we're going to externalize the database configuration using Spring Boot's application properties, like we did in the last course [https://www.katacoda/ng-dloring/java-ms-config/java-1].

The first thing we have to do is to modify our gradle build file to pull in the embedded redis server gs-rest-service/complete/build.gradle`{{open}}.  Add <pre class="file" data-filename="gs-rest-service/complete/build.gradle" data-target="insert" data-marker="  implementation 'io.lettuce:lettuce-core'">  implementation 'io.lettuce:lettuce-core'  
  testImplementation('it.ozimov:embedded-redis:0.7.2')</pre>

Next, we have to create a couple new files to add our application properties, create the directory  `mkdir src/test/resources`{{execute}} and the application.properties file `touch src/test/resources/application.properties`{{execute}} and we'll also create a RedisProperties class `touch src/main/java/com/example/restservice/RedisProperties.java`{{execute}}.

Now, we'll add the properties to application.properties <pre class="file" data-filename="gs-rest-service/complete/src/test/resources/application.properties" data-target="prepend">spring.redis.host=localhost
spring.redis.port=6370</pre> and we'll create a RedisProperties class <pre class="file" data-filename="gs-rest-service/complete/src/main/java/com/example/restservice/RedisProperties.java" data-target="prepend">package com.example.restservice;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RedisProperties {
    private int redisPort;
    private String redisHost;

    public RedisProperties(
      @Value("${spring.redis.port}") int redisPort, 
      @Value("${spring.redis.host}") String redisHost) {
        this.redisPort = redisPort;
        this.redisHost = redisHost;
    }

	public int getRedisPort() {
		return redisPort;
	}

	public String getRedisHost() {
		return redisHost;
	}

    
}</pre>

Now we'll add the embedded redis server to our unit tests <pre class="file" data-filename="gs-rest-service/complete/src/main/test/com/example/restservice/GreetingControllerTests.java" data-target="prepend">package com.example.restservice;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import javax.annotation.PostConstruct;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

import redis.embedded.RedisServer;

@SpringBootTest
@AutoConfigureMockMvc
public class GreetingControllerTests {

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

	@Autowired
	private RedisProperties redisProperties;

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void noParamGreetingShouldReturnDefaultMessage() throws Exception {

		this.mockMvc.perform(get("/greeting")).andDo(print()).andExpect(status().isOk())
				.andExpect(jsonPath("$.content").isString());
	}

	@Test
	public void paramGreetingShouldReturnTailoredMessage() throws Exception {

		this.mockMvc.perform(get("/greeting").param("name", "Spring Community")).andDo(print())
				.andExpect(status().isOk()).andExpect(jsonPath("$.content").isString());
	}

}
</pre>

	
Run`./gradlew bootRun &`{{execute}} in the background.

Run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message with an increasing counter.  You can stop and start the web application and the count will be saved in redis until redis is restarted.
	
So, we cheated by running gradle bootRun before running gradle build, if we run `./gradlew build`{{execute}} before we have the redis server running, we'll have test failures.  We don't want our unit tests to depend on externally running services, so in the next course, we'll learn how to use Spring application.properies to swap out redis backing services based on where we are running the web service.

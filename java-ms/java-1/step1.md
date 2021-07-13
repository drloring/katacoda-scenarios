Welcome to the Java Web Services example

To get started, clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{execute}}

CD to the complete directory `cd gs-rest-service/complete`{{execute}}

Run the gradle command to get all tasks available `gradle tasks`{{execute}}

Identify the task to run the boot application `gradle bootRun &`{{execute}}

After a while, run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message.

Now run `curl http://localhost:8080/greeting?name=User`{{execute}} to demonstrate request parameters

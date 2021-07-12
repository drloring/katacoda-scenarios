Welcome to the Java Web Services example

To get started, clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{execute interrupt}}

CD to the complete directory `cd gs-rest-service/complete`{{execute interrupt}}

Run the gradle command to get all tasks available `gradle tasks`{{execute interrupt}}

Identify the task to run the boot application `gradle bootRun &`{{execute interrupt}}

After a while, run the following command to verify that the spring boot application is running `http://localhost:8080/greeting`{{execute interrupt}} to display the Hello World message.

Now run `http://localhost:8080/greeting?name=User`{{execute interrupt}} to demonstrate request parameters

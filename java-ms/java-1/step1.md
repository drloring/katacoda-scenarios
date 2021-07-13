Welcome to the Java Web Services example

To get started, clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{copy}}

CD to the complete directory `cd gs-rest-service/complete`{{copy}}

Run the gradle command to get all tasks available `gradle tasks`{{copy}}

Identify the task to run the boot application `gradle bootRun &`{{copy}}

After a while, run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{copy}} to display the Hello World message.

Now run `curl http://localhost:8080/greeting?name=User`{{copy}} to demonstrate request parameters

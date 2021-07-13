Welcome to the Java Web Services example project.  This project is explained [here](https://spring.io/guides/gs/rest-service/).

To get started, clone the Spring Boot Web Service project at `git clone https://github.com/spring-guides/gs-rest-service.git`{{execute}}

CD to the gs-rest-service directory `cd gs-rest-service/`{{execute}}

Perform an `ls`{{execute}} to see the directory listing.  Notice there are two directories, initial and complete.  We'll be working with the complete example, if you have time and want to follow along with the website listed above, perform the steps below from the initial directory and then follow up with the complete directory.

CD to the complete directory `cd complete/`{{execute}}

Run the gradle command to get all tasks available `gradle tasks`{{execute}}

Identify the task to run the boot application `gradle bootRun &`{{execute}}.  Note: the ampersand after the command runs the command in the background and allows you to continue to enter commands from the command line.

Run the following command to verify that the spring boot application is running `curl http://localhost:8080/greeting`{{execute}} to display the Hello World message in a json formatted string.

Now run `curl http://localhost:8080/greeting?name=User`{{execute}} to demonstrate how request parameters can be parsed and returned in the response json string.



Now that we can build the docker image and deploy with helm automatically, we're going to automate the maven build so that we can have skaffold automatically deploy when we make changes to out application code and configurations.  First, kill the `skaffold dev` command with `Ctrl + C`

Open the maven POM in `java-1-step-1/pom.xml`{{open}} and add the following plugin to the build plugins list:

  <plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
      <to>
        <image>java-ws</image>
      </to>
    </configuration>
</plugin>

Once that's complete, you can verify that is working by running `mvn compile jib:dockerBuild`{{execute}}.  With this configuration, and the command line, we're using Jib to build the OCI-compliant image and installing in the local docker engine.  If you didn't have docker available, you could just run `mvn compile jib:build`{{execute}} and bypass the docker engine altogether.  For this example, however, we want it installed in our docker engine.

Run `docker images | grep java`{{execute}} and notice the difference in image sizes.  Jib uses jdk 8 by default, but is overridable in the `skaffold.yaml` file.

If it's not already open, then open `java-1-step-1/skaffold.yaml`{{open}} and remove the `#` signs from the yaml.  You'll notice that we've setup a custom build that executes the command you previously tested.  Now, if you run `skaffold build`{{execute}}, you'll see that maven is integrated into the build.

Once that works, run `skaffold dev`{{execute}} to monitor for changes.  To verify it's working, open `java-1-step-1/src/main/java/com/example/restservice/GreetingController.java`{{open}}, change `Hello` to `Hola` and watch the service get rebuilt and redeployed automatically.

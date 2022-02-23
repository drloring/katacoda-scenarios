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

Once that's done, you can verify the plugin is working by running `mvn compile jib:dockerBuild`{{execute}}.  With this configuration, and the command line, we're using Jib to build the OCI-compliant image and installing in the local docker engine.  If you didn't have docker available, you could just run `mvn compile jib:build` and bypass the docker engine altogether, however that requires login credentials to the public docker registry.  For this example, we want it installed in our local docker engine.

Run `docker images | grep java`{{execute}} and notice the difference in image sizes.  Jib uses jdk 8 by default, but is overridable in the `skaffold.yaml` file.  Running `docker history java-ws`{{execute}} shows the pedigree of the image with the jib builder.

If it's not already open, then open `java-1-step-1/skaffold.yaml`{{open}} and remove the `#` signs from the yaml.  You'll notice that we've setup a custom build that executes the command you previously tested.  Now, if you run `skaffold build`{{execute}}, you'll see that maven is integrated into the build.

Once that works, run `skaffold dev`{{execute}} to monitor for changes.  To verify it's working, open `java-1-step-1/src/main/java/com/example/restservice/GreetingController.java`{{open}}, change `Hello` to `Hola` and watch the service get rebuilt and redeployed automatically.  In a separate window you can export your `NODE_IP` and `NODE_PORT` with `export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")`{{execute}} and `export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services ws)`{{execute}}, respectively.  Then `curl $NODE_IP:$NODE_PORT/greeting`{{execute}} to verify.

Skaffold eases debugging your application inside the cluster as well.  Running `skaffold debug` works much like `skaffold dev` with the addition that it detects what type of app is built, and adds remote debugging to the startup of the container.  It also port-forwards the debugging port so that you can connect to it with your IDE.

I hope you found this course interesting and can find a use to include skaffold in the next cloud-native project you embark on!

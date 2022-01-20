In this scenario, we're going to see how we can pass configurations in from the docker file and docker runtime environment.

One way we could change the configuration of the service is to modify the Dockerfile to supply the changes like we did from the command line in the previous scenario.  If we look at `java-1-step-1/Dockerfile.1`{{open}}, we see a similar command on the line
<pre>
CMD java -jar svc.jar --server.port=9999
</pre>

To test this out, run `docker build -t java-ws -f Dockerfile.1 .`{{execute}}, then `docker run --name jws --rm -d -p9999:9999 java-ws`{{execute}} and finally `curl http://localhost:9999/greeting?name=User`{{execute}} to verify it is exposed on port 9999.

That is a pretty inflexible way since the configuration is baked into the image.  There may be scenarios where you want individual docker images for various environments, but in general, that approach is too restrictive.

Another way we can change configuration options at runtime is to set environment variables when we run the container and let the Docker runtime pass those along to the running service. 

First we have to stop our currently running container `docker container stop $(docker container ls -q --filter name=jws)`{{execute}}

If we look at the original `java-1-step-1/Dockerfile`{{open}}, we see the `--server.port=9999` has been removed.  Also notice that we're exposing port 9090 in the Dockerfile.  Even though you can override it, it's best practice to keep the EXPOSE parameter current with the default parameter.  This is how you communicate the intention through the Dockerfile.

Let's rebuild our image with the original Dockerfile now `docker build -t java-ws .`{{execute}}.

Now, we'll add the --env or -e parameters to the docker command line to override the default behavior in the service `docker run --name jws --rm -d -p9999:9999 -e server.port=9999 java-ws`{{execute}} and  `curl http://localhost:9999/greeting?name=User`{{execute}} to see the effects.  Notice the difference between exposing ports and port mapping.  Even though our Dockerfile exposes port 9090, we can override that with the -p option from the command line.

Remember to stop our currently running container `docker container stop $(docker container ls -q --filter name=jws)`{{execute}}

In the next scenario, we going to see how we can use the container environment overrides to map to kubernetes configmaps.



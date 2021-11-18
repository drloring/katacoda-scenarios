Let's get the docker image that we created in past courses `wget https://github,,,`{{execute}}.

Next, we'll load that docker image into our local repository `docker load -i java-ms.tar`{{execute}}

Now we can exec back into the Anchore CLI pod `kubectl exec -it ancore-cli -- bash`{{execute}} and add the image `anchore-cli add image java-ms`{{execute}}

Once we see that the status is `analyzed` with `anchore-cli image list`{{execute}}.




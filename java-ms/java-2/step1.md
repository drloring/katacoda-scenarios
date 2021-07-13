In this step, we're going to take the web service we created in the previous lesson and package it into a Docker image.

First, we're going to collect the executable jar we made in the last lesson `curl https://github.com/drloring/katacoda-resources/raw/main/rest-service-0.0.1-SNAPSHOT.jar`{{execute}}.

Now, we'll create an empty Dockerfile `touch Dockerfile`{{execute}}

Open the `Dockerfile`{{open}} in the editor 

<pre class="file" data-filename="Dockerfile" data-target="prepend">FROM ubuntu</pre>

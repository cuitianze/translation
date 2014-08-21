#### http://blog.tutum.co/2014/08/07/using-cadvisor-to-monitor-docker-containers/


### Using cAdvisor to Monitor Docker Containers


Google first created cAdvisor to monitor their own lmctfy containers and added Docker container support (those that use the default lib container execdriver). cAdvisor is short for Container Advisor. You can use cAdvisor to get a detailed look into the resource usage and the various performance characteristics of your running containers. cAdvisor includes a simple UI to view the live data, a simple API to retrieve the data programmatically, and the ability to store the data in an external InfluxDB.

We’ll be taking an introductory look into cAdvisor.

If you want to monitor all of the containers you have running on a host, you can deploy the cAdvisor image as a container in that same host. It will then have access to the resource usage and performance characteristics of your other containers in that host.

You can try out cAdvisor as long as you’re running Docker version 0.11 or above. Google has created a Docker image with everything you need to get started. You can do this by running:

sudo docker run \

–volume=/var/run:/var/run:rw \

–volume=/sys:/sys:ro \

–volume=/var/lib/docker/:/var/lib/docker:ro \

–publish=8080:8080 \

–detach=true \

–name=cadvisor \

google/cadvisor:latest

It’s that simple. If everything goes as expected you’ll now have another container running alongside your other Docker containers. You can check cAdvisor is running by pointing your browser to the host’s IP or hostname and port 8080. If running locally, try http://localhost:8080. This will bring up the built-in Web UI.

To put cAdvisor to a test we deployed a Unixbench Docker container alongside the cAdvisor container. To try that, you can simply execute:

docker run -d borja/unixbench
Below is a screenshot of what cAdvisor’s Web UI look like.

 

cAdvisor monitoring Docker container

 

This provides a high level overview of the resources being used on the host. Clicking on /docker in Subcontainers takes you to a new window with all of the Docker containers listed individually.

However, cAdvisor can be fairly limited in this capacity. It samples once a second with only 1 minute worth of historical data.

In order for this information to be useful we need to access this data programmatically. Luckily, cAdvisor exposes its raw and processed stats via a versioned remote REST API. This is accessible here:

http://<hostname>:<port>/
To access container information, the resource name is as follows:

/api/v1.1/subcontainers/
All of this container information is returned as a JSON object containing:

Absolute container name
List of subcontainers
ContainerSpec which describes the resource isolation enabled in the container
Detailed resource usage statistics of the container for the last N seconds (N is globally configurable in cAdvisor)
Histogram of resource usage from the creation of the container
We currently use our own in-house developed solution for monitoring at Tutum. However, we think cAdvisor is great, with active development, and a roadmap that excites us. We’re looking to integrate it with our system and contribute back to this exciting project in the Docker ecosystem.  Here are some features in its roadmap:

Advise on the performance of a container (e.g.: when it is being negatively affected by another, when it is not receiving the resources it requires, etc)
Auto-tune the performance of the container based on previous advise
Provide usage prediction to cluster schedulers and orchestration layers.
In another article we’ll be showing how to import all of the live data into InfluxDB in order to access historical data. So stay tuned! Make sure to subscribe to our blog so you don’t miss any new articles.

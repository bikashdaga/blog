---
toc: true
layout: post
description: "Run your Jupyter Notebook as a stand alone web app."
categories: [markdown, data science, jupyter, ml]
comments: true
image: images/jupyterhub/hublogo.svg
title: "Tutorial: Stop Running Jupyter Notebooks from your Command Line!"
---

Jupyter Notebooks provide a great platform to produce human-readable documents containing code, equations, analysis their descriptions. Some consider it a powerful tool for development when combining it with NBDev. For such an integral tool, the out of the box start up is not the best.  Each use requires starting the Jupyter web application from the command line and entering your token or password. The entire web application relies on that terminal window being open. Some might "daemonize" the process and then use [nohup](https://www.computerhope.com/unix/unohup.htm) to detach it from their terminal, but that's not the most elegant and maintainable solution.

Lucky for us, Jupyter has already come up with a solution to this problem by coming out with an extension of Jupyter Notebooks that runs as a sustainable web application and has built in user authentication. To add a cherry on top it can be managed and sustained through Docker allowing for isolated development environments. 

By the end of this post we will have a Jupyter Notebook instance which can be accessed on command without a terminal, from multiple devices within your network, and a much more user friendly authentication method.

## Prerequisites
---
The technical requirements are basic Docker and command line skills.

Regarding hardware, I recommend doing this on the most powerful device you have and one that is on for most of the day, preferably all day. One of the benefits of this setup is that you will be able to use the Jupyter Notebook from any device on your network, but have all the computation happen on the device we configure.

## What is Jupyter Hub
---
JupyterHub brings the power of notebooks to groups of users. The idea behind JupyterHub was to scale out the use of Jupyter Notebooks to enterprises, classrooms, and large groups of users. On the other hand, Jupyter Notebook is supposed to run in a local instance, on a single node, by a single developer. Unfortunately, there was no middle ground to have the usability and scalability of JupyterHub and the simplicity of running a local Jupyter Notebook. That is, until now.

JupyterHub has pre-built Docker images that we can utilize to spawn a single notebook on a whim, with little to no overhead in technical complexity. We are going to use the combination of Docker and JupyterHub to access Jupyter Notebooks from anytime, anywhere at the same URL.

## Architecture
---
At a high level the architecture is not complex and can even scale if you have more than one user you want to set this up for!

1) Type in the IP address or host name of the machine JupyterHub is running on.

2) Authenticate using username & password

3) Start your server & you're good to go!

![]({{ site.baseurl }}/images/jupyterhub/architecture.png)

## Building the Docker Images
---
To build our at home JupyterHub server we will use the pre-built Docker images of JupyterHub & JupyterLab.

### JupyterHub

#### Docker

The JupyterHub Docker image is simple.

```Dockerfile
FROM jupyterhub/jupyterhub:1.2

# Copy the JupyterHub configuration in the container
COPY jupyterhub_config.py .

# Download script to automatically stop idle single-user servers
COPY cull_idle_servers.py .

# Install dependencies (for advanced authentication and spawning)
RUN pip install dockerspawner
```

We use the pre-built JupyterHub Docker Image but then add our own configuration file along side code to stop idle servers, `cull_idle_servers.py`. Lastly, we install additional packages to spawn JupyterLab instances via Docker.

#### Docker Compose

To bring everything together, let's create a `docker-compose` file to define our deployments and configuration.

```Dockerfile
version: '3'

services:
  # Configuration for Hub+Proxy
  jupyterhub:
    build: .                # Build the container from this folder.
    container_name: jupyterhub_hub   # The service will use this container name.
    volumes:                         # Give access to Docker socket.
      - /var/run/docker.sock:/var/run/docker.sock
      - jupyterhub_data:/srv/jupyterlab
    environment:                     # Env variables passed to the Hub process.
      DOCKER_JUPYTER_IMAGE: jupyter/tensorflow-notebook
      DOCKER_NETWORK_NAME: ${COMPOSE_PROJECT_NAME}_default
      HUB_IP: jupyterhub_hub
    ports:
      - 8080:8080
    restart: unless-stopped

  # Configuration for the single-user servers
  jupyterlab:
    image: jupyter/tensorflow-notebook
    command: echo

volumes:
  jupyterhub_data:
```

The key environment variable to note is the `DOCKER_JUPYTER_IMAGE`. This is the Jupyter environment that will be spun up every time we access the server.

For more information on selecting Jupyter images you can visit the following Jupyter [documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html).

#### Stopping Idle Servers

Since this is our home setup, we want to be able to stop idle instances to preserve memory on our machine. JupyterHub has services that can run along side itself and one of them is [jupyterhub-idle-culler](https://github.com/jupyterhub/jupyterhub-idle-culler). This service stops any instances that are idle for a prolonged duration.

Create a new file called `cull_idle_servers.py` and copy the contents of this file from the [jupyterhub-idle-culler project](https://raw.githubusercontent.com/jupyterhub/jupyterhub-idle-culler/master/jupyterhub_idle_culler/__init__.py) into it.


{% include info.html text="Ensure `cull_idle_servers.py` is in the same folder as the Dockerfile." %}

To find out more about services, JupyterHub has official [documentation](https://jupyterhub.readthedocs.io/en/stable/reference/services.html) on them.

#### Jupyterhub Config

To finish off, we need to define configuration options for our JupyterHub instance. For example, volume mounts, Docker images, services, authentication, etc.

Below is a simple configuration I use that gets the job done.

```python
import os
import sys

c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'
c.DockerSpawner.image = os.environ['DOCKER_JUPYTER_IMAGE']
c.DockerSpawner.network_name = os.environ['DOCKER_NETWORK_NAME']
c.JupyterHub.hub_connect_ip = os.environ['HUB_IP']
c.JupyterHub.hub_ip = "0.0.0.0" # Makes it accessible from anywhere on your network

c.JupyterHub.admin_access = True

c.JupyterHub.services = [
    {
        'name': 'cull_idle',
        'admin': True,
        'command': [sys.executable, 'cull_idle_servers.py', '--timeout=42000']
    },
]

c.Spawner.default_url = '/lab'

notebook_dir = os.environ.get('DOCKER_NOTEBOOK_DIR') or '/home/jovyan/work'
c.DockerSpawner.notebook_dir = notebook_dir
c.DockerSpawner.volumes = {
    '/home/sidhu': '/home/jovyan/work'
}
```

The configuration options you want to take note of and change are the following:

- `'command': [sys.executable, 'cull_idle_servers.py', '--timeout=42000']` : Timeout is the number of seconds until an idle Jupyter instance is shut down.

- `c.Spawner.default_url = '/lab'`: Uses Jupyterlab instead of Jupyter Notebook. Comment out this line to use Jupyter Notebook.

- `'/home/sidhu': '/home/jovyan/work'`: I mounted my home directory to the Jupyter home directory to have access to any projects and notebooks I have on my Desktop already. This also allows us to achieve persistence in the case we create new notebooks, they are saved to our local machine and will not get deleted if our Jupyter Notebook Docker container deleted. **Remove this line if you do not wish to mount your home directory and do not forget to change `sidhu` to your user name**.

## Start the Server
---

To start the server, simply run `docker-compose up -d` - navigate to `localhost:8080` in your browser and you should be able to see the JupyterHub landing page.

![]({{ site.baseurl }}/images/jupyterhub/landingpage.png)


To access it on other devices on your network - a laptop, an iPad, etc, first identify the IP of the host machine by running `ifconfig` on Unix machines & `ipconfig` on windows.

![]({{ site.baseurl }}/images/jupyterhub/ifconfig.png)

From your other device, navigate to the IP you found on port 8080: `http://IP:8080` and you should see the JupyterHub landing page!

### Authenticating

That leaves us with the last task of authenticating to the server. Since we did not set up a LDAP server or OAuth, JupyterHub will use PAM (Pluggable Authentication Module) authentication to authenticate users. In English, JupyterHub uses the user name and passwords of of the host machine to authenticate. To make use of this, we are going to have to create a user on the JupyterLab Docker container. There are other ways of doing this such as having a script placed on the container and executed at container start up but we will do it manually as an exercise. It should be noted that if you tear down or rebuild the container that you will have to repeat this process, so an exercise to automate this process or switch to a different authentication method would be invaluable.

{% include alert.html text="I do not recommend hard coding user credentials into any script or Dockerfile." %}

1) Find the JupyterLab container ID: `docker ps -a`

![]({{ site.baseurl }}/images/jupyterhub/containerid.png)

2) "SSH" into the container: `docker exec -it $YOUR_CONTAINER_ID bash`

3) Create a user and follow the terminal prompts to create a password: `useradd $YOUR_USERNAME`

4) Sign in with the credentials and you're all set!

![]({{ site.baseurl }}/images/jupyterhub/jupyterlab.png)

## Feedback
---
I encourage any and all feedback about any of my posts and tutorials. You can message me on [twitter](https://twitter.com/ashtonasidhu) or e-mail me at sidhuashton@gmail.com.

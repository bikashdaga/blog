---
toc: true
description: Run your Jupyter Notebook as a stand alone web app.
categories: [markdown, data science, jupyter, ml]
comments: true
image: images/jupyterhub/intro.jpg
---

# TITLE HERE

Jupyter is a pivotal tool for data exploration, experimentation and even development when combining it with NBDev. For such an integral tool of the Data Science and for some, the development work flow, the out of the box usability is not the best. Every use it requires starting the Jupyter web application from the command line and entering your token or your password. The entire web application relies on that terminal window being open. Some might daemonize the process and then use [nohup]() to detach it from their terminal, but that's not the most elegant and maintainable solution.

Lucky for us, Jupyter has already come up with a solution to this problem by coming out with an extension of the Jupyter web application that runs as a sustainable web application server and built in user authentication inherited from the machine it is running on. To add a cherry on top it can even be ran, managed and sustained through Docker allowing for isolated development environments. 

By the end of this post we will have a sustainable Jupyter Lab instance which can be accessed on command without a terminal, from multiple devices within your network, and a much more convenient authentication method.

## Prerequisites

All you will need to know to set this up is basic knowledge and usage of Docker and optionally, basic networking if you have internal DNS resolution in your network.

In terms of hardware, I recommend doing this on the most powerful device you have and one that is preferentially on for most of the day, if not all day. One of the benefits of this setup is that you will be able to use Jupyter notebooks from any device on your network but have all the computation happen on the one device we setup.

## What is Jupyter Hub

Jupyter Hub

## Architecture

Draw diagram on surface or find online

## Building the Docker Images

Explain files in home made docker images
Give repo

## OPTIONAL: Internal DNS Config

### Reverse Proxy

### Websockets

## Conclusion


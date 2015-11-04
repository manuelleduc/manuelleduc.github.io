---
layout: post
title:  "Docker compose and HAProxy"
date:   2015-11-04 22:28:36
categories: docker compose haproxy scalability
---
More of a reminder than anything but here is a simple docker-compose.yml file
which starts a haproxy container and a single nginx container.
We will see afterward how to scale the number of nginx instance.


**docker-compose.yml**
{% highlight yaml %}
loadbalancer:
    image: tutum/haproxy
    ports:
      - "8045:80"
    links:
      - nginx
nginx:
  image: nginx
{% endhighlight %}

1. Installing [docker](https://docs.docker.com/engine/installation/) and [docker compose](https://docs.docker.com/compose/install/).
2. Move next to **docker-compose.yml**
3. Starting the containers

{% highlight bash %}
$ docker-compose up -d
Creating protohaproxy_nginx_1
Creating protohaproxy_loadbalancer_1
{% endhighlight %}

4. Checking docker status

{% highlight bash %}
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                     NAMES
550277fd7d64        tutum/haproxy       "python /haproxy/main"   About a minute ago   Up About a minute   443/tcp, 1936/tcp, 0.0.0.0:8045->80/tcp   protohaproxy_loadbalancer_1
836a338264f0        nginx               "nginx -g 'daemon off"   About a minute ago   Up About a minute   80/tcp, 443/tcp                           protohaproxy_nginx_1
{% endhighlight %}

At this point you can access http://localhost:8045/ and see the following result.

> ![](/images/2015-11-04-docker-compose-haproxy-1.png)

5. Consulting `docker-machine logs` while reloading http://localhost:8045/ will reveal that every request go through *protohaproxy_loadbalancer_1* and then handled by *protohaproxy_nginx_1*

6. We can now scale the number of nginx instances behind the loadbalancer by using the scale command.

{% highlight bash %}
docker-compose scale nginx=3
Creating and starting 2 ... done
Creating and starting 3 ... done
{% endhighlight %}

*protohaproxy_nginx_2* and *protohaproxy_nginx_3* are now listed.
{% highlight bash %}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                     NAMES
b4c798f514a2        nginx               "nginx -g 'daemon off"   57 seconds ago      Up 56 seconds       80/tcp, 443/tcp                           protohaproxy_nginx_2
b32b426d5d13        nginx               "nginx -g 'daemon off"   57 seconds ago      Up 56 seconds       80/tcp, 443/tcp                           protohaproxy_nginx_3
550277fd7d64        tutum/haproxy       "python /haproxy/main"   13 minutes ago      Up 13 minutes       443/tcp, 1936/tcp, 0.0.0.0:8045->80/tcp   protohaproxy_loadbalancer_1
836a338264f0        nginx               "nginx -g 'daemon off"   13 minutes ago      Up 13 minutes       80/tcp, 443/tcp                           protohaproxy_nginx_1
{% endhighlight %}


Now protohaproxy_loadbalancer_1 have 3 services available in backed. By stressing http://localhost:8045/ while monitoring the logs you'll see that a we have now a loadbalancing system running on our machine.

Links :

 - [tutom/haproxy container](https://hub.docker.com/r/tutum/haproxy/)
 - [nginx container](https://hub.docker.com/r/_/nginx/)

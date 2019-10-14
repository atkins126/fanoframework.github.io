---
title: Deploy Fano application with Apache load balancer module
description: Tutorial on how to deploy Fano application with Apache mod_proxy_balancer.
---

<h1 class="major">Deploy Fano application with Apache load balancer module</h1>

Apache provides reverse proxy load balancer to distribute load to one or more application servers with goal to improve performance and scalability.

## Requirement

- [Apache 2.4](http://httpd.apache.org/docs/2.4/)
- [mod_proxy_balancer](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html)
- [mod_proxy_scgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_scgi.html)
- [mod_proxy_fcgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html)

## Setting up mod_proxy_balancer

Make sure that `mod_proxy_balancer` is enabled. For example, in Debian-based, run

```
$ sudo a2enmod proxy_balancer
```

## Deploy Fano Application with load balancer with Fano CLI

Apart from task for [scaffolding web application](/scaffolding-with-fano-cli),
[Fano CLI](https://github.com/fanoframework/fano-cli) also provides `--deploy-lb-scgi=[domain name]` and `--deploy-lb-fcgi=[domain name]` to help setup load balancer during development for SCGI and FastCGI web application, respectively.

Inside Fano SCGI web application project directory, run

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano
```

Command above, will create virtual host for Apache web server that utilize `mod_proxy_balancer` module, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.fano` domain in `/etc/hosts`.

Replace with `--deploy-lb-fcgi` for setting up FastCGI web application with load balancer.

## Running multiple application with load balancer

Build the application and then run two applications at once with consecutive listening ports.

```
$ ./bin/app.cgi --port=20477 & ./bin/app.cgi --port=20478 &
```

`&` is required to make sure that both applications is running in background.

## Access application from browser

Open `http://myapp.fano` you should see main controller is invoked. There is no visual indication compare to application running without load balancer. Indication that your application is running with `mod_proxy_balancer` is availability of new environment variables, for examples `BALANCER_NAME`, `BALANCER_WORKER_NAME` and `BALANCER_WORKER_ROUTE`. Try to access route that does not exist, so that environment variables are printed on the browser.

## Change number of balancer member

By default, if parameter `--total-member` not set, it is assumed that you will use two application instances. Set `--total-member` according to your need.

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --total-member=4
```

Run four applications at once with consecutive listening ports.

```
$ ./bin/app.cgi --port=20477 & ./bin/app.cgi --port=20478 & \
./bin/app.cgi --port=20479 & ./bin/app.cgi --port=20480 &
```

## Change start listening port

By default, if parameter `--port` not set, 20477 is used as starting port. Set `--port` according to your need.

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --port=8080 --total-member=4
```

Run four applications at once with consecutive listening ports.

```
$ ./bin/app.cgi --port=8080 & ./bin/app.cgi --port=8081 & \
./bin/app.cgi --port=8082 & ./bin/app.cgi --port=8083 &
```

Developer is responsible to make sure that no other application is using those listening ports.

## Change load balancing scheduler algorithm

By default, if parameter `--lbmethod` is not set, `byrequests` is assumed, which will use `mod_lbmethod_byrequests` module to distribute requests. Please refer to [mod_proxy_balancer documentation](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html) for available algorithms.

For example, to use `mod_lbmethod_bybusiness` module,

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --lbmethod=bybusiness
```

## Explore more

- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Back to Deployment](/deployment)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
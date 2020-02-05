# README

This a code repository template. You can fork it and use it as a base for your service(s) or download it and fit it into your existing repository.

## code repository template

This is the start of the journey with your stack.

This stack is meant to be used with Docker and docker-compose to provide teams with a standard way to have :
- web service code base (what ever you want)
- markdown documentation repository (this is a jekyll based setup)
- databases services (edit the docker-compose to your liking for this)

You could have one single service code base in the _code_ folder but you could also have more and use this as a single repository setup for all your services and possibly infrastructure too.

We see it as a way to get everything the team needs into a single place with git versionning it all. And if you use git-appraise then you even get code reviews within.

### ./code

One or more services code bases. Do what you want in this one.

### ./documentation

Technical documentation for the service(s) and possibly including your team code of conduct, team charter, coding style, deployment and release processes documentation, ...

### docker-compose.yml

Docker is a handy way to get services dependencies running. The default `docker-compose.yml` included in this repository comes with a few services :
- service_haproxy : an haproxy instance to more easily use the service container and the documentation one at least, if not more. The configuration file lives in the `haproxydata` folder check it out to figure out how to extend for more services
- service_db : configured to be an instance of PostgreSQL. It's sturdy, open source, and many use it. Its data will live in the `pgdata` folder so that you don't lose everything upon each stop and start of the container
- service_redis : a redis instance; mostly as an example of something else here, but often useful
- service_devdocs : a jekyll based "simply documentation" container with live reloading and so on. Document as much as you want through markdown files in the `documentation` folder, they will appear automatically in your doc web host.
- service_web : your actual service. It's expected to have a Dockerfile and to be able to run. The default includes a Rails 6 friendly setup but you can alter that to your needs

## haproxy config and hosts

HaProxy is used to simplify your life here : no specific port to remember, you can use hostnames, if you adjust the configuration in a few files.

### haproxy config

While a bit intimidating it's actually fairly straight forward. The `haproxydata` folder contains an example to start from.

If you want a web service and the documentation to be pointed to with using hostnames such as web.example.org.local and doc.example.org.local you should have the following :

```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	daemon

defaults
	log	global
	mode	http
	option httplog
	option dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000

frontend web
    bind *:8000
    mode http
    acl web hdr_sub(host) -i web.example.org.local
    acl doc hdr_sub(host) -i doc.example.org.local
    use_backend web if web
    use_backend documentation if doc
    option httpclose
    default_backend web 

backend web
  timeout server 4h
  balance leastconn
  server web1 172.17.0.1:3000

backend documentation
  timeout server 4h
  balance leastconn
  server doc1 172.17.0.1:4000
```

So alter the hostnames according to your needs.

It's good to note that the ip address used for the backends are the one of the docker host. So depending on your settings you might want to change that too.

### /etc/hosts edit

Once you have made the changes in the haproxy config file you want to edit your /etc/hosts file and add entries pointing to the right place :

```
0.0.0.0 web.example.org.local doc.example.org.local
```

### setup needed for documentation

Finally, if you want everything to stay on doc.example.org.local you will want to change the `_config.yml` file in the `documentation` folder.

```
title: Documentation
email: changeme@example.org
description: >- # this means to ignore newlines until "baseurl:"
  This is the internal documentation for the ChangeMe team
baseurl: "/" # the subpath of your site, e.g. /blog
url: "http://doc.example.org.local:8000"

markdown: kramdown
theme: just-the-docs
plugins:
  - jekyll-feed

search_enabled: false

```

`url` should be set correctly depending on the hostname you decided to use for the documentation part. The port should be the one you picked for the haproxy service, which is 8000 by default.

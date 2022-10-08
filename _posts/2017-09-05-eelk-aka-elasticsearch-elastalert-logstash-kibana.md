---
layout: post
title: EELK (aka ElasticSearch, ElastAlert, Logstash, Kibana)
categories:
- weirich
tags:
- elastalert
- elasticsearch
- guide
- intro
- logstash
- pip
- python
- tutorial
- walkthrough
- yelp
---
Every project deserves security and log monitoring, but not every project can afford X-Pack.  X-Pack is Elastic's commercial companion to the open source ElasticSearch.  There's a two week free trial, but after that it's gonna cost $$$.  Luckily for us small fries, the folks at Yelp have open sourced their ElasticSearch powered alerting tool (aptly named [ElastAlert](https://github.com/Yelp/elastalert))

_Note: This guide assumes you have the ELK stack set up and running. You can follow a guide [here](https://www.google.com/search?q=elk+set+up&amp;oq=elk+set+up&amp;aqs=chrome..69i57j0l5.1574j0j7&amp;sourceid=chrome&amp;ie=UTF-8)._

So we have our data pumping into LogStash and out into ElasticSearch.  We took the first step to carefully monitoring our data, but now we need to actually, you know, monitor it.  Without X-Pack, how are we any further ahead than we were before?  ElastAlert is the answer.

_Note: I set up my ElastAlert instance on the same box as my ElasticSearch instance, but feel free to modify this guide to fit your ELK setup._

**Installation**

The first thing we have to do is install pip on our ElasticSearch instance since ElastAlert is a python based tool:
```
sudo apt install python-pip python-dev build-essential
```
The next step is to install ElastAlert:
```
pip install elastalert
```

**Configuration**

And the final (but longest) step is to configure ElastAlert:

All of the configurations are stored in the directory indicated by the <code>--config</code> flag.
For example:
```
/home/ubuntu/.local/bin/elastalert --config /home/ubuntu/elastalert-config.yaml
```
(I'd recommend creating a service following this [guide](https://www.ubuntudoc.com/how-to-create-new-service-with-systemd/).)

The first necessary configuration is to set the rules folder, this folder is where we're going to store all of our configurations for the types of alerts we'd like to get.
```
rules_folder: /home/ubuntu/rules_folder
```
The next important configuration to set is the address of our ElasticSearch instance.
```
es_host: localhost
es_port: 9200
```
Everything else can be borrowed from [here](https://github.com/Yelp/elastalert/blob/master/config.yaml.example) to start.
Now that our configuration is ready, we need to set up an index on our ElasticSearch instance to store the writeback logs that ElastAlert generates (bonus points for being meta enough to set up alerts on your alerting system).
```
elastalert-create-index --config elastalert-config.yaml
```
Congratulations, a basic ElastAlert service is now set up and configured! Add some rules to your rules folder, restart the service, and you're on your way!

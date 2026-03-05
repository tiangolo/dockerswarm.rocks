# Project Generators

There are several project generators designed to be used in a Docker Swarm mode cluster with a main Traefik HTTPS proxy, all set up with the ideas from [DockerSwarm.rocks](https://dockerswarm.rocks).

## Full Stack FastAPI PostgreSQL - project generator

Link: [https://github.com/tiangolo/full-stack-fastapi-postgresql](https://github.com/tiangolo/full-stack-fastapi-postgresql)

* Python backend with [FastAPI](https://github.com/tiangolo/fastapi).
* Frontend with [Vue.js](https://vuejs.org/).
* [Celery](http://www.celeryproject.org/) for asynchronous jobs with [RabbitMQ](https://www.rabbitmq.com/).
* [PostgreSQL](https://www.postgresql.org/) as the database.
* Email notifications.
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.

## Full Stack FastAPI Couchbase - project generator

Link: [https://github.com/tiangolo/full-stack-fastapi-couchbase](https://github.com/tiangolo/full-stack-fastapi-couchbase)

* Python backend with [FastAPI](https://github.com/tiangolo/fastapi).
* Frontend with [Vue.js](https://vuejs.org/).
* [Celery](http://www.celeryproject.org/) for asynchronous jobs with [RabbitMQ](https://www.rabbitmq.com/).
* [Couchbase](https://www.couchbase.com/) as the NoSQL database.
* Email notifications.
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.

## Full Stack Flask and PostgreSQL - project generator

Link: [https://github.com/tiangolo/full-stack](https://github.com/tiangolo/full-stack)

* Python backend with [Flask](http://flask.pocoo.org/).
* Frontend with [Vue.js](https://vuejs.org/).
* [Celery](http://www.celeryproject.org/) for asynchronous jobs with [RabbitMQ](https://www.rabbitmq.com/).
* [PostgreSQL](https://www.postgresql.org/) as the database.
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.

## Full Stack Flask Couchbase - project generator

Link: [https://github.com/tiangolo/full-stack-flask-couchbase](https://github.com/tiangolo/full-stack-flask-couchbase)

* Python backend with [Flask](http://flask.pocoo.org/).
* Frontend with [Vue.js](https://vuejs.org/).
* [Celery](http://www.celeryproject.org/) for asynchronous jobs with [RabbitMQ](https://www.rabbitmq.com/).
* [Couchbase](https://www.couchbase.com/) as the NoSQL database.
* Email notifications.
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.

## Full Stack Flask CouchDB - project generator

Link: [https://github.com/tiangolo/full-stack-flask-couchdb](https://github.com/tiangolo/full-stack-flask-couchdb)

* Python backend with [Flask](http://flask.pocoo.org/).
* Frontend with [Vue.js](https://vuejs.org/).
* [Celery](http://www.celeryproject.org/) for asynchronous jobs with [RabbitMQ](https://www.rabbitmq.com/).
* [CouchDB](http://couchdb.apache.org/) as the NoSQL database.
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.

## Flask Frontend Docker - project generator

Link: [https://github.com/tiangolo/flask-frontend-docker](https://github.com/tiangolo/flask-frontend-docker)

* Python backend with [Flask](http://flask.pocoo.org/).
* Frontend with [Vue.js](https://vuejs.org/).
* [GitLab CI](https://about.gitlab.com/product/continuous-integration/) integration.


## Terraform + Ansible + Docker Swarm boilerplate (T.A.D.S.) - project generator

Link: [https://github.com/Thomvaill/tads-boilerplate](https://github.com/Thomvaill/tads-boilerplate)

* Cloud infrastructure created with [Terraform](https://www.terraform.io/).
* Production-like environment reproduced locally with [Vagrant](https://www.vagrantup.com/).
* [Ansible](https://www.ansible.com/) provisioning (same playbooks for local machine and remote environments).
* [Ansible](https://www.ansible.com/) again to deploy services (Traefik and your apps).
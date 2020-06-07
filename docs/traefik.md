# Traefik Proxy with HTTPS

## Note - version 2

This guide is updated for Traefik version 2. ✨

If you are looking for the previous guides for Traefik version 1, check them in <a href="https://dockerswarm.rocks/traefik-v1/" class="external-link" target="_blank">DockerSwarm.rocks/traefik-v1/</a>.

!!! note
    There are many applications and some project generators based on the previous guides for Traefik version 1.

    If you have something already deployed, there are chances it uses those previous guides.

    But for new projects, continue here. 🚀

## Intro

So, you have a **Docker Swarm mode** cluster set up as described in <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a>.

Now you can add a main <a href="https://traefik.io/" target="_blank">**Traefik**</a> load balancer/proxy to:

* Handle **connections**.
* **Expose** specific services and applications based on their domain names.
* Handle **multiple domains** (if you need to). Similar to "virtual hosts".
* Handle **HTTPS**.
* Acquire (generate) **HTTPS certificates automatically** (including renewals) with <a href="https://letsencrypt.org/" target="_blank">Let's Encrypt</a>.
* Add HTTP **Basic Auth** for any service that you need to protect and doesn't have its own security, etc.
* Get all its configurations automatically from **Docker labels** set in your stacks (you don't need to update configuration files).

These ideas, techniques, and tools would also apply to other cluster orchestrators, like Kubernetes or Mesos, to add a main load balancer with HTTPS support, certificate generation, etc. But this article is focused on Docker Swarm mode.

### User Interface

The guide includes how to expose the internal Traefik web UI dashboard through the same Traefik load balancer, using a secure HTTPS certificate and HTTP Basic Auth.

<img src="https://dockerswarm.rocks/img/traefik-screenshot.webp">

## How it works

The idea is to have a main load balancer/proxy that covers all the Docker Swarm cluster and handles HTTPS certificates and requests for each domain.

But doing it in a way that allows you to have other Traefik services inside each stack without interfering with each other, to redirect based on path in the same stack (e.g. one container handles `/` for a web frontend and another handles `/api` for an API under the same domain), or to redirect from HTTP to HTTPS selectively.

## Preparation

* Connect via SSH to a manager node in your cluster (you might have only one node) that will have the Traefik service.
* Create a network that will be shared with Traefik and the containers that should be accessible from the outside, with:

```bash
docker network create --driver=overlay traefik-public
```

* Get the Swarm node ID of this node and store it in an environment variable:

```bash
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

* Create a tag in this node, so that Traefik is always deployed to the same node and uses the same volume:

```bash
docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
```

* Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates, e.g.:

```bash
export EMAIL=admin@example.com
```

* Create an environment variable with the domain you want to use for the Traefik UI (user interface), e.g.:

```bash
export DOMAIN=traefik.sys.example.com
```

* You will access the Traefik dashboard at this domain, e.g. `traefik.sys.example.com`. So, make sure that your DNS records point the domain to one of the IPs of the cluster. Better if it is the IP where the Traefik service runs (the manager node you are currently connected to).

* Create an environment variable with a username (you will use it for the HTTP Basic Auth for Traefik and Consul UIs), for example:

```bash
export USERNAME=admin
```

* Create an environment variable with the password, e.g.:

```bash
export PASSWORD=changethis
```

* Use `openssl` to generate the "hashed" version of the password and store it in an environment variable:

```bash
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
```

**(Optional)**: Alternatively, if you don't want to put the password in an environment variable, you could type it interactively, e.g.:

```console
$ export HASHED_PASSWORD=$(openssl passwd -apr1)
Password: $ enter your password here
Verifying - Password: $ re enter your password here
```

* You can check the contents with:

```bash
echo $HASHED_PASSWORD
```

It will look like:

```
$apr1$89eqM5Ro$CxaFELthUKV21DpI3UTQO.
```

## Create the Docker Compose file

* Download the file `traefik.yml`:

```bash
curl -L dockerswarm.rocks/traefik.yml -o traefik.yml
```

* ...or create it manually, for example, using `nano`:

```bash
nano traefik.yml
```

* And copy the contents inside:

```YAML
{!./traefik.yml!}
```

!!! tip
    Read the internal comments to learn what each configuration is for.

    The file without comments is actually quite smaller, but the comments should give you an idea of what everything is doing.

!!! info
    This is just a standard Docker Compose file.

    It's common to name the file `docker-compose.yml` or something like `docker-compose.traefik.yml`.

    Here it's named just `traefik.yml` for brevity.

## Deploy it

Deploy the stack with:

```bash
docker stack deploy -c traefik.yml traefik
```

It will use the environment variables you created above.


## Check it

* Check if the stack was deployed with:

```bash
docker stack ps traefik
```

It will output something like:

```
ID             NAME                IMAGE          NODE              DESIRED STATE   CURRENT STATE          ERROR   PORTS
w5o6fmmln8ni   traefik_traefik.1   traefik:v2.2   dog.example.com   Running         Running 1 minute ago
```

* You can check the Traefik logs with:

```bash
docker service logs traefik_traefik
```

## Check the user interface

After some seconds/minutes, Traefik will acquire the HTTPS certificates for the web user interface (UI).

You will be able to securely access the web UI at `https://traefik.<your domain>` using the created username and password.

Once you deploy a stack, you will be able to see it there and see how the different hosts and paths map to different Docker services / containers.

## Getting the client IP

If you need to read the client IP in your applications/stacks using the `X-Forwarded-For` or `X-Real-IP` headers provided by Traefik, you need to make Traefik listen directly, not through Docker Swarm mode, even while being deployed with Docker Swarm mode.

For that, you need to publish the ports using "host" mode.

So, the Docker Compose lines:

```YAML
    ports:
      - 80:80
      - 443:443
```

need to be:

```YAML
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
```

You can use all the same instructions above, downloading the host-mode file:

```bash
curl -L dockerswarm.rocks/traefik-host.yml -o traefik-host.yml
```

Or alternatively, copying it directly:

```YAML
{!./traefik-host.yml!}
```

And then deploying with:

```bash
docker stack deploy -c traefik-host.yml traefik
```

## Distributed Let's Encrypt

There was a guide in DockerSwarm.rocks for setting up Traefik with Consul to store the Let's Encrypt certificates in a distributed way.

Nevertheless, that technique was fragile and error prone. Because of that, the Traefik team disabled that functionality in Traefik version 2.

In many cases the technique described here should be enough. But if you have a big and complex system that requires a distributed Let's Encrypt store for Traefik, you should check <a href="https://containo.us/traefikee/" class="external-link" target="_blank">Traefik Enterprise Edition</a> that supports it.

## What's next

The next thing would be to deploy a stack (a complete web application, with backend, frontend, database, etc) using this Docker Swarm mode cluster.

It's actually very simple, as you can use Docker Compose for local development and then use the same files for deployment in the Docker Swarm mode cluster.

If you want to try it right now, you can check this project generator with a FastAPI backend and a frontend using Vue.js <a href="https://github.com/tiangolo/full-stack-fastapi-postgresql" class="external-link" target="_blank">https://github.com/tiangolo/full-stack-fastapi-postgresql</a>.

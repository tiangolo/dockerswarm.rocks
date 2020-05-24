<p align="center">
  <a href="https://dockerswarm.rocks"><img src="https://dockerswarm.rocks/img/logo-light-blue-vectors.svg" alt="dockerswarm.rocks"></a>
</p>

## Why?

<a href="https://www.docker.com/" target="_blank">Docker</a> is a great tool (the "de facto" standard) to build **Linux containers**.

<a href="https://docs.docker.com/compose/" target="_blank">Docker Compose</a> is great to **develop locally** with Docker, in a replicable way.

<a href="https://docs.docker.com/engine/swarm/" target="_blank">Docker Swarm Mode</a> is great to **deploy** your application stacks to **production**, in a **distributed cluster**, using the same files used by Docker Compose locally.

So, with Docker Swarm Mode you have:

* Replicability, use the same files as when developing locally.
* Simplicity and speed for development and deployment.
* Robustness and security, with fault-tolerant clusters.

## Docker Swarm mode

If you have Docker installed, you already have Docker Swarm, it's integrated into Docker.

You don't have to install anything else.

### Note

<blockquote>

<p>Whenever you read here "Docker Swarm" we are actually talking about "<strong>Docker Swarm mode</strong>".</p>

<p>Not the deprecated product called "Docker Swarm".</p>

</blockquote>

## Alternatives

Some of the main alternatives are:

* <a href="https://kubernetes.io/" target="_blank">Kubernetes</a>.
* <a href="http://mesos.apache.org/" target="_blank">Mesos</a>.

To use any of them you need to learn a huge new set of concepts, configurations, files, commands, etc.

### About Docker Swarm mode

Docker Swarm mode is comparable to them.

But it, with all the ideas described here, is what I would recommend for teams of **less than 200 developers**, or clusters of **less than 1000 machines**.

This includes **small / medium size organizations** (like when you are not Google or Amazon), **startups**, one-man projects, and "hobby" projects.

Try it.

Set up a distributed cluster ready for production.

...In about **20 minutes**.

If it doesn't work for you, then you can go for Kubernetes, Mesos or any other.

Those are great tools. But learning them might take weeks. So, the **20 minutes spent here** are not much (and up to here you already spent 3 minutes).

## Single server

With Docker Swarm mode you can start with a "cluster" of a single server.

You can set it up, deploy your applications and do everything on a $5 USD/month server.

And then, when the time to grow comes, you can add more servers to the cluster.

With a **one-line command**.

And you can create your applications to be ready for massive scale from the beginning, starting from a single small server.

## About **Docker Swarm Rocks**

This is not associated with Docker or any of the tools suggested here.

It's mainly a set of ideas, documentation and tools to use existing open source products efficiently together.

## Prerequisites

* To know some Linux.
* To know some Docker.

## Install and set up

### Install a new Linux server with Docker

* Create a new remote VPS ("virtual private server").
* Deploy the latest Ubuntu LTS ("long term support") version. At the time of this writing it's `Ubuntu 18.04`.
* Connect to it via SSH, e.g.:

```bash
ssh root@172.173.174.175
```

* Define a server name using a subdomain of a domain you own, for example `dog.example.com`.
* Make sure the subdomain DNS records point to your VPS's IP address.
* Create a temporal environment variable with the name of the host to be used later, e.g.:

```bash
export USE_HOSTNAME=dog.example.com
```

* Set up the server `hostname`:

```bash
# Set up the server hostname
echo $USE_HOSTNAME > /etc/hostname
hostname -F /etc/hostname
```

**Note**: If you are not a `root` user, you might need to add `sudo` to these commands. The shell will tell you when you don't have enough permissions. Note that `sudo` does not preserve environment variables by default, but this can be enabled via the `-E` flag.

* Update packages:

```bash
# Install the latest updates
apt-get update
apt-get upgrade -y
```

* Install Docker following <a href="https://docs.docker.com/install/" target="_black">the official guide</a>...
* ...or alternatively, run the official convenience script:

```bash
# Download Docker
curl -fsSL get.docker.com -o get-docker.sh
# Install Docker using the stable channel (instead of the default "edge")
CHANNEL=stable sh get-docker.sh
# Remove Docker install script
rm get-docker.sh
```

* If you are setting up multiple nodes (servers/VPSs), repeat these steps for each one.
    * Make sure you use a different domain/subdomain for each node.

### Set up swarm mode

In Docker Swarm Mode you have one or more "manager" nodes and one or more "worker" nodes (that can be the same manager nodes).

The first step is to configure one (or more) manager nodes.

* On the main manager node, run:

```bash
docker swarm init
```

**Note**: if you see an error like:

```
Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on interface eth0 (138.68.58.48 and 10.19.0.5) - specify one with --advertise-addr
```

...select the public IP (e.g. `138.68.58.48` in this example), and run the command again with `--advertise-addr`, e.g.:

```bash
docker swarm init --advertise-addr 138.68.58.48
```

### Add manager nodes (optional)

* On the main manager node, for each additional manager node you want to set up, run:

```bash
docker swarm join-token manager
```

* Copy the result and paste it in the additional manager node's terminal, it will be something like:

```bash
 docker swarm join --token SWMTKN-1-5tl7yaasdfd9qt9j0easdfnml4lqbosbasf14p13-f3hem9ckmkhasdf3idrzk5gz 172.173.174.175:2377
```

### Add worker nodes (optional)

* On the main manager node, for each additional worker node you want to set up, run:

```bash
docker swarm join-token worker
```

* Copy the result and paste it in the additional worker node's terminal, it will be something like:

```bash
docker swarm join --token SWMTKN-1-5tl7ya98erd9qtasdfml4lqbosbhfqv3asdf4p13-dzw6ugasdfk0arn0 172.173.174.175:2377
```

### Check it

* Check that the cluster has all the nodes connected and set up:

```bash
docker node ls
```

It outputs something like:

```
ID                            HOSTNAME             STATUS    AVAILABILITY    MANAGER STATUS    ENGINE VERSION
ndcg2iavasdfrm6q2qwere2rr *   dog.example.com      Ready     Active          Leader            18.06.1-ce
3jrutmd3asdf1ombqwerr9svk     cat.example.com      Ready     Active          Reachable         18.06.1-ce
i9ec9hjasdfaoyyjqwerr3iqa     snake.example.com    Ready     Active          Reachable         18.06.1-ce
```

## Done

That's it.

You have a distributed Docker swarm mode cluster set up.

Check other sections in the documentation at <a href="https://dockerswarm.rocks">https://dockerswarm.rocks</a> to see how to set up HTTPS, you still have time, the 20 minutes are not over yet.

Then you can see how to deploy stacks, etc.

You already did the hard part, the rest is easy.

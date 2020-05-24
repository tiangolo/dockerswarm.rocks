# Portainer web user interface for your Docker Swarm cluster

<a href="https://github.com/portainer/portainer" target="_blank">Portainer</a> is a web UI (user interface) that allows you to see the state of your Docker services in a Docker Swarm mode cluster and manage it.

Follow this guide to integrate it in your Docker Swarm mode cluster deployed as described in <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a> with a global Traefik HTTPS proxy.

Here's one of the screens:

<img src="https://dockerswarm.rocks/img/portainer.png">

## Preparation

* Connect via SSH to a Docker Swarm manager node.

* Create an environment variable with the domain where you want to access your Portainer instance, e.g.:

```bash
export DOMAIN=portainer.sys.example.com
```

* Make sure that your DNS records point that domain (e.g. `portainer.sys.example.com`) to one of the IPs of the Docker Swarm mode cluster.

* Get the Swarm node ID of this (manager) node and store it in an environment variable:

```bash
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

* Create a tag in this node, so that Portainer is always deployed to the same node and uses the existing volume:

```bash
docker node update --label-add portainer.portainer-data=true $NODE_ID
```

## Create the Docker Compose file

* Download the file `portainer.yml`:

```bash
curl -L dockerswarm.rocks/portainer.yml -o portainer.yml
```

* ...or create it manually, for example, using `nano`:

```bash
nano portainer.yml
```

* And copy the contents inside:

```YAML
{!./portainer.yml!}
```

!!! info
    This is just a standard Docker Compose file.

    It's common to name the file `docker-compose.yml` or something like `docker-compose.portainer.yml`.

    Here it's named just `portainer.yml` for brevity.

## Deploy it

Deploy the stack with:

```bash
docker stack deploy -c portainer.yml portainer
```

It will use the environment variables you created above.

## Check it

* Check if the stack was deployed with:

```bash
docker stack ps portainer
```

It will output something like:

```
ID             NAME                       IMAGE                        NODE              DESIRED STATE   CURRENT STATE          ERROR   PORT
xvyasdfh56hg   portainer_agent.b282rzs5   portainer/agent:latest       dog.example.com   Running         Running 1 minute ago
j3ahasdfe0mr   portainer_portainer.1      portainer/portainer:latest   cat.example.com   Running         Running 1 minute ago
```

* You can check the Portainer logs with:

```bash
docker service logs portainer_portainer
```

## Check the user interface

After some seconds/minutes, Traefik will acquire the HTTPS certificates for the web user interface.

You will be able to securely access the web UI at `https://<your portainer domain>` where you can create your username and password.

### Timing Note

Make sure you login and create your credentials soon after Portainer is ready, or it will automatically shut down itself for security.

If you didn't create the credentials on time and it shut down itself automatically, you can force it to restart with:

```bash
docker service update portainer_portainer --force
```

## References

This guide on Portainer is adapted from the <a href="http://portainer.readthedocs.io/en/stable/agent.html" target="_blank">official Portainer documentation for Docker Swarm mode clusters</a>, adding deployment restrictions to make sure the same volume and database is always used and to enable HTTPS via Traefik, using the same ideas from <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a>.

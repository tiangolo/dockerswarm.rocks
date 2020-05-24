# Swarmpit web user interface for your Docker Swarm cluster

<a href="https://swarmpit.io/" target="_blank">Swarmpit</a> provides a nice and clean way to manage your Docker Swarm cluster.

Follow this guide to integrate it in your Docker Swarm mode cluster deployed as described in <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a> with a global Traefik HTTPS proxy.

Here's one of the screens:

![Swarmpit UI](./img/swarmpit.png)

## Preparation

* Connect via SSH to a Docker Swarm manager node.

* Create an environment variable with the domain where you want to access your Swarmpit instance, e.g.:

```bash
export DOMAIN=swarmpit.sys.example.com
```

* Make sure that your DNS records point that domain (e.g. `swarmpit.sys.example.com`) to one of the IPs of the Docker Swarm mode cluster.

* Get the Swarm node ID of this (manager) node and store it in an environment variable:

```bash
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

* Create a label in this node, so that the CouchDB database used by Swarmpit is always deployed to the same node and uses the existing volume:

```bash
docker node update --label-add swarmpit.db-data=true $NODE_ID
```

* Create another label in this node, so that the Influx database used by Swarmpit is always deployed to the same node and uses the existing volume:

```bash
docker node update --label-add swarmpit.influx-data=true $NODE_ID
```

## Create the Docker Compose file

* Download the file `swarmpit.yml`:

```bash
curl -L dockerswarm.rocks/swarmpit.yml -o swarmpit.yml
```

* ...or create it manually, for example, using `nano`:

```bash
nano swarmpit.yml
```

* And copy the contents inside:

```YAML
{!./swarmpit.yml!}
```

!!! info
    This is just a standard Docker Compose file.

    It's common to name the file `docker-compose.yml` or something like `docker-compose.swarmpit.yml`.

    Here it's named just `swarmpit.yml` for brevity.

## Deploy it

Deploy the stack with:

```bash
docker stack deploy -c swarmpit.yml swarmpit
```

It will use the environment variables you created above.

## Check it

* Check if the stack was deployed with:

```bash
docker stack ps swarmpit
```

It will output something like:

```
ID             NAME                       IMAGE                      NODE                DESIRED STATE   CURRENT STATE          ERROR   PORT
kkhasdfvce30   swarmpit_agent.ndasdfav5   swarmpit/agent:latest      dog.example.com     Running         Running 3 minutes ago
k8oasdfg70jm   swarmpit_agent.i9asdfjps   swarmpit/agent:latest      cat.example.com     Running         Running 3 minutes ago
kcvasdft0yzj   swarmpit_agent.3jasdfd3k   swarmpit/agent:latest      snake.example.com   Running         Running 3 minutes ago
9onasdfzopve   swarmpit_agent.r6asdfb20   swarmpit/agent:latest      snake.example.com   Running         Running 3 minutes ago
fxoasdfwjrbj   swarmpit_db.1              couchdb:2.3.0              dog.example.com     Running         Running 3 minutes ago
m4jasdf3369c   swarmpit_app.1             swarmpit/swarmpit:latest   cat.example.com     Running         Running 3 minutes ago
```

* You can check the Swarmpit logs with:

```bash
docker service logs swarmpit_app
```

## Check the user interfaces

After some seconds/minutes, Traefik will acquire the HTTPS certificates for the web user interface.

You will be able to securely access the web UI at `https://<your swarmpit domain>` where you can create your username and password.

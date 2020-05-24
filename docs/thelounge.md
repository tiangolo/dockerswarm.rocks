# The Lounge - self-hosted web IRC client

<a href="https://thelounge.chat/" target="_blank">The Lounge</a> is a self-hosted web IRC client.

Follow this guide to integrate it in your Docker Swarm mode cluster deployed as described in <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a> with a global Traefik HTTPS proxy.

## Preparation

* Connect via SSH to a Docker Swarm manager node.

* Create an environment variable with the domain where you want to access your instance, e.g.:

```bash
export DOMAIN=thelounge.example.com
```

* Make sure that your DNS records point that domain (e.g. `thelounge.example.com`) to one of the IPs of the Docker Swarm mode cluster.

* Get the Swarm node ID of this node (or any other node) and store it in an environment variable:

```bash
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

* Create a tag in this node, so that the data used by The Lounge is always deployed to the same node and uses the existing volume:

```bash
docker node update --label-add thelounge.thelounge-data=true $NODE_ID
```

## Create the Docker Compose file

* Download the file `thelounge.yml`:

```bash
curl -L dockerswarm.rocks/thelounge.yml -o thelounge.yml
```

* ...or create it manually, for example, using `nano`:

```bash
nano thelounge.yml
```

* And copy the contents inside:

```YAML
{!./thelounge.yml!}
```

!!! info
    This is just a standard Docker Compose file.

    It's common to name the file `docker-compose.yml` or something like `docker-compose.thelounge.yml`.

    Here it's named just `thelounge.yml` for brevity.

## Deploy it

Deploy the stack with:

```bash
docker stack deploy -c thelounge.yml thelounge
```

It will use the environment variables you created above.

## Check it

* Check if the stack was deployed with:

```bash
docker stack ps thelounge
```

It will output something like:

```
ID             NAME               IMAGE                         NODE               DESIRED STATE    CURRENT STATE          ERROR   PORT
kxdsp9f1nuil   thelounge_app.1    thelounge/thelounge:latest    dog.example.com    Running          Running 7 seconds ago
```

* You can check the logs with:

```bash
docker service logs thelounge_app
```

...you might see a message like:

```
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:42 [INFO] Configuration file created at /var/opt/thelounge/config.js.
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:43 [INFO] The Lounge v3.0.1 (Node.js 10.15.1 on linux x64)
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:43 [INFO] Configuration file: /var/opt/thelounge/config.js
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:43 [INFO] Available at http://:::9000/ in private mode
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:43 [INFO] New VAPID key pair has been generated for use with push subscription.
thelounge_app.1.kxdsp9f1nuil@dog.example.com    | 2019-05-12 09:58:43 [INFO] There are currently no users. Create one with thelounge add <name>.
```

## Create a user

* Now, to create a user, enter into a bash session inside the running container, type:

```bash
docker exec -it thelounge_app
```

and hit `Tab` to get autocompletion, then type `bash`. It would look something like:

```bash
docker exec -it thelounge_app.1.kxdsp9f1nuilj0vclhhfuc4ho bash
```

Then hit `Enter`, you will be in a bash session inside the running container.

You will see a prompt like:

```
root@61d7cc2c6e44:/#
```

* Now create the user, if the user is named `admin`, it would be something like:

```bash
thelounge add admin
```

* You will see a prompt for a password. Enter your password and hit `Enter`.
* It will ask you if you want to save, hit `Enter` again and the user will be created.

## Login and configure

Now you can open the URL in your browser, e.g.: `https://thelounge.example.com`.

Use the user and password you just created and finish configuring it in the web browser.

# Traefik Proxy with HTTPS using volume

This article lives in:

* <a href="https://medium.com/@tiangolo/docker-swarm-mode-and-traefik-for-a-https-cluster-20328dba6232" target="_blank">Medium</a>
* <a href="https://github.com/tiangolo/medium-posts/tree/master/docker-swarm-mode-and-traefik-for-a-https-cluster" target="_blank">GitHub</a>
* <a href="https://dockerswarm.rocks/traefik-with-volume/" target="_blank">DockerSwarm.rocks</a>

## Note about Traefik v2

This article is for Traefik version 1.

There is now a guide for Traefik version 2, if you are starting a new project, you should check that one at <a href="https://dockerswarm.rocks/traefik/" class="external-link" target="_blank">DockerSwarm.rocks/traefik/</a>.

## Set up

Set up a main load balancer with Traefik that handles the public connections and Let's encrypt HTTPS certificates.

* Connect via SSH to a manager node in your cluster (you might have only one node) that will have the Traefik service.
* Create a network that will be shared with Traefik and the containers that should be accessible from the outside, with:

```bash
docker network create --driver=overlay traefik-public
```

* Create a volume in where Traefik will store HTTPS certificates:

```bash
docker volume create traefik-public-certificates
```

**Note**: you can <a href="https://dockerswarm.rocks/traefik/" target="_blank">store certificates in Consul and deploy Traefik in each node as a fully distributed load balancer</a>, this guide covers deploying to a single node, with a mounted volume instead of a distributed Consul.

* Get the Swarm node ID of this node and store it in an environment variable:

```bash
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

* Create a tag in this node, so that Traefik is always deployed to the same node and uses the existing volume:

```bash
docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
```

* Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates:

```bash
export EMAIL=admin@example.com
```

* Create an environment variable with the name of the host, e.g.:

```bash
export USE_HOSTNAME=dog.example.com
# or if you have your $HOSTNAME variable configured:
export USE_HOSTNAME=$HOSTNAME
```

* You will access the Traefik dashboard at `traefik.<your hostname>`, e.g. `traefik.dog.example.com`. So, make sure that your DNS records point `traefik.<your hostname>` to one of the IPs of the cluster. Better if it is the IP where the Traefik service runs (the manager node you are currently connected to).

* Create an environment variable with a username (you will use it for the HTTP Basic Auth), for example:

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

* You can check the contents with:

```bash
echo $HASHED_PASSWORD
```

It will look like:

```
$apr1$89eqM5Ro$CxaFELthUKV21DpI3UTQO.
```

* Create a Traefik service, copy this long command in the terminal:

```bash
docker service create \
    --name traefik \
    --constraint=node.labels.traefik-public.traefik-public-certificates==true \
    --publish 80:80 \
    --publish 443:443 \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
    --mount type=volume,source=traefik-public-certificates,target=/certificates \
    --network traefik-public \
    --label "traefik.frontend.rule=Host:traefik.${USE_HOSTNAME?Variable not set}" \
    --label "traefik.enable=true" \
    --label "traefik.port=8080" \
    --label "traefik.tags=traefik-public" \
    --label "traefik.docker.network=traefik-public" \
    --label "traefik.frontend.entryPoints=http,https" \
    --label "traefik.frontend.redirect.entryPoint=https" \
    --label "traefik.frontend.auth.basic.users=${USERNAME?Variable not set}:${HASHED_PASSWORD?Variable not set}" \
    traefik:v1.7 \
    --docker \
    --docker.swarmmode \
    --docker.watch \
    --docker.exposedbydefault=false \
    --constraints=tag==traefik-public \
    --entrypoints='Name:http Address::80' \
    --entrypoints='Name:https Address::443 TLS' \
    --acme \
    --acme.email=${EMAIL?Variable not set} \
    --acme.storage=/certificates/acme.json \
    --acme.entryPoint=https \
    --acme.httpChallenge.entryPoint=http\
    --acme.onhostrule=true \
    --acme.acmelogging=true \
    --logLevel=INFO \
    --accessLog \
    --api
```

You will be able to securely access the web UI at `https://traefik.<your domain>` using the created username and password.

The previous command explained:

* `docker service create`: create a Docker Swarm mode service
* `--name traefik`: name the service "traefik"
* `--constraint=node.labels.traefik-public.traefik-public-certificates==true` make it run on a specific node, to be able to use the certificates stored in a volume in that node
* `--publish 80:80`: listen on ports 80 - HTTP
* `--publish 443:443`: listen on port 443 - HTTPS
* `--mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock`: communicate with Docker, to read labels, etc.
* `--mount type=volume,source=traefik-public-certificates,target=/certificates`: create a volume to store TLS certificates
* `--network traefik-public`: listen to the specific network traefik-public
* `--label "traefik.frontend.rule=Host:traefik.${USE_HOSTNAME?Variable not set}"`: enable the Traefik API and dashboard in the host `traefik.$USE_HOSTNAME`, using the `$USE_HOSTNAME` environment variable created above. If the variable is not set, show an error.
* `--label "traefik.enable=true"`: make Traefik expose "itself" as a Docker service, this is what makes the Traefik dashboard available with HTTPS and basic auth
* `--label "traefik.port=8080"`: when Traefik exposes itself as a service (for the dashboard), use the internal service port `8080`
* `--label "traefik.tags=traefik-public"`: as the main Traefik proxy will only expose services with the `traefik-public` tag (using a parameter below), make the dashboard service have this tag too, so that the Traefik public (itself) can find it and expose it
* `--label "traefik.docker.network=traefik-public"`: make the dashboard service use the `traefik-public` network to expose itself
* `--label "traefik.frontend.entryPoints=http"`: make the web dashboard listen and serve on HTTP and HTTPS
* `--label "traefik.frontend.redirect.entryPoint=https"`: make Traefik redirect HTTP trafic to HTTPS for the web dashboard
* `--label "traefik.frontend.auth.basic.users=${USERNAME?Variable not set}:${HASHED_PASSWORD?Variable not set}"`: enable basic auth, so that not every one can access your Traefik web dashboard, it uses the username and password created above. If the variables are not set, show an error.
* `traefik:v1.7`: use the image `traefik:v1.7`
* `--docker`: enable Docker
* `--docker.swarmmode`: enable Docker Swarm Mode
* `--docker.watch`: enable "watch", so it reloads its config based on new stacks and labels
* `--docker.exposedbydefault=false`: don't expose all the services, only services with traefik.enable=true
* `--constraints=tag==traefik-public`: only show services with traefik.tag=traefik-public, to isolate from possible intra-stack traefik instances
* `--entrypoints='Name:http Address::80'`: create an entrypoint http, on port 80
* `--entrypoints='Name:https Address::443 TLS'`: create an entrypoint https, on port 443 with TLS enabled
* `--acme`: enable Let's encrypt
* `--acme.email=${EMAIL?Variable not set}`: let's encrypt email, using the environment variable. If not set, show an error.
* `--acme.storage=/certificates/acme.json`: where to store the Let's encrypt TLS certificates - in the mapped volume
* `--acme.entryPoint=https`: the entrypoint for Let's encrypt - created above
* `--acme.httpChallenge.entryPoint=http`: use HTTP for the ACME (Let's Encrypt HTTPS certificates) challenge, as HTTPS was disabled after a security issue
* `--acme.onhostrule=true`: get new certificates automatically with host rules: "traefik.frontend.rule=Host:web.example.com"
* `--acme.acmelogging=true`: log Let's encrypt activity - to debug when and if it gets certificates
* `--logLevel=INFO`: default logging, if the web UI is not enough to debug configurations and hosts detected, or you want to see more of the logs, set it to `DEBUG`. Have in mind that after some time it might affect performance.
* `--accessLog`: enable the access log, to see and debug HTTP traffic
* `--api`: enable the API, which includes the dashboard

## Check it

To check if it worked, check the logs:

```bash
docker service logs traefik
# To make it scrollable with `less`, run:
# docker service logs traefik | less
```

And open `https://traefik.<your domain>` in your browser, you will be asked for the username and password that you set up before, and you will be able to see the Traefik web UI interface. Once you deploy a stack, you will be able to see it there and see how the different hosts and paths map to different Docker services / containers.

## Getting the client IP

If you need to read the client IP in your applications/stacks using the `X-Forwarded-For` or `X-Real-IP` headers provided by Traefik, you need to make Traefik listen directly, not through Docker Swarm mode, even while being deployed with Docker Swarm mode.

For that, you need to publish the ports using "host" mode.

So, the two lines above:

```
    --publish 80:80 \
    --publish 443:443 \
```

need to be:

```
    --publish mode=host,target=80,published=80 \
    --publish mode=host,target=443,published=443 \
```

Here's the complete command with those lines updated:

```bash
docker service create \
    --name traefik \
    --constraint=node.labels.traefik-public.traefik-public-certificates==true \
    --publish mode=host,target=80,published=80 \
    --publish mode=host,target=443,published=443 \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
    --mount type=volume,source=traefik-public-certificates,target=/certificates \
    --network traefik-public \
    --label "traefik.frontend.rule=Host:traefik.${USE_HOSTNAME?Variable not set}" \
    --label "traefik.enable=true" \
    --label "traefik.port=8080" \
    --label "traefik.tags=traefik-public" \
    --label "traefik.docker.network=traefik-public" \
    --label "traefik.frontend.entryPoints=http,https" \
    --label "traefik.frontend.redirect.entryPoint=https" \
    --label "traefik.frontend.auth.basic.users=${USERNAME?Variable not set}:${HASHED_PASSWORD?Variable not set}" \
    traefik:v1.7 \
    --docker \
    --docker.swarmmode \
    --docker.watch \
    --docker.exposedbydefault=false \
    --constraints=tag==traefik-public \
    --entrypoints='Name:http Address::80' \
    --entrypoints='Name:https Address::443 TLS' \
    --acme \
    --acme.email=${EMAIL?Variable not set} \
    --acme.storage=/certificates/acme.json \
    --acme.entryPoint=https \
    --acme.httpChallenge.entryPoint=http\
    --acme.onhostrule=true \
    --acme.acmelogging=true \
    --logLevel=INFO \
    --accessLog \
    --api
```

## What's next

The next thing would be to deploy a stack (a complete web application, with backend, frontend, database, etc) using this Docker Swarm mode cluster.

It's actually very simple, as you can use Docker Compose for local development and then use the same files for deployment in the Docker Swarm mode cluster.

If you want to try it right now, you can check this very simple <a href="https://github.com/tiangolo/flask-frontend-docker" target="_blank">project generator with a minimal Flask backend and Vue.js frontend</a>.

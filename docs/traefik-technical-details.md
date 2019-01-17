### Consul

Consul by default expects to be running independent of any cluster orchestrator. To have fixed IPs, etc.

This configuration includes everything necessary to make it work in Docker Swarm, in a distributed and resilient manner.

Consul expects to have a single leader at each time, and to have all the Consul instances communicating to each other, knowing each other's specific IP address/host.

So, by default, you would have to make sure to set up a specific Consul instance per node in your cluster.

And if your cluster has more nodes, or if one node goes down, you would have to manually create another Consul instance, etc.

### Consul leader

In this Docker Compose stack we define a single Consul leader named `consul-leader`, that is able to self-elect as a leader (`-boostrap`), and tries to join another consul replica by the name `consul-replica`.

In reality, `consul-replica` is a service with multiple replicas, but each one of them will contact `consul-leader`, so, they will all end up exchanging their IP addresses and being able to synchronize.

It's configured to listen to the first internal private IP address by using the environment variable `CONSUL_BIND_INTERFACE` listening on the first "ethernet" (virtual) interface `eth0`.

It is attached to the `default` network to be able to talk to the other `consul-replica` service (of multiple replica containers) and to the external network `traefik-public`, to be able to expose its web user interface with Traefik.

It has several deployment labels, these are what make Traefik expose the Consul web UI with specific settings:

* `traefik.frontend.rule=Host:consul.${DOMAIN}`: use as a host, the subdomain `consul` of the domain set in the environment variable `DOMAIN`. This host name is what will be used to genereate/acquire the HTTPS certificates.
* `traefik.enable=true`: tell Traefik to expose this service in the web (otherwise, it wouldn't).
* `traefik.port=8500`: expose the content from the port `8500` (that's the port inside the container).
* `traefik.tags=${TRAEFIK_PUBLIC_TAG:-traefik-public}`: as the main Traefik proxy will only expose services with the `traefik-public` tag (using a parameter below), make the Consul service have this tag too, so that the Traefik public can find it and expose it. Use as the tag the environment variable `TRAEFIK_PUBLIC_TAG`, or by default, set it to `traefik-public`.
* `traefik.docker.network=traefik-public`: tell Traefik to get the contents provided by this service using that shared network.
* `traefik.redirectorservice.frontend.entryPoints=http`: make the service listen to HTTP, so that it can redirect to HTTPS.
* `traefik.redirectorservice.frontend.redirect.entryPoint=https`: make Traefik redirect HTTP trafic to HTTPS for the web UI.
* `traefik.webservice.frontend.entryPoints=https`: make the web UI listen and serve on HTTPS.
* `traefik.frontend.auth.basic.users=${USERNAME}:${HASHED_PASSWORD}`: enable basic auth, so that not everyone can access your Traefik web dashboard, it uses the username and password created above.

```YAML hl_lines="4 6 10 12 13 16 17 18 19 20 21 22 23 24 25 26"
{!traefik.yml!}
```

### Consul replicas

`consul-replica` is a service with multiple replicas (multiple containers).

It is set to distribute those replicas spread across the cluster with `spread: node.id`.

It uses the environment variable `CONSUL_REPLICAS`, set by default to 3, to set the number of replicas.

So, if you have a huge cluster and you want to have more Consul services, you can just update that environment variable, to a bigger number, right before deploying the stack.

As each replica will send its data (IP, ID, etc) to the service `consul-leader`, that service will have all the IPs and IDs of the replicas, and then it will send their data to the rest.

That way, all the Consul instances will be able to communicate and synchronize.

```YAML hl_lines="27 29 38 41"
{!traefik.yml!}
```

### Traefik

The `traefik` service exposes the ports `80` (standard for HTTP) and `443` (standard for HTTPS).

It creates replicas set to the environment variable `TRAEFIK_REPLICAS`, or by default, `3`.

It is marked to be started on Docker Swarm manager nodes, as it needs to be able to communicate to Docker directly, to be able to read the labels you create in the rest of the stacks.

It has labels that configure how its own UI interface should be exposed (by himself), all the labels are very similar to the ones described above for Consul. 

Here are some specific details:

* `traefik.frontend.rule=Host:traefik.${DOMAIN}`: use as a host, the subdomain `traefik` of the domain set in the environment variable `DOMAIN`. This host name is what will be used to genereate/acquire the HTTPS certificates.
* `traefik.port=8080`: the content of the Traefik web UI is served in the container port `8080`, this tells Traefik to get the content from this port when serving pages to the public (using the standard HTTPS port, `443`).


The Traefik service is configured to communicate with Docker directly, using a mounted volume for `/var/run/docker.sock`. This is needed for it to be able to read the labels in the stacks you create later.

The command has several flags:

* `--docker`: enable Docker.
* `--docker.swarmmode`: enable Docker Swarm Mode.
* `--docker.watch`: enable "watch", so it reloads its config based on new stacks and labels.
* `--docker.exposedbydefault=false`: don't expose all the services, only services with traefik.enable=true.
* `--constraints=tag==traefik-public`: only show services with traefik.tag=traefik-public, to isolate from possible intra-stack traefik instances.
* `--entrypoints='Name:http Address::80'`: create an entrypoint http, on port 80.
* `--entrypoints='Name:https Address::443 TLS'`: create an entrypoint https, on port 443 with TLS enabled.
* `--consul`: enable Consul to store configurations.
* `--consul.endpoint="consul-leader:8500"`: use the `consul-leader` host (the service in the same stack) with its domain to communicate with Consul.
* `--acme`: enable Let's encrypt.
* `--acme.email=${EMAIL}`: let's encrypt email, using the environment variable.
* `--acme.storage="traefik/acme/account"`: store the HTTPS certificates in this location in Consul.
* `--acme.entryPoint=https`: the entrypoint for Let's encrypt - created above.
* `--acme.httpChallenge.entryPoint=http`: use HTTP for the ACME (Let's Encrypt HTTPS certificates) challenge, as HTTPS was disabled after a security issue.
* `--acme.onhostrule=true`: get new certificates automatically with host rules: "traefik.frontend.rule=Host:web.example.com".
* `--acme.acmelogging=true`: log Let's encrypt activity - to debug when and if it gets certificates.
* `--logLevel=INFO`: default logging, if the web UI is not enough to debug configurations and hosts detected, or you want to see more of the logs, set it to `DEBUG`. Have in mind that after some time it might affect performance.
* `--accessLog`: enable the access log, to see and debug HTTP traffic.
* `--api`: enable the API, which includes the dashboard.

It is connected to the internal stack `default`, to store the configurations in Consul, and to `traefik-public`, to be able to communicate with other services in other stacks that are also attached to that network. That's how it can later serve the content from those services.

Technically, Traefik and Consul could just use the network `traefik-public` to communicate, to store configurations, HTTPS certificates, etc. But it's more explicit what each network does having the intra-stack `default` network declared too.

```YAML hl_lines="45 46 48 51 55 57 67 69 70 77 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 89 90 99 100"
{!traefik.yml!}
```

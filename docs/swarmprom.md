# Swarmprom for real-time monitoring and alerts

This article lives in:

* <a href="https://medium.com/@tiangolo/docker-swarm-with-swarmprom-for-real-time-monitoring-and-alerts-282da7890698" target="_blank">Medium</a>
* <a href="https://github.com/tiangolo/medium-posts/tree/master/docker-swarm-with-swarmprom-for-real-time-monitoring-and-alerts" target="_blank">GitHub</a>
* <a href="https://dockerswarm.rocks/swarmprom/" target="_blank">DockerSwarm.rocks</a>

## Intro

Let's say you already set up a **<a href="https://dockerswarm.rocks" target="_blank">Docker Swarm mode</a>** cluster, with a <a href="https://dockerswarm.rocks/traefik/" target="_blank">Traefik HTTPS proxy</a>.

Here's how you can set up <a href="https://github.com/stefanprodan/swarmprom" target="_blank">Swarmprom</a> to monitor your cluster.

It will allow you to:

* Monitor **CPU**, **disk**, **memory usage**, etc.
* Monitor it all per **node**, per **service**, per **container**, etc.
* Have a nice, interactive, **real-time dashboard** with all the data nicely plotted.
* Trigger **alerts** (for example, in Slack, Rocket.chat, etc) when your services/nodes pass certain thresholds.
* And more...

Swarmprom is actually just a set of tools pre-configured in a smart way for a Docker Swarm cluster.

It includes:

* <a href="https://prometheus.io/" target="_blank">Prometheus</a>
* <a href="https://grafana.com/" target="_blank">Grafana</a>
* <a href="https://github.com/google/cadvisor" target="_blank">cAdvisor</a>
* <a href="https://github.com/prometheus/node_exporter" target="_blank">Node Exporter</a>
* <a href="https://github.com/prometheus/alertmanager" target="_blank">Alert Manager</a>
* <a href="https://github.com/cloudflare/unsee" target="_blank">Unsee</a>

Here's how it looks like:

<img src="https://dockerswarm.rocks/img/swarmprom.png">

## Instructions

* Clone Swarmprom repository and enter into the directory:

```bash
$ git clone https://github.com/stefanprodan/swarmprom.git
$ cd swarmprom
```

* Set and export an `ADMIN_USER` environment variable:

```bash
export ADMIN_USER=admin
```

* Set and export an `ADMIN_PASSWORD` environment variable:

```bash
export ADMIN_PASSWORD=changethis
```

* Set and export a hashed version of the `ADMIN_PASSWORD` using `openssl`, it will be used by Traefik's HTTP Basic Auth for most of the services:

```bash
export HASHED_PASSWORD=$(openssl passwd -apr1 $ADMIN_PASSWORD)
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

it will look like:

```
$apr1$89eqM5Ro$CxaFELthUKV21DpI3UTQO.
```

* Create and export an environment variable `DOMAIN`, e.g.:

```bash
export DOMAIN=example.com
```

and make sure that the following sub-domains point to your Docker Swarm cluster IPs:

* `grafana.example.com`
* `alertmanager.example.com`
* `unsee.example.com`
* `prometheus.example.com`

(and replace `example.com` with your actual domain).

**Note**: You can also use a subdomain, like `swarmprom.example.com`. Just make sure that the subdomains point to (at least one of) your cluster IPs. Or set up a wildcard subdomain (`*`).

* If you are using Slack and want to integrate it, set the following environment variables:

```bash
export SLACK_URL=https://hooks.slack.com/services/TOKEN
export SLACK_CHANNEL=devops-alerts
export SLACK_USER=alertmanager
```

**Note**: by using `export` when declaring all the environment variables above, the next command will be able to use them.

## Create the Docker Compose file

* Download the file `swarmprom.yml`:

```bash
curl -L dockerswarm.rocks/swarmprom.yml -o swarmprom.yml
```

* ...or create it manually, for example, using `nano`:

```bash
nano swarmprom.yml
```

* And copy the contents inside:

```YAML
{!./swarmprom.yml!}
```

!!! info
    This is just a standard Docker Compose file.

    It's common to name the file `docker-compose.yml` or something like `docker-compose.swarmprom.yml`.

    Here it's named just `swarmprom.yml` for brevity.

* Deploy the Traefik version of the stack:

```bash
docker stack deploy -c swarmprom.yml swarmprom
```

To test it, go to each URL:

* `https://grafana.example.com`
* `https://alertmanager.example.com`
* `https://unsee.example.com`
* `https://prometheus.example.com`

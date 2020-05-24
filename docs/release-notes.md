# Release Notes

## Latest Changes

* Update Portainer guide to use the same conventions for `DOMAIN` as the rest of the guides. PR [#25](https://github.com/tiangolo/dockerswarm.rocks/pull/25) by [@Mantosh](https://github.com/Mantosh).
* Add CI with GitHub actions to deploy to Netlify. PR [#34](https://github.com/tiangolo/dockerswarm.rocks/pull/34).
* Move development dependencies from Pipenv to Poetry. PR [#33](https://github.com/tiangolo/dockerswarm.rocks/pull/33).
* Update and simplify labels for redirections. PR [#22](https://github.com/tiangolo/dockerswarm.rocks/pull/22) by [@ldez](https://github.com/ldez).
* Add [Issue Manager](https://github.com/tiangolo/issue-manager).
* Update Docker Compose files to show an error if the environment variables are not set. PR [#15](https://github.com/tiangolo/dockerswarm.rocks/pull/15) by [@dmrty](https://github.com/dmrty).
* Update development dependencies for security.
* Add warning about `sudo` and environment variables. PR [#13](https://github.com/tiangolo/dockerswarm.rocks/pull/13) by [@dmontagu](https://github.com/dmontagu).

## 0.2.0

* Add notes about starting Traefik in host mode. To be able to get the real IP from clients. Here's the updated <a href="https://dockerswarm.rocks/traefik/#getting-the-client-ip" target="_blank">documentation for distributed Traefik</a> and here's the updated <a href="https://dockerswarm.rocks/traefik-with-volume/#getting-the-client-ip" target="_blank">documentation for a single Traefik using a volume</a>.

## 0.1.0

* Add <a href="https://swarmpit.io/" target="_blank">Swarmpit</a> with deployment stack compatible with <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a>.

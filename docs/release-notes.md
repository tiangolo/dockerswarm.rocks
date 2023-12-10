# Release Notes

## Latest Changes

* 🚨 Deprecate website. PR [#104](https://github.com/tiangolo/dockerswarm.rocks/pull/104) by [@tiangolo](https://github.com/tiangolo).
* ✏ Fix typo in comment for traefik-host.yml. PR [#52](https://github.com/tiangolo/dockerswarm.rocks/pull/52) by [@iRhonin](https://github.com/iRhonin).
* 👷 Add latest-changes GitHub action. PR [#65](https://github.com/tiangolo/dockerswarm.rocks/pull/65) by [@tiangolo](https://github.com/tiangolo).
* Remove unwanted Swarmpit port 888. PR [#51](https://github.com/tiangolo/dockerswarm.rocks/pull/51) by [@inmishrar](https://github.com/inmishrar).
* Fix typo in YAML comment. PR [#46](https://github.com/tiangolo/dockerswarm.rocks/pull/46) by [@bmaggard](https://github.com/bmaggard).
* Reword comment about Traefik image to use. PR [#45](https://github.com/tiangolo/dockerswarm.rocks/pull/45) by [@bmaggard](https://github.com/bmaggard).
* Fix typo in comment on Traefik YAML file. PR [#44](https://github.com/tiangolo/dockerswarm.rocks/pull/44) by [@bmaggard](https://github.com/bmaggard).
* Add alternative way to type passwords interactively. PR [#42](https://github.com/tiangolo/dockerswarm.rocks/pull/42).
* Add T.A.D.S. project generator. PR [#36](https://github.com/tiangolo/dockerswarm.rocks/pull/36) by [@Thomvaill](https://github.com/Thomvaill).

### Internal

* 📝 Update admonition syntax. PR [#111](https://github.com/tiangolo/dockerswarm.rocks/pull/111) by [@tiangolo](https://github.com/tiangolo).
* 👷 Add GitHub Action to comment in PRs with previews. PR [#110](https://github.com/tiangolo/dockerswarm.rocks/pull/110) by [@tiangolo](https://github.com/tiangolo).
* 👷 Deploy docs to Cloudflare. PR [#109](https://github.com/tiangolo/dockerswarm.rocks/pull/109) by [@tiangolo](https://github.com/tiangolo).
* 👷 Update Dependabot. PR [#105](https://github.com/tiangolo/dockerswarm.rocks/pull/105) by [@tiangolo](https://github.com/tiangolo).
* ⬆ Bump actions/setup-python from 1 to 4. PR [#97](https://github.com/tiangolo/dockerswarm.rocks/pull/97) by [@dependabot[bot]](https://github.com/apps/dependabot).
* ⬆ Bump tiangolo/issue-manager from 0.2.0 to 0.4.0. PR [#98](https://github.com/tiangolo/dockerswarm.rocks/pull/98) by [@dependabot[bot]](https://github.com/apps/dependabot).
* ⬆ Bump nwtgck/actions-netlify from 1.0.3 to 2.1.0. PR [#99](https://github.com/tiangolo/dockerswarm.rocks/pull/99) by [@dependabot[bot]](https://github.com/apps/dependabot).
* ⬆ Bump actions/checkout from 2 to 4. PR [#100](https://github.com/tiangolo/dockerswarm.rocks/pull/100) by [@dependabot[bot]](https://github.com/apps/dependabot).
* 👷 Add dependabot. PR [#96](https://github.com/tiangolo/dockerswarm.rocks/pull/96) by [@tiangolo](https://github.com/tiangolo).
* 👷 Update latest-changes GitHub Action. PR [#94](https://github.com/tiangolo/dockerswarm.rocks/pull/94) by [@tiangolo](https://github.com/tiangolo).
* 🐛 Fix config for latest-changes GitHub Action. PR [#95](https://github.com/tiangolo/dockerswarm.rocks/pull/95) by [@tiangolo](https://github.com/tiangolo).

## 0.3.0

* Add [GitHub Sponsors](https://github.com/sponsors/tiangolo) button. PR [#41](https://github.com/tiangolo/dockerswarm.rocks/pull/41).
* Simplify errors for env vars not set to make them consistent and re-usable. PR [#40](https://github.com/tiangolo/dockerswarm.rocks/pull/40).
* Upgrade everything to use Traefik v2. PR [#39](https://github.com/tiangolo/dockerswarm.rocks/pull/39).
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

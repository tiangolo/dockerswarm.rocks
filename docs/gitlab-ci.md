# GitLab CI runner for CI/CD

<a href="https://about.gitlab.com/" target="_blank">GitLab</a> is an open source Git code management system, similar to GitHub and Bitbucket.

It has integrated CI/CD (continuous integration and continuous deployment).

If you are using GitLab, you can run a GitLab CI "runner" in your Docker Swarm mode cluster to test, build and deploy automatically your code.

You could also test and build your code on "runners" in dedicated, isolated machines.

And then deploy it to your production Docker Swarm mode cluster using another GitLab CI runner configured in the same cluster.

## Create the GitLab Runner in Docker standalone mode

You probably want to run the GitLab runner in Docker standalone, even when you deploy it in a Docker Swarm mode Manager Node to deploy production stacks.

**Technical details**: This is because the Runner configurations will persist in the created container after the registration. If you create the GitLab Runner as a Docker Swarm mode service, your Runner could be deployed to a different Docker Swarm mode Manager Node the next time, or the container might be destroyed and replaced by a new one (even when running on the same machine), and then you would lose the registration configuration.

* To install a GitLab runner in a standalone Docker run:

```bash
docker run -d \
    --name gitlab-runner \
    --restart always \
    -v gitlab-runner:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner:latest
```

* Then, enter into that container with a Bash session:

```bash
docker exec -it gitlab-runner bash
```

* In a web browser, go to the GitLab "Admin Area -> Runners" section.
* Get the URL and create an environment variable in your terminal running inside the Docker container, e.g.:

```bash
export GITLAB_URL=https://gitlab.example.com/
```

* Get the registration token from the web browser in your GitLab and create an environment variable inside the Docker container, e.g.:

```bash
export GITLAB_TOKEN=WYasdfJp4sdfasdf1234
```

* Run the next command editing the name and tags as you need.

```bash
gitlab-runner \
    register -n \
    --name "Docker Runner" \
    --executor docker \
    --docker-image docker:latest \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
    --url $GITLAB_URL \
    --registration-token $GITLAB_TOKEN \
    --tag-list dog-cat-cluster,stag,prod
```

* The runner will appear in your GitLab web user interface, in the runner's section.

* You can edit the runner data from the GitLab admin section (name, tags, etc).

## Use it

GitLab CI is controlled using a `.gitlab-ci.yml` file that lives in your same code repository.

To learn more about it, you can check <a href="https://about.gitlab.com/product/continuous-integration/" target="_blank">GitLab CI's official documentation</a>.

If you have a Docker Swarm mode cluster with a main Traefik proxy set up using the ideas from <a href="https://dockerswarm.rocks" target="_blank">DockerSwarm.rocks</a>, your `.gitlab-ci.yml` file could look like:

```YAML
image: tiangolo/docker-with-compose

before_script:
  - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY

stages:
  - build
  - deploy
  
build-prod:
  stage: build
  script:
    - docker-compose build
  only:
    - master

deploy-prod:
  stage: deploy
  script:
    - docker stack deploy -c docker-compose.yml --with-registry-auth my-stack
  only:
    - master
```

To see more complete examples, check the <a href="https://dockerswarm.rocks/project-generators/" target="_blank">Project Generators at DockerSwarm.rocks</a>.

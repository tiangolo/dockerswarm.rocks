# Docker Swarm Mode or Kubernetes

I was a big fan and advocate of Docker Swarm Mode.

Unfortunately, it seems **Kubernetes** has the overwhelming majority of the market. 🥲

I think Docker Swarm Mode was a great product, and I didn't understand why it didn't get more adoption and kept growing.

Right now, I think **it's not sensible to build a new product using Docker Swarm Mode**, just because there's no strong support behind it.

I still think Kubernetes is overly complex for most cases, and I currently don't know of a better alternative (and I've tried several).

I think deployment is not a solved problem.

Maybe another abstraction layer simplifying everything is needed. Or maybe cloud services and platform as a service providers are just a better option now that the alternative is to go through the enormous complexity of Kubernetes.

The safest bet right now seems to be using Kubernetes or managed cloud services, not Docker Swarm Mode.

Given that, this website is *deprecated*, and kept around mainly for historical reasons.

Sadly, I don't have a good alternative or set of guides to recommend or offer right now. You can follow me on [Twitter](https://twitter.com/tiangolo) or [LinkedIn](https://www.linkedin.com/in/tiangolo/) to see what else I discover in the future.

Meanwhile, you could check the [awesome-swarm](https://github.com/BretFisher/awesome-swarm) list for more resources about Docker Swarm Mode. 🤓

## History

Docker, Inc., the company, created Docker Swarm Mode, it was integrated in Docker, it was the competition of Kubernetes.

At some point, Docker, Inc. decided to shift focus to Docker Desktop and other products. They sold all the Docker Swarm Mode side to Mirantis, Inc. that inherited the enterprise clients. And now [Docker Swarm Mode is a Mirantis product under their Kubernetes engine](https://www.mirantis.com/software/swarm/).

As someone looking from the outside, it seems Mirantis bought it to continue providing the enterprise support for the existing clients while probably helping them migrate to Kubernetes.

It probably wouldn't make sense to create a new project based on Docker Swarm Mode instead of Kubernetes, when the current main company behind it is seemingly also fully focused on Kubernetes.

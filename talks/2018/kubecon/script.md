footer: KubeCon EU - @dantoml [she/her] 🌈
slidenumbers: true

## Write Less __Code__,
## Use More __Tools__
### @dantoml

---

## $ whoami

^
Hey, I'm Dani. I live in Berlin, and I'm a Staff Software Engineer on the build execution team at CircleCI. I'm also a maintainer of CocoaPods, a dependency manager for Cocoa applications, and fastlane, build and release automation tooling for mobile engineers.

---
[.background-color: #212121]

![50%](https://assets.brandfolder.com/otz73u-2kwjjs-3sbkgf/original/circle-logo-horizontal-white.png)

^
For those of you who haven't heard about CircleCI before, CircleCI is a continuous integration and delivery platform with first class support for Docker, macOS, and Linux VMs.

---
[.build-lists: true]

## What am I covering?

- Why we replatformed
- Why we chose to run Kubernetes and Nomad
- How our deployment has evolved

^
- So what am I going to cover in this talk? [click]
- I'm going to start out by covering why we decided to replatform our infra [click]
- I'm then going to talk about why we specifically chose to use Nomad and Kubernetes [click]
- And then I'm going to talk about how our deployment has evolved, in a series of post-mortems that show some of the barriers we faced, and importantly, how we managed to overcome them.

---

## CircleCI 2.0

^
Almost a year ago, we launched the current generation of the CircleCI infrastructure, aptly named, 2.0. It was a huge amount of work, that involved everyone inside the company by the time it was released - which might sound a little strange coming from a CI/CD company.

---

## Why __2.0__?

^
CircleCI was founded in 2011, and this was a very different time for many software companies. Many places were not yet doing CI/CD, we didn't yet have Docker, and many teams deployed AMIs, or were running configuration management tools against hosts in production to git pull and bundle exec rake serve.

^
The platform was originially designed around being fairly opintionated about how your CI/CD process should work, with explicit sections in config for build/test/deploy - and this was great when we were all writing monolithic applications with relatively simple deployment piplines.

---

## The industry was changing

^
In the time since then, however, the industry has changed a lot. Docker was released in 2013, and since then, it's had an incredibly broad impact on how teams are building and releasing software.
We could see where our own and the wider industries requirements around CI/CD were going, but our existing infrastructure was rapidly becoming a problem - and we couldn't see an incremental path to servicing them.

---

# 🙀

^
We eventually decided that our best option was to re-platform our core infrastructure, to allow us to service these ever changing needs.

^
Replatforming is terrifying, it's a lot of work. You're trying to rebuild a better version of an existing product - and you have to work without the guidance of constant customer feedback and validation. Our customers were also scared, because it looked like we weren't really developing new features any more.

^
However, you have the advantage of existing domain knowledge and understanding of what you need to get from your new infrastructure, and the lessons of past experiences in what did not work as we scaled.

---

## Requirements

^
We had a couple of key components to think about when we started down the road to 2.0. We needed to handle both the needs of our build infrastructure, and a way to scale out our platform as we migrated from a monolith to microservices.

---

## Build Infrastructure

^
To many engineers, running other people's code on your infrastructure, and on the same servers as other untrusted code is terrifying - and it is.
Because of this, we have to bake isolation into everything we build, and we have to do it from the beginning.
There were only a few ways we could do isolation in a _reasonably_ safe way, and the one we picked for CircleCI 1 was LXC.
On 2.0, alongside wanting to support containerized builds, we also wanted to be able to better support builds in alternative environments, such as full linux or macOS vms, for supporting things like privalliged docker and container building.

---

### Build Infrastructure
## LXC and Pluggability

^
Over the years, we’ve become pretty adept at managing the nuances of LXC and its tools — we know what output they generate, and we understand the edge cases. The platform team wanted to capitalize on that expertise by picking a tool that shared many of these qualities.
We also didn't want to necessarily be tied in to any given technology or abstraction for this however, because we don't just schedule builds inside containers - We also provide support for running full linux or macOS virtual machines.

---

### Build Infrastructure
## Fast Scheduling

^
At our peak we push thousands of jobs per minute, and although it's rarely the bottleneck of our critical patch, many schedulers cannot handle running large volumes of short lived batch jobs.

^
Many job schedulers that are primarily designed for running services or long running processing tasks tend to have fairly slow schedulers that optimise for finding the optimal placement for a given task, usually by trying to find the best fit from a large pool of the available placement nodes. These types of scheduling algorithms tend to become quite slow when scheduling a large number of tasks on a large cluster of worker nodes.

---

### Build Infrastructure
## Operational Simplicity

^
We also ship an on-premise product - so any third party tool we ship as part of our product needs to be easy to operate by people who don't necessarily do this full time, with good documentation, and easy scriptability.

---

## Service orchestration 

^
We also wanted a great platform to run the services that compose our product features and allow us to scale out beyond the single instance that was the development incarnation of 2.0.

^
Our initial search landed us with two main options, Mesos and Nomad. 

---

## Mesos

^
Mesos had the advantage of being extremely robust and battle tested, operating at large scale in places like Twitter, AirBnB, and Apple. However, a big part of its ability to support those organisations rests on significant overhead for operators.

^
To use Mesos you need to use an orchestration framework like Marathon or Chronos - and depending on the one you choose, you get a significantly different Mesos experience. Marathon focuses on services, and Chronos focuses on cron jobs. 

---

## Not right for us

^
For our build infrastructure however, we needed support for batch jobs, and Mesos didn't have a great solution for this.

^
Mesos is great, but it definitely was not the right tool for us, with the engineering capacity that we had available. We'd have had to write a lot of code to get it to do the things that we needed, and to get the isolation we require for running customer code.

---

![75%](/Users/danielle/Downloads/nomad.png)

^
We already use a bunch of the Hashicorp stack, provisioning our infrastructure with Terraform, and building our AMI's with Packer - so HashiCorp's still very young nomad was an interesting contender. 

---

### Nomad
## Great at batch jobs

^
Unlike many orchestrators, nomad can choose between multiple scheduling algorithms, and for batch jobs uses an algorithm based on the Berkley Sparrow scheduler, that applies the power of two choices load balancing technique to job scheduling. This results in a job scheduler that is incredibly fast.

---

### Nomad
## Pluggable

^
Nomad's executor is also quite pluggable, with built in support for running various container technologies, or simply executing a binary in place, allowing us to precisely control how we execute our customer jobs - this is an incredibly useful facility for us.

---

### Nomad
## Cooperative API

^
It also exposes much of its functionality and state through a rest and rpc api. This is both great for building out our scheduling capabilities, but also really useful for building out tools on top and around nomad to augment its capabilities with our own needs, when things would not necessarily be suitable for upstreaming.

---

### Nomad
## Still Evolving 

^
Nomad was also still very young, and we could talk with the nomad team and influence its direction, and contribute our own patches upstream as necessary with relative ease.
It's also still rapidly evolving, with many ops changes coming that allow us to remove many of our own out of band patches, or sidecar tooling.

---

### Nomad
## Great for our jobs, but...

^
That being said, this youthfullness came with some problems with our initial prototypes, whereby there were various stability and operational issues that would force us to ssh in and restart the server process, because things would be wedged, or somewhat randomly failing.

^
Because of our desire for strong isolation for builds and internal services however, this lead us to an interesting realisation - we didn't need to use the same tool for both our job execution and our internal infrastructure.

---

## One size doesn't fit all?

^
We realised that we could use other tools to run our services and infrastructure, to leverage those tools, without tying us in to any particular toolset over time, and to allow us to fix our issues and concerns. So we started looking into other orchestrators - we still didn't want to operationalise Mesos, and eventually we settled on Kubernetes.

---

![100%](https://github.com/kubernetes/kubernetes/blob/master/logo/name_white.png?raw=true)

^
Unlike many of the other orchestration options, Kubernetes was already approaching the point of being incredibly stable, used in a growing number of large scale production environments, and had a wonderful community. We also had members of our SRE team that had used it in previous jobs.

^
The lifespan and adoption of a tool give us confidence that others have encountered similar problems and (perhaps) know how to solve them. Being able to go to an IRC channel or mailing list and have people have potential ideas about how to solve the problem is a wonderful thing. 

---

### Kubernetes
## Community

^
A big part of why we didn't use Mesos was that a lot of its operational knowledge was locked down in the silos of large enterprises - and nobody on our team already knew how to use it, and filling in those gaps without a community would've been an impossible mountain.

^
The ecosystem and community of kubernetes gave us the confidence we needed to push for its use and ship it to production.

--- 

### Kubernetes
## Rolling Updates and Readiness Checks

^
The kubernetes support for rolling deployments with readiness checking was also incredibly valuable to us.
We continuously deliver all of our software, and regardless of how much you test a service, knowing that if you push a bad config change, your service will crash, and not be rolled out to all of the pods is incredibly warm.
And in the case of running a stateful service like nomad, it also allows you to maintain quorum during deployments, which was one of the more frustrating issues during our nomad testing phases.

---

## Nomad is __part__ of our product

^
We ultimately made an important distinction - Nomad is part of our product. It is an important part of running builds in 2.0, and it ships to both our on-premise product and in our SaaS offering. It's where the majority of our builds execute, and gives us much of the flexibility we need to do that and build out new functionality.

---

## Kubernetes is for __scaling__ our product

^
Kubernetes is the platform that we use to build out and to operationalize our product. We leverage its features to allow us to run our application layer services - and to provide abstractions and automation over our other cloud resources.

---

## So how does this work?

^
So now we know the basic tools at our disposal, lets take a look at how we set this up, and explore some of the things that did not necessarily work so well for us and how we leveraged the tools to fix them.

---

## scheduler
## __inside__ 
## a scheduler

^
From an operations perspective, nomad is comprised of two main components, the nomad server, which is the nomad control plane, and nomad-clients, which execute the jobs provided by the nomad server. We run the nomad servers with kubernetes, and the clients are external. And for us, a nomad deployment also has some auxiliary services that exist to integrate Nomad with the rest of our domain, monitoring, and some amount of ops automation.

---

## nomad-__server__

^
Nomad is an inherently stateful service, built on top of raft for handling log replication, and with a couple of different persistence options. Nomad requires some form of service discovery for initial cluster bootstrapping, after which a gossip protocol takes over with HashiCorp Serf.

---

### Nomad
## Persistence

^
We handle nomad storage by having dedicated kubernetes clients for nomad server workloads, which we then mount the host storage of into a nomad server container. This allows us to leverage our existing practices for managing persistent services and simplifies our kubernetes configuration.

---

### Nomad
## Service Discovery

^
Because nomad is a distributed stateful service, it also requires some way to do discovery of other server nodes, and there are a few different ways in which we can do this.

---

## Consul

^
HashiCorp recommend using Consul for Nomad service discovery, and it works reasonably well, with relatively little overhead to get setup and configure, and is a very stable piece of software. However, we initially set up Consul as a little bit of a snowflake cluster for our initial nomad deployment, and it was not running within kubernetes.

---

## Multiple clusters

^
So when we started expanding our nomad requirements to support multiple clusters, we found we were initially using the same Consul cluster for multiple deployments of Nomad, which, when it comes to minimising shared dependencies and isolating clusters was not an ideal situation, so we started evaluating alternatives, before trying to deploy another stateful service to kubernetes.

---

## Kubernetes + StatefulSet

^
We eventually decided on switching Nomad into using a Kubernetes StatefulSet, which gave us more predictable upgrade semantics, and also allowed us to do our initial nomad server bootstrapping using kubernetes for service discovery.

---

## nomad-__clients__

^
Nomad clients are where jobs actually execute, and these don't run inside kubernetes, but instead within an AWS AutoScaling Group.

---

## Scaling Clients

^
We scale clients with a fairly simple autoscaler that runs in Kubernetes, and this is driven by our metrics around available capacity, and is easy to shutdown or override when we want to over provision for expected spikes when recovering from backlogs that occur during incidents or when cutting over to a new cluster.

---

## Draining Clients

^
We originally used a patched version of nomad to support draining clients without rescheduling jobs on other nodes - and a nomad draining service that consumes the aws lifecycle hooks to enable and disable draining as required.
Nomad 0.8 adds first class support for controlling the drain behaviour of clients, and as we start our testing and upgrades, it is fairly simple to update our drainer to use the first class API for this.

---

## Health Checks

^
We run an agent on nomad-clients that validates that nomad is up, the status of the docker engine, and the internet reachability.
This then reports to both statsd and the aws health checking, so that we can automatically drain and remove unhealthy nodes without manual intervention.

---

## What have we learnt?

^
I would like to start out by saying that nomad is a really great tool, and Alex and the nomad team are incredibly receptive to hearing about issues that you run into. They've worked on fixes for, or roadmapped features for many of the things that we've experienced, and are a super friendly crowd.
Nomad is however still very young, and we've augmented lots of its more unfinished behaviour with kubernetes while we work on more long term fixes for some of the problems. 

---

## Managing __outages__

![120%](/Users/danielle/Desktop/Screen Shot 2018-04-30 at 13.23.47.png)

^
The first major set of production nomad outages we experienced was in January, and taught us a lot both about some less obvious behaviour in nomad, and resulted in some changes to how we deploy our clusters to kubernetes.

---

## The Symptoms

^
The symptoms were quite interesting, we initially got paged for the nomad job queue starting to rise quickly - despite us having plenty of capacity to run user jobs. And all of the nomad clients and servers appeared to be up, servicing requests, and generally being healthy by our metrics.

---

# 😕

^
We were confused by this, because we were definitely no where near our peak load, but as we looked deeper, we saw that the nodes were simply not receiving jobs any more.
So we started to take a look into the nomad-server containers, and saw that almost all of their logging had stopped, outside of a significant number of `failed to delete job` logs.
In an attempt to unwedge the hosts, we started cycling nomad-server pods, while being careful to maintain quorum, and this allowed some amount of jobs to pass through.

---

## Declaring Bankruptcy

^
Eventually we made a tough call - it was approaching the time of day when our load was about to significantly increase, so we turned off our build scheduler, and allowed the running builds to drain. We then dropped the nomad database, and bought up the cluster with completely fresh state.

---

## Follow up

^
When we were re-analyzing the nomad configuration and source code after we'd recovered, we noticed that we were missing some options that would give us greater visibility into the operations of its internal state machine, so we quickly enabled them, so we could get more data the next time the incident occured.
As it turned out, we wouldn't have to wait long...

---

## The next day

![120%](/Users/danielle/Desktop/Screen Shot 2018-04-30 at 13.25.40.png)

^
The next day we were once again greeted by a rising job queue, but this time, we had the metrics we needed to identify the issue. We were quick to declare bankruptcy after validating we had the data we needed, and doing some manual inspection of the running state.

---

## Analysis

^
When analyzing the data we'd collected, we eventually realized that the issue was occuring during garbage collection cycles, where nomad will cycle through its completed jobs, and remove them from the database.

---

```go
for _, job := range gcJob {
		req := structs.JobDeregisterRequest{
			JobID: job.ID,
			Purge: true,
    }
    ...
}
```

^
The original implementation of garbage collection in Nomad would iterate over every job that needed to be garbage collected, and issued a JobDeregister request, which would take a slot in the raft log, which would need to be applied to the finite state machine.
Each of these requests would then call out to the current nomad leader, and block until the deletion had been applied. Which was not ideal when several hundred thousand jobs would need to be deregistered at once.

---

## Tuning garbage collection

^
We initially tried reducing the cooldown for GC from the default 4 hours, down to about 30 minutes, which resulted in much more stable behaviour. However it wasn't quite perfect, around our peak times, scheduling would halt for a a few seconds every gc cycle, which was just slow enough to trip our pending jobs alerts.

---

## `nomad-gc`

^
Because we only really use the nomad api to inspect jobs that have failed, or during incidents, we don't really need to keep jobs in a successful terminal state around, so we wrote a small sidecar service that uses the nomad api to continuously collect successful dead jobs, and we use the built in collector as a failsafe in case that our service is unavailable, or cannot reach the nomad api.
This has resulted in signficiantly more stable operation and was signficiantly easier to deploy through our usage of kubernetes to orchestrate nomad.

---

## Nomad 0.8

^
The latest version of nomad now actually uses a batched deletion request which should drastically improve performance under these types of workloads, and I'm excited to begin testing that soon.

---

## Multi-Cluster

^
At around the same time, we went from running a single nomad cluster, to running multiple clusters. This changed a little bit of how we deploy them, and how we integrate with kubernetes. 

---

## Terraform, terraform, terraform

^
We deploy almost all of our infrastructure using terraform, because it gives us great visibility into changes to our deployment, and allows us to reason about and easily change our configuration.
One of the first steps in migrating to a multi cluster deployment was to modularlise our terraform to allow us to easily create multiple downstream clusters with a minimal amount of duplication.

---

## 💜 Helm

^
We then migrated our kubernetes manifests to helm. We actually package the entire deployment, alongside the gc, drainer, metrics generator, etc into a single helm package. This makes it reasonably simple to be able to both CD changes, while also being able to manually deploy a cluster if the platform is unavailable, or when working in a development cluster.

---

## Namespaces

^
We then deploy each cluster to its own namespace in kubernetes, which acts as a neat grouping mechanism for all of the related services, allows us to easily manage an individual cluster, and acts as a good isolation boundary for services that want to talk to nomad.

---

## Service Discovery

^
The use of kubernetes namespaces, alongside with a stateful set for kubernetes also makes it much easier to handle both auxillary services to discover their related nomad deployment, and also for nomad servers to discover each other via kubernetes, as they have determensitic DNS names provided by the stateful set, and after an initial connection, Serf takes over.

---

# 🦄

^
So in summary, kubernetes, helm, and terraform have made it sigificantly easier to orchestrate our build system, and I'm personally really happy with the power and control they've given us over that, and the ways that they've given us to handle our clusters - and also with Nomad, which is continually improving and becoming easier to operate, while still being incredibly fast at scheduling our customers workloads.

---

## __thank you__.

### @dantoml


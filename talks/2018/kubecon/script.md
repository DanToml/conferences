footer: KubeCon EU - @dantoml [she/her]
slidenumbers: true

## Write Less __Code__,
## Use More __Tools__
### @dantoml

---

## $ whoami

^
Hey, I'm Dani. I'm a Staff Software Engineer working on build infrastructure at CircleCI. I also maintain some open source tools used by a lot of mobile developers, CocoaPods, and Fastlane.

---
[.background-color: #212121]

![50%](https://assets.brandfolder.com/otz73u-2kwjjs-3sbkgf/original/circle-logo-horizontal-white.png)

^
For those of you who haven't heard about CircleCI before, CircleCI is a continous integration and delivery platform with first class support for Docker, macOS, and Linux VMs.
{{should elaborate here, but don't wanna spend more than a little bit of time on this}}

---
[.build-lists: true]

## What am I covering?

- Why did we replatform
- What technology choices we made
- The issues we faced in production
- How we solved them

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
Replatforming is terrifying, it's a lot of work. You're trying to rebuild a better version of an existing product - and you have to work without the guidance of constant customer feedback and validation. 

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
## LXC

^
Over the years, we’ve become pretty adept at managing the nuances of LXC and its tools — we know what output they generate, and we understand the edge cases. The platform team wanted to capitalize on that expertise by picking a tool that shared many of these qualities.
We also didn't want to necessarily be tied in to any given technology or abstraction for this however.

---

### Build Infrastructure
## Fast Scheduling

^
At our peak, and when recovering from outages, we push thousands of jobs per minute, and although it's unlikely to be the bottleneck of our critical path, many schedulers cannot handle running large volumes of short lived batch jobs.
They often break down due to overretention of data, or get bogged down with functionality that only really make sense if you are scheduling long running services.

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
Many job schedulers that are primarily designed for running services or long running processing tasks tend to have fairly slow schedulers that optimise for finding the optimal placement for a given task, usually by trying to find the best fit from a large pool of the available placement nodes.
However, these types of scheduling algorithms tend to become quite slow when scheduling a large number of tasks on a large cluster of worker nodes.
When you're scheduling short lived batch jobs, past a certain point the optimality of placement is often less important.
Nomad's batch scheduler uses an implementation based on the Berkley Sparrow scheduler, that applies the power of two choices load balancing technique to job scheduling.
This results in a job scheduler that is incredibly fast.

---

### Nomad
## Pluggable

^
Nomad's executor is also quite pluggable, with built in support for running various container technologies, or simply executing a binary in place, allowing us to precisely control how we execute our customer jobs - this is incredibly useful for us and our goals. 

---

### Nomad
## Cooperative API

^
It also exposes much of its functionality and state through a rest and rpc api. This is both great for building out our scheduling capabilities, but also really useful for building out tools on top and around nomad to augment its capabilities with our own needs, when things would not necessarily be suitable for upstream use.

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
We realised that we could use other tools to run our services and infrastructure, to leverage those tools, without tying us in to any particular toolset over time. So we started looking into other orchestrators - we still didn't want to operationalise Mesos, and eventually we settled on Kubernetes.

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
The ecosystemm and community of kubernetes gave us the confidence we needed to push for its use and ship it to production.

--- 

### Kubernetes
## Rolling Updates and Readiness Checks

^
The kubernetes support for rolling deployments with readiness checking was also incredibly valuable to us - we continuously deliver all of our software, and testing isn't perfect. Knowing that if you push a bad config change, your service will crash, and not be rolled out to all of the pods is incredibly warm. And in the case of running a stateful service like nomad, it allows you to maintain quorum during deployments, which was one of the more frustrating issues during our nomad testing phases.

---

## Nomad is __part__ of our product

^
Nomad is an important part of running builds in 2.0, and it ships to both our on premise product and in cloud. It's where the majority of our builds execute, and gives us much of the flexibility we need to do that and build out new functionality.

---

## Kubernetes is for __scaling__ our product

^
Kubernetes is the platform that we use to build out operationalize our platform. It lets us  <blah blah?>

---

## So how did this work?

^
So now we know the basic tools at our disposal, lets take a look at how this initially worked, and then let's talk about some of the problems we ran into, and how we solved them.

---

### terraform, terraform, terraform

^
We deploy all of our persistent infrastructure with terraform, this is no different for our kubernetes clusters. It gives us great visibility into how and why every change into our infrastrucutre was made, 

---

## nomad-server Kubernetes Nodes

---

## `$ kubectl apply`

---

## nomad-drainer

---

Services deploy with helm

---

Service layer deployed with helm

---

Includes our auxillary services

---

Autoscaler

---

Garbage Collector

---

Metrics

---

## __thanks__.

### @dantoml



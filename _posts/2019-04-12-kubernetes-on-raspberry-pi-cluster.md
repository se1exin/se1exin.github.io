---
layout: post
title:  "Kubernetes on Raspberry Pi Cluster"
date:   2019-04-12 11:37:08 +1000
---

About two weeks ago I decided to build a bare-metal K8s cluster using Raspberry Pis to both scratch my project-itch,
and to move some services in my home network off a traditional linux box into docker containers.

Here's a glamour shot of my cluster: 

![alt text][header_image]

> I'll be posting some details on how to get things such as encrypted DNS, Sonarr, Couchpotato, and NZBGet working on an rpi K8s cluster in the near future (I'll link them up here as I write them).
..perhaps that's also a good hint at the 'services' I wanted to move off my linux box 😉.


Now I know there are already a bunch of blog posts out there that already cover this (many of which I used for inspiration),
so I won't be going over the very basics of buying the parts, flashing the pis, or joining the nodes all together.
For more info on the basics I highly recommend checking out [Alex Ellis' k8s-on-raspbian repo](https://github.com/alexellis/k8s-on-raspbian) (arguably the root of all raspberry pi k8s clusters).
I also found [Lyle Franklin's k8s-pi repo](https://github.com/ljfranklin/k8s-pi) very helpful as well.
  
While I did use many sources for reference while building my cluster, including the two above, I didn't actually end up running any of their scripts - as I wanted to set my cluster up manually to learn more about k8s.
I did however come across a few things during my manual setup that were not immediately obvious or easy to find online  - specifically *load balancer networking* and *persistent storage*.
I'm going to go over how I solved **load balancer networking** in the rest of this post, and will cover **persistent storage** in a later post (will link it up once it is published).
  
## Load Balancer Networking

First thing after getting setup that I was not entirely clear on was how to expose my k8s services to my home network.
I know that NodePort is an option, but *I want Load Balancing* - otherwise whats the point right?!

Typically when running k8s on a cloud provider you just create an Ingress and wohla, you get a Load Balancer with a public IP address. Unfortunately on bare-metal it's not that simple.
Fortunately however, it isn't complicated either ..enter 🤘 [MetalLB](https://metallb.universe.tf/) 🤘.

I won't rewrite [MetalLB's 'Why' section](https://metallb.universe.tf/#why), which gives a better description of the package than I can - just know that it is super simple way to get k8s load balancing working on an rpi cluster on a typical home network.

> It's probably also worth mentioning at this point that my cluster is **not** public internet-facing. MetalLB is still in beta at the time of writing, and not 'ready' for production use - although I personally have not had any issues with it.

MetalLB has two alternate config options - [**Layer 2**](https://metallb.universe.tf/configuration/#layer-2-configuration) and [**BGP**](https://metallb.universe.tf/configuration/#bgp-configuration). 
My router is pretty standard and dumb (at least compared to cloud networking) and does not support BGP, nor did I want to mess with my perfectly stable router anyway, so I went the route of Layer 2.
I recommend this as well if you don't want to mess with your router, and have the typical amount of home devices on your network (e.g. 10-30) - meaning your IP address pool has lots of capacity.
You can read more about the differences between the two via MetalLB links above.

MetalLB has [a tutorial page on installing and setting up with Layer 2](https://metallb.universe.tf/tutorial/layer2/), but in case your lazy here's the gist of it:

I choose to go the route of installing MatelLB directly (that is, **without** Helm) using the provided manifest (see the [Installation page](https://metallb.universe.tf/installation/)). 

```
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
```

After that there was only one more step - to create a metallb-config `ConfigMap` defining which IP range it may use (among other things), and apply it.
For reference, my home network uses the IP range `10.1.1.1 - 10.1.1.254`. I decided to define my MetalLB Layer 2 pool as `10.1.1.150-10.1.1.170`,
as my home network DHCP allocations are very unlikely to reach 150.
If your following along and installing MetalLB, your network IP range might be different, so just change your metallb-config to match your IP pool in an unreserved IP space.
If your playing with Kubernetes I'm going to assume you know either know, or can figure out, your network's topology. 

```
# File: metallb-configmap.yml

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.1.1.150-10.1.1.170
```

```
kubectl apply -f metallb-configmap.yml
```

That's It! Now I'm able to create k8s `Service`s with `Type: LoadBalancer` and I'll be given an IP on my home network that is accessible from any other device on the network.

For example, here is my DNSCrypt Proxy k8s `Service`:

```
apiVersion: v1
kind: Service
metadata:
  name: dnscrypt-proxy
  labels:
    app: dnscrypt-proxy
spec:
  type: LoadBalancer
  selector:
    app: dnscrypt-proxy
  ports:
  - name: dnscrypt-proxy
    port: 53
    targetPort: 5353
    protocol: UDP
```

And here is its exposed to the network with an IP from the defined pool of `10.1.1.150-10.1.1.170`:
```
$ kubectl get services
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dnscrypt-proxy   LoadBalancer   10.104.238.191   10.1.1.150    53:32460/UDP   9h
```

[header_image]: /assets/img/2019-04-12-rpi-cluster.jpg "4x Raspberry Pi Model 3+ Cluster"

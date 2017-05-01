# kubernetes-playpen
Playpen repository for Kubernetes experiments.

## Running Kubernetes locally via Docker
The Kubernetes [documentation](https://kubernetes.io/docs/home/) illustrates many different ways to install Kubernetes, however most of those are, quite reasonably, focussed on installing on a multi-host cluster.

The currently recommended way of installing on a single node is to use [minikube](https://github.com/kubernetes/minikube) however this *currently* fires up a Virtual Machine rather than running Kubernetes in containers on the host. I believe that there is work in progress to allow minikube to be run directly in containers on the host but that's not yet complete.


In the Kubernetes [Picking the Right Solution](https://kubernetes.io/docs/setup/pick-right-solution/) there *was* a link to a **Portable Multi-Node Cluster** that ran Kubernetes in Docker, but I never managed to get that to work properly and moreover it seems to have disappeared from the *Picking the Right Solution* documentation. Doing a search for "docker-multinode" yields this page: https://kubernetes.io/docs/getting-started-guides/docker-multinode/, which indicates that the docker-multinode guide has been superseded by [kubeadm](https://kubernetes.io/docs/getting-started-guides/kubeadm/) though nothing in the kubeadm guide implied running on containers, rather it looks like a bare metal install. The original docker-multinode guide seems to be mirrored here https://kubernetes-io-vnext-staging.netlify.com/docs/getting-started-guides/docker-multinode/


In the mean time in order to run Kubernetes locally via Docker there are instructions here https://zwischenzugs.wordpress.com/2015/04/06/play-with-kubernetes-quickly-using-docker/ and here http://janetkuo.github.io/kubernetes/v1.0/docs/getting-started-guides/docker.html which both present a similar approach though do **note** that they are a little out of date and using [minikube](https://github.com/kubernetes/minikube) is likely to be a better supported approach.

I also came across [kid - Kubernetes in Docker](https://github.com/vyshane/kid) though I've not tried that yet.



### Getting started

The instructions in the guides cited above are similar to each other but they differ on things like the versions of etcd, hyperkube and kubectl used.

For example:
```
docker run --net=host -d gcr.io/google_containers/etcd:2.0.9 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
```
```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  gcr.io/google_containers/hyperkube:v0.21.2 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests
```
```
docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v0.21.2 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```

Note that this is using etcd version 2.0.9, whereas https://github.com/kubernetes/kubernetes/tree/master/cluster/images/etcd suggests that the current version being used is 3.0.17 and is using Kubernetes version v0.21.2, which is quite old.


Get the current Kubernetes version so that subsequent steps can use more up to date versions of Kubernetes:
```
https://dl.k8s.io/release/stable.txt
```

### Step One: Run etcd

It turns out not to be quite as simple as replacing gcr.io/google_containers/etcd:2.0.9 with gcr.io/google_containers/etcd:3.0.17 as --addr and --bind-addr seem to have been deprecated in etcd v3.
```
-addr=:0: DEPRECATED: Use -advertise-client-urls instead.
-advertise-client-urls=http://localhost:2379,http://localhost:4001: List of this member's client URLs to advertise to the rest of the cluster

-bind-addr=:0: DEPRECATED: Use -listen-client-urls instead.
-listen-client-urls=http://localhost:2379,http://localhost:4001: List of URLs to listen on for client traffic

-data-dir=: Path to the data directory

-peer-addr=:0: DEPRECATED: Use -initial-advertise-peer-urls instead.
-initial-advertise-peer-urls=http://localhost:2380,http://localhost:7001: List of this member's peer URLs to advertise to the rest of the cluster

-peer-bind-addr=:0: DEPRECATED: Use -listen-peer-urls instead.
-listen-peer-urls=http://localhost:2380,http://localhost:7001: List of URLs to listen on for peer traffic

-peers=: DEPRECATED: Use -initial-cluster instead
-peers-file=: DEPRECATED: Use -initial-cluster instead
-initial-cluster=default=http://localhost:2380,default=http://localhost:7001: Initial cluster configuration for bootstrapping
```
See also: https://github.com/coreos/etcd/blob/master/Documentation/op-guide/configuration.md

Using the new commands instead of the deprecated ones gives:
```
docker run --net=host -d gcr.io/google_containers/etcd:3.0.17 /usr/local/bin/etcd -advertise-client-urls=http://127.0.0.1:4001 -listen-client-urls=http://0.0.0.0:4001 -data-dir=/var/etcd/data
```

### Step Two: Run the master
In the instructions in the previously cited guides running the master uses the following instructions:
```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  gcr.io/google_containers/hyperkube:v0.21.2 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests
```

That actually seems to work, resulting in a running hyperkube container and pause containers, however simply replacing the version with a more recent version (e.g. v1.6.2) e.g.

```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  gcr.io/google_containers/hyperkube:v1.6.2 /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=127.0.0.1 --config=/etc/kubernetes/manifests
```

results in a stopped container indicating that something is broken in the configuration.



## Under the hood
The best place to look to understand exactly *what* Kubernetes is doing as opposed to blindly making use of the work of others is the [Creating a Custom Cluster from Scratch](https://kubernetes.io/docs/getting-started-guides/scratch/) guide.


## Useful Links
Kubernetes Documentation: https://kubernetes.io/docs/home/

Play with Kubernetes quickly using Docker: https://zwischenzugs.wordpress.com/2015/04/06/play-with-kubernetes-quickly-using-docker/

Running Kubernetes locally via Docker: http://janetkuo.github.io/kubernetes/v1.0/docs/getting-started-guides/docker.html

Launch Kubernetes Cluster Tutorial: https://www.katacoda.com/courses/kubernetes/launch-cluster



# hyperkube

a Makefile to build your own hyperkube image

## Summary
This Makefile and Dockerfile can be used to build your own version
of hyperkube.  There is a dependency on the kubernetes source.


## Usage
### to make the current image hyperkube
```
make DOCKER_REPO=quay.io/foobar  V=v1.0.1
```
#### Make Variables
```
DOCKER_REPO - registry (default: bogus/repo)
V - container image tag (default: unknown)
KUBEROOT - location of base kubernetes source tree (default: ../../kubernetes)
```
#### Make Targets
```
all 
   hybercube.tmp
      KUBEROOT/_output/dockerized/bin/linux/amd64/hypercube
         (KUBEROOT/build/run.sh hack/build-go.sh) 
   hypercube
clean (clean local build and images)
clean-all (clean + clean kubernetes)
   KUBEROOT/make clean
push (upload to given DOCKER_REPO)
```
## Notes

If you do not supply the V flag, and enable the current version check in the Makefile.  The current version
is calculated from the script:
```
wget -q -O- https://storage.googleapis.com/kubernetes-release/release/latest.txt
```

The V flag specifies the version of the hyperkube however no validation is performed that the kubernetes source tree is the coresponding version.

This uses Kubernetes's `build/run.sh hack/build-go.sh`  instead of `make` so we get a cross-compile from OS-X to linux via a golang build container.  *So, you have to have boot2docker running for this build to work on OS-X.*

#### Invoke:
##### MASTER
```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock  quay.io/mikeln/hyperkube:v1.0.1b /hyperkube kubelet --api_servers=http://localhost:8080 --v=2 --address=0.0.0.0 --hostname_override=$(hostname -i) --enable_server --config=/etc/kubernetes/manifests-multi --cluster_domain=kubernetes.local --cluster_dns=10.100.0.10

docker run -d --net=host --privileged quay.io/mikeln/hyperkube:v1.0.1b /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```
##### NODE

```
docker run --net=host -d -v /var/run/docker.sock:/var/run/docker.sock quay.io/mikeln/hyperkube:v1.0.1b /hyperkube kubelet --api_servers=http://10.22.6.241:8080 --v=2 --address=0.0.0.0 --enable_server --hostname_override=$(hostname -i) --cluster_domain=kubernetes.local --cluster_dns=10.100.0.10

docker run -d --net=host --privileged quay.io/mikeln/hyperkube:v1.0.1b /hyperkube proxy --master=http://10.22.6.241:8080 --v=2

```

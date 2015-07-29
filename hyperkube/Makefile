# build a hyperkube image.

.PHONY: all repo-warning clean clean-all

# this is the only variable you need to set.
# it can be set on the make command line:
# make DOCKER_REPO=mydockerhub
# or with an exported environment variable:
# export DOCKER_REPO=mydockerhub
# you can include the repo if not on dockerhub, like:
# make DOCKER_REPO=gcr.io/google_containers
#
DEFAULT_REPO := samsung_ag
DOCKER_REPO ?= $(DEFAULT_REPO)
#DOCKER_REPO ?= quay.io/mikeln

# this picks up the latest VERSION.  You can simply
# hard code it if you want to build an older one:
# V=v0.18.2 for example
#V ?= $(shell wget -q -O- https://storage.googleapis.com/kubernetes-release/release/latest.txt)
# the version is used to checkout from the kubernetes repo
DEFAULT_V := unknown
V ?= unknown
#V ?= v1.0.1

# relative to the cluster directory.  specific files are plucked from there.
# if your git checkout of kubernetes is not in ../../kubernetes,
# then set the variable KUBEROOT
KUBEROOT ?= ../../kubernetes
K=${KUBEROOT}/cluster

H=hyperkube
IMAGENAME = ${DOCKER_REPO}/$H

repo-warning:
	@if  [ $(DOCKER_REPO) =  $(DEFAULT_REPO) ]; then \
		echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
		echo "  You have not changed DOCKER_REPO from: $(DOCKER_REPO)"; \
		echo "  You MUST set DOCKER_REPO in your environment"; \
		echo "  or directly in this Makefile unless you are"; \
		echo "  building for the group"; \
		echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
		false; \
	else \
		echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
		echo "  Your DOCKER_REPO is set to: $(DOCKER_REPO)"; \
		echo "  Please execute 'make all' to build"; \
		echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
		if [ $(V) = $(DEFAULT_V) ]; then \
			echo "  You have not set the version in the V variable."; \
	 		echo "  Please set a value, otherwise '$(V)' will be used."; \
			echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
			false; \
		else \
			echo " You will build $(IMAGENAME):$(V)"; \
			echo "  Please execute 'make all' to build"; \
			echo "+++++++++++++++++++++++++++++++++++++++++++++++++++"; \
		fi \
	fi

all: $(H)


$(H): $(H).tmp master-multi.json.tmp master.json.tmp safe_format_and_mount.tmp Dockerfile
	@echo "building app $(VERSION)"
	docker build -t $(IMAGENAME):$(V) --rm=true --force-rm=true .
	@touch $@
	@docker images $(IMAGENAME)

$(H).tmp: $(KUBEROOT)/_output/dockerized/bin/linux/amd64/$H
	cp $(KUBEROOT)/_output/dockerized/bin/linux/amd64/$H $H.tmp
	sed -e "s/VERSION/$V/g" -e "s/gcr.io\/google_containers/quay.io\/mikeln/g" $K/images/hyperkube/master-multi.json > master-multi.json.tmp
	sed -e "s/VERSION/$V/g" -e "s/gcr.io\/google_containers/quay.io\/mikeln/g" $K/images/hyperkube/master.json > master.json.tmp
	cp $K/saltbase/salt/helpers/safe_format_and_mount safe_format_and_mount.tmp

$(KUBEROOT)/_output/dockerized/bin/linux/amd64/$H :
	(cd $(KUBEROOT)/build; ./run.sh hack/build-go.sh )
#	(cd $(KUBEROOT)/build; ./run.sh hack/build-cross.sh )
	#(cd $(KUBEROOT); make )

push:
	docker push $(IMAGENAME):$(V)
			
clean-all: clean
	(cd $(KUBEROOT); make clean)

clean:
	-docker rmi $(IMAGENAME):$(V)
	-rm -f *.tmp *.src $(H)



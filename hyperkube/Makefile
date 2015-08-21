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
DOCKER_REPO_ESC := $(subst /,\/,$(DOCKER_REPO))

# kubernetes source code location, relative to the cluster directory
# eg: KUBEROOT=$GOPATH/src/k8s.io/kubernetes
KUBEROOT ?= ../../kubernetes
K=${KUBEROOT}/cluster

# kubernetes version
# eg: V ?= $(shell wget -q -O- https://storage.googleapis.com/kubernetes-release/release/latest.txt)
DEFAULT_V := unknown
V ?= $(DEFAULT_V)
# allow VERSION=sha to use the git sha of HEAD in KUBEROOT
ifeq ($(V), sha)
	V := $(shell cd $(KUBEROOT) && git log -n1 --format='%h')
endif

BINARY=hyperkube
IMAGENAME = ${DOCKER_REPO}/$(BINARY)

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
		echo "  Your DOCKER_REPO is set to: $(DOCKER_REPO) ";\
		echo "  escaped: $(DOCKER_REPO_ESC)"; \
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

all: $(BINARY)


$(BINARY): $(BINARY).tmp master-multi.json.build master.json.build safe_format_and_mount.build Dockerfile
	@echo "building app $(VERSION)"
	docker build -t $(IMAGENAME):$(V) --rm=true --force-rm=true .
	@touch $@
	@docker images $(IMAGENAME)

$(BINARY).tmp: $(KUBEROOT)/_output/dockerized/bin/linux/amd64/$(BINARY)
	cp $(KUBEROOT)/_output/dockerized/bin/linux/amd64/$(BINARY) $(BINARY).tmp

master-multi.json.build: master-multi.json $(BINARY).tmp
	#sed -e "s/VERSION/$V/g" -e "s/gcr.io\/google_containers/$(DOCKER_REPO_ESC)/g" $K/images/hyperkube/master-multi.json > master-multi.json.tmp
	sed -e "s/VERSION/${V}/g" -e "s/REPO/${DOCKER_REPO_ESC}/g" master-multi.json > master-multi.json.build

master.json.build: master.json $(BINARY).tmp
	#sed -e "s/VERSION/$V/g" -e "s/gcr.io\/google_containers/$(DOCKER_REPO_ESC)/g" $K/images/hyperkube/master.json > master.json.tmp
	sed -e "s/VERSION/${V}/g" -e "s/REPO/${DOCKER_REPO_ESC}/g" master.json > master.json.build

safe_format_and_mount.build: $(BINARY).tmp
	#cp $K/saltbase/salt/helpers/safe_format_and_mount safe_format_and_mount.tmp
	cp $K/saltbase/salt/helpers/safe_format_and_mount safe_format_and_mount.build


$(KUBEROOT)/_output/dockerized/bin/linux/amd64/$(BINARY) :
	(cd $(KUBEROOT)/build; ./run.sh hack/build-go.sh )
#	(cd $(KUBEROOT)/build; ./run.sh hack/build-cross.sh )
	#(cd $(KUBEROOT); make )

push:
	docker push $(IMAGENAME):$(V)
			
clean-all: clean
	(cd $(KUBEROOT); make clean)

clean:
	-docker rmi $(IMAGENAME):$(V)
	-rm -f *.tmp *.build *.src $(BINARY)



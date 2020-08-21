
$(warning $(shell IMAGE_NAME=$(IMAGE_NAME) printenv | grep IMAGE_NAME))
ifndef IMAGE_NAME
	#$(warning IMAGE_NAME is not set)
	IMAGE_NAME=nginx-certbot
endif

# If we have `--squash` support, then use it!
ifneq ($(shell docker build --help 2>/dev/null | grep squash),)
DOCKER_BUILD = docker build --squash
else
DOCKER_BUILD = docker build
endif

all: build

build: Makefile Dockerfile
	$(DOCKER_BUILD) -t $(IMAGE_NAME) .
	@echo "Done!  Use docker run $(IMAGE_NAME) to run"

push:
	docker push $(IMAGE_NAME)

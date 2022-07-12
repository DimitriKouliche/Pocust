SHELL = /bin/bash

include build.env
export

DOCKER_IMAGE_BASE = $(DOCKER_REGISTRY_DEV)/$(DOCKER_REPOSITORY_NAME)
GIT_HASH = $(shell git rev-parse --short HEAD)


# Adapt environment variables based on BUILD_ID which is given in CI context
ifeq ("$(BUILD_ID)", "")
	BUILD_ID = $(APP_VERSION)-$(GIT_HASH)
	FINAL_APP_VERSION = $(BUILD_ID)
	IMAGE_TAG = $(APP_VERSION)
	USER_UID_USER_GID_BUILD_ARGS = --build-arg USER_UID=$(shell id -u $$USER) --build-arg USER_GID=$(shell id -g $$USER)
	PACKAGECLOUD_TOKEN=$(shell berglas access sm://ma-dev2/packagecloud-token\#1)
else
	FINAL_APP_VERSION = $(APP_VERSION)-$(GIT_HASH)
	IMAGE_TAG = $(BUILD_ID)
	USER_UID_USER_GID_BUILD_ARGS =
endif

DOCKER_IMAGE = $(DOCKER_IMAGE_BASE):$(IMAGE_TAG)
DOCKER_IMAGE_TEST = $(DOCKER_IMAGE_BASE):$(IMAGE_TAG)-test

.DEFAULT_GOAL := help

## all: Run all targets
.PHONY: all
all: init format style complexity test build

help: ## Display list of commands
	@grep -E '(^[a-zA-Z_-]+:.*?##.*$$)|(^##)' $(firstword $(MAKEFILE_LIST)) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[32m%-30s\033[0m %s\n", $$1, $$2}' | sed -e 's/\[32m##/[33m/'

## version: Prints the version as x.y.z-commitHash
.PHONY: version
version:
	@printf "%s-%s" "$(APP_VERSION)" "$(GIT_HASH)"


init: ## Bootstrap your application. e.g. fetch some data files, make some API calls, request user input etc...
	docker build --pull \
		--rm -t $(DOCKER_IMAGE_TEST) $(DOCKER_BUILD_CONTEXT) \
		--platform linux/amd64 \
		-f Dockerfile \
		--target test \
		--build-arg APP_VERSION=$(FINAL_APP_VERSION) \
		--build-arg PYTHON_VERSION=$(PYTHON_VERSION) \
		--build-arg PACKAGECLOUD_TOKEN=$(PACKAGECLOUD_TOKEN) \
		--build-arg USER=$(APP_NAME) \
		--build-arg APP_NAME=$(APP_NAME) \
		--build-arg BUILD_ID=$(BUILD_ID) \
		$(USER_UID_USER_GID_BUILD_ARGS) \
		&& docker tag  $(DOCKER_IMAGE_TEST) $(DOCKER_REGISTRY_DEV)/$(DOCKER_REPOSITORY_NAME):latest


.PHONY: build
build:  ## Builds the docker image associated with the project
	docker build \
		--target prod \
		--build-arg PYTHON_VERSION=$(PYTHON_VERSION) \
		--build-arg USER=$(APP_NAME) \
		--build-arg APP_NAME=$(APP_NAME) \
		--build-arg PACKAGECLOUD_TOKEN=$(PACKAGECLOUD_TOKEN) \
		--build-arg APP_VERSION=$(FINAL_APP_VERSION) \
		--build-arg BUILD_ID=$(BUILD_ID) \
		. -t $(DOCKER_IMAGE)

.PHONY: distribute
distribute: ## Push the docker image to the docker registry.
	docker push $(DOCKER_IMAGE)

shell-tests:
	docker-compose -p ${APP_NAME}-dev -f docker-compose-dev.yml run --rm app bash

format:
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml run --rm -T --no-deps \
	    	app bash -c "isort stress_tests tools tests; black ."

.PHONY: style
style: ## Check lint, code styling rules. e.g. pylint, phpcs, eslint, style (java) etc ...
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml run --rm -T --no-deps \
		app bash -c "flake8 ."
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml run --rm -T --no-deps \
		app bash -c "mypy ."
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml run --rm -T --no-deps \
	 	app bash -c "black --check ."

.PHONY: complexity
complexity: ## Cyclomatic complexity (McCabe) and maintainability check
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml run --rm -T --no-deps \
		app bash -c \
		"radon cc -s -n B . | tee /tmp/cc.txt && if [ -s /tmp/cc.txt ]; then exit 1; fi; \
		radon mi -n B . | tee /tmp/mi.txt && if [ -s /tmp/mi.txt ]; then exit 1; fi"

HEADLESS ?= 'false'
.PHONY: stress-tests
stress-tests: ## launch stress tests on a specified host
	docker run -p 8089:8089 \
	    -e LOCUST_HEADLESS=$(HEADLESS) \
	    -e LOCUST_LOCUSTFILE=../src/$(TEST_SUITE).py \
	    -e LOCUST_HOST=$(HOST) \
	    -v $(PWD):/home/ma_locust/app \
	    -it $(DOCKER_IMAGE)

## clean: Remove temporary files and docker images
.PHONY: clean
clean: ## Clean related docker ressources
	docker-compose -p $(APP_NAME)-dev -f docker-compose-dev.yml down -v --remove-orphans
	docker image prune -af --filter label=BUILD_ID=$(BUILD_ID) || true

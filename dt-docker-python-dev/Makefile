IMAGE := $(or $(IMAGE), dt-docker-python-dev)
ENV := $(or $(ENV),local)
TAG := $(or $(TAG),latest)
STAGE := $(or $(STAGE),$(USERNAME))
AWS_ACCOUNT := $(or $(AWS_ACCOUNT),373741672660)
AWS_ACCOUNT_NAME := $(or $(AWS_ACCOUNT_NAME), cbre-gcso-platform-dev)
REGION := $(AWS_DEFAULT_REGION)
AWS_DEFAULT_REGION := $(or $(AWS_DEFAULT_REGION), $(REGION), 'us-east-1')

BUILD_VERSION := $(or $(BUILD_VERSION), 0.0.1)
BUILD_TAG := $(or $(BUILD_TAG), base)
RELEASE_TAG := $(or $(RELEASE_TAG), runtime)
SAMPLE_TAG := $(or $(SAMPLE_TAG), sample)
DOCKER_REGISTRY := $(or $(DOCKER_REGISTRY),local)
PYTHONUNBUFFERED := 1

info:
	@echo "INFO: TAG = ${TAG}"
	@echo "INFO: IMAGE = ${IMAGE}"
	@echo "INFO: ENV = ${ENV}"
	@echo "INFO: STAGE = ${STAGE}"
	@echo "INFO: REGION = ${REGION}"
	@echo "INFO: AWS_ACCOUNT = ${AWS_ACCOUNT}"
	@echo "INFO: AWS_ACCOUNT_NAME = ${AWS_ACCOUNT_NAME}"

build: set-executable info
	docker build \
	 	--build-arg BUILD_VERSION=$(BUILD_VERSION) \
		--build-arg PYTHONUNBUFFERED=$(PYTHONUNBUFFERED) \
		--target base \
		--tag $(IMAGE):$(BUILD_TAG) \
		.
	
	docker run --rm \
		-e ARTIFACTORY_USER \
		-e ARTIFACTORY_API_KEY \
		-e BUILD_VERSION=$(BUILD_VERSION) \
		-e LINT=true \
		$(IMAGE):$(BUILD_TAG) \
		./abin/build.sh 
# initial build will throw an error because the release tag doesn't exist, don't worry about it the tag in this build takes care of it	
	docker build \
		--cache-from $(IMAGE):$(BUILD_TAG) \
		--cache-from $(IMAGE):$(RELEASE_TAG) \
		--tag $(IMAGE):$(RELEASE_TAG) \
		.

	# docker build \
	# 	--cache-from $(IMAGE):$(BUILD_TAG) \
	# 	--cache-from $(IMAGE):$(SAMPLE_TAG) \
	# 	--tag $(IMAGE):$(SAMPLE_TAG) \
	# 	.

shell: info 
	docker run --rm \
		-it \
		-e ENV=${ENV} \
		-e STAGE=${STAGE} \
		-e REGION=${AWS_DEFAULT_REGION} \
		-e AWS_ACCOUNT \
		-e AWS_ACCESS_KEY_ID \
		-e AWS_SECRET_ACCESS_KEY \
		-e AWS_SESSION_TOKEN \
		-e AWS_DEFAULT_REGION \
		-e TAG=${TAG} \
		${IMAGE}:${RELEASE_TAG} 

run: info 
	docker run -d \
		-p 5000:5000 \
		-e ENV=${ENV} \
		-e STAGE=${STAGE} \
		-e REGION=${AWS_DEFAULT_REGION} \
		-e AWS_ACCOUNT \
		-e AWS_ACCESS_KEY_ID \
		-e AWS_SECRET_ACCESS_KEY \
		-e AWS_SESSION_TOKEN \
		-e AWS_DEFAULT_REGION \
		-e TAG=${TAG} \
		${IMAGE}:${RELEASE_TAG} 

# utility rules
set-executable:
	chmod 755 ./abin/*.sh

ctrim:
	docker ps -a | grep Exited | cut -d ' ' -f 1 | xargs docker rm

itrim:
	docker image ls -a | grep none | xargs docker rmi

dnames:
	docker inspect -f '{{.Name}}' $(docker ps -aq --no-trunc) | cut -c2-
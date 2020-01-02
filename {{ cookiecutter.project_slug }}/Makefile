.PHONY: clean clean-model clean-pyc docs help init init-docker create-container start-container jupyter test lint profile clean clean-data clean-docker clean-container clean-image sync-from-source sync-to-source
.DEFAULT_GOAL := help

###########################################################################################################
## SCRIPTS
###########################################################################################################

define START_DOCKER_CONTAINER
if [ `$(DOCKER) inspect -f {% raw -%}{{.State.Running}}{%- endraw %} $(CONTAINER_NAME)` = "false" ] ; then
        $(DOCKER) start $(CONTAINER_NAME)
fi
endef
export START_DOCKER_CONTAINER

###########################################################################################################
## VARIABLES
###########################################################################################################

{%- if cookiecutter.use_nvidia_docker == 'yes' %}
DOCKER					:= nvidia-docker
{%- else %}
DOCKER					:= docker
{%- endif %}
TARGET					?= ""
PWD						= ${abspath .}
PROJECT_NAME			:= {{ cookiecutter.project_slug }}
IMAGE_NAME				:= $(PROJECT_NAME)-image
CONTAINER_NAME			:= $(PROJECT_NAME)-container
DATA_SOURCE				:= ${{ cookiecutter.data_source }}
JUPYTER_HOST_PORT		:= {{ cookiecutter.jupyter_host_port }}
JUPYTER_CONTAINER_PORT	:= 8888
PYTHON					:= python3


###########################################################################################################
## GENERAL TARGETS
###########################################################################################################

help: ## this help
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z_-]+:.*?##/ { printf "  \033[36m%-22s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)

## Non-Docker

install: ## install dependencies via pipenv
	pipenv install --ignore-pipfile

install-dev:
	pipenv install --ignore-pipfile --dev

##@ Docker

init: init-docker sync-from-source ## initialize repository for traning

sync-from-source: ## download data data source to local envrionment
{%- if cookiecutter.data_source_type == 's3' %}
	-aws s3 sync $(DATA_SOURCE) ./data/
{%- elif cookiecutter.data_source_type == 'nfs' %}
	cp -r $(DATA_SOURCE)/* ./data/
{%- elif cookiecutter.data_source_type == 'url' %}
	wget $(DATA_SOURCE) -P ./data/
{% endif %}

init-docker: ## initialize docker image
	$(DOCKER) build -t $(IMAGE_NAME) --build-arg UID=$(shell id -u) .

init-docker-no-cache: ## initialize docker image without cachhe
	$(DOCKER) build --no-cache -t $(IMAGE_NAME) --build-arg UID=$(shell id -u) .

sync-to-source: ## sync local data to data source
{%- if cookiecutter.data_source_type == 's3' %}
	-aws s3 sync ./data/ $(DATA_SOURCE)
{%- elif cookiecutter.data_source_type == 'nfs' %}
	cp -r ./data/* $(DATA_SOURCE)/
{%- elif cookiecutter.data_source_type == 'url' %}
	echo "no sync target for url data source..."
{% endif %}

create-container: ## create docker container
	$(DOCKER) run -it -v $(PWD):/work -p $(JUPYTER_HOST_PORT):$(JUPYTER_CONTAINER_PORT) --name $(CONTAINER_NAME) $(IMAGE_NAME)

start-container: ## start docker container
	@echo "$$START_DOCKER_CONTAINER" | $(SHELL)
	@echo "Launched $(CONTAINER_NAME)..."
	$(DOCKER) attach $(CONTAINER_NAME)

jupyter: ## start Jupyter Notebook server
	jupyter-notebook --ip=0.0.0.0 --port=${JUPYTER_CONTAINER_PORT}

##@ DevTools

lint: ## check style with flake8
	flake8 {{ cookiecutter.project_slug }}

profile: ## show profile of the project
	@echo "CONTAINER_NAME: $(CONTAINER_NAME)"
	@echo "IMAGE_NAME: $(IMAGE_NAME)"
	@echo "JUPYTER_PORT: `$(DOCKER) port $(CONTAINER_NAME)`"
	@echo "DATA_SOURE: $(DATA_SOURCE)"

##@ Cleanup

clean: clean-pyc clean-docker ## remove all artifacts

clean-model: ## remove model artifacts
	rm -fr model/*

clean-pyc: ## remove Python file artifacts
	-find . -name '*.pyc' -exec rm -f {} +
	-find . -name '*.pyo' -exec rm -f {} +
	-find . -name '*~' -exec rm -f {} +
	-find . -name '__pycache__' -exec rm -fr {} +

distclean: clean clean-data clean-model ## remove all the reproducible resources including Docker images

clean-data: ## remove files under data
	rm -fr data/*

clean-docker: clean-container clean-image ## remove Docker image and container

clean-container: ## remove Docker container
	-$(DOCKER) rm $(CONTAINER_NAME)

clean-image: ## remove Docker image
	-$(DOCKER) image rm $(IMAGE_NAME)


###########################################################################################################
## ADD TARGETS SPECIFIC TO "{{ cookiecutter.project_name }}"
###########################################################################################################

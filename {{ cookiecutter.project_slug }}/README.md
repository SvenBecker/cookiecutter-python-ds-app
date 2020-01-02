# {{ cookiecutter.project_name }}

## Docker Version

### Prerequisities

{% if cookiecutter.use_nvidia_docker == "yes" %}
- [Docker-Nvidia](github.com/nvidia/docker-nvidia)
- Cuda Ready GPU https://nvidia.com/cuda
{% else %}
- [Docker](docker.io)
{% endif %}

### Install

```sh

```

## Non-Docker Version

### Prerequisities

- Python{{ cookiecutter.python_version }}
- Pipenv

### Install

```sh
make install
```

### Getting Started

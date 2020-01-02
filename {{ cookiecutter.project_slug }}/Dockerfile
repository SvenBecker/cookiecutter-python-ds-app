{%- if cookiecutter.use_nvidia_docker == 'yes' -%}
FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04 AS {{ cookiecutter.project_slug }}-release
{%- else -%}
FROM ubuntu:18.04 AS {{ cookiecutter.project_slug }}-release
{%- endif %}

RUN apt-get update && apt-get install -y \
  git \
  python{{ cookiecutter.python_version }} \
  python3-pip \
  python{{ cookiecutter.python_version }}-dev

RUN pip3 install --upgrade pip \
    pip3 install pipenv

COPY . /work

ARG UID
RUN useradd docker -l -u $UID -s /bin/bash -m
USER docker

WORKDIR /work

RUN pipenv install --ignore-pipfile --system --dev

ENTRYPOINT ["/bin/bash"]

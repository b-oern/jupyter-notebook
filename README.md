# Minimal Dockerfiles for Binder

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/b-oern/jupyter-notebook/master)

[Binder](https://mybinder.org) needs only one thing for images to work:

- to be able to launch `jupyter notebook` as a specified user (passed via docker build args as NB_UID/NB_USER)

What this means in practice is that the `notebook` package must be installed and on PATH:

```docker
RUN pip install --no-cache notebook
```

That's *almost* everything.

The remaining piece is that the specified user must be able to *start* the notebook,
which requires certain permissions like being able to write to the home directory.

The absolute bare minimum for this is to set HOME to `/tmp` so that it's writable,
which would make the shortest, smallest Dockerfile that works on Binder:

```docker
FROM python:3.7-slim
RUN pip install --no-cache notebook
ENV HOME=/tmp
```

However, it would be better to consume the NB_UID/NB_USER arguments and create a real user:

```docker
# create user with a home directory
ARG NB_USER
ARG NB_UID
ENV USER ${NB_USER}
ENV HOME /home/${NB_USER}

RUN adduser --disabled-password \
    --gecos "Default user" \
    --uid ${NB_UID} \
    ${NB_USER}
WORKDIR ${HOME}
```

From this point, you can start adding files, installing packages, etc.

Python-Pakete können über die [requirements.txt](https://github.com/binder-examples/requirements/blob/master/requirements.txt) von Binder installiert werden. Es werden verschiedene [Konfigurations-Dateien](https://mybinder.readthedocs.io/en/latest/config_files.html#config-files) z.B. apt.txt ünterstützt. Die Shell-Datei `postBuild` wird von binder ebenfalls ausgeführt. repo2docker: If a Dockerfile is present, all other configuration files will be ignored.

Wenn das Notebook `index.ipynb` existiert wird dieses Notebook von Jupyter geöffnet.

Die übergabe von Parametern über die URL ist auch möglich: https://github.com/manics/jupyter-urlparams

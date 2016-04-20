
1. Use `network_mode: "host"` in the docker-compose.yml for the hub service.

1. `export DOCKER_NETWORK_NAME=host`.

1. Use hacked version of dockerspawner in JupyterHub Dockerfile:

  ```
  RUN /opt/conda/bin/pip install \
      -e git+https://github.com/jupyter/jupyterhub@62a5e9dbce86cbb8992def81600ff9881d515935#egg=jupyterhub \
      -e git+https://github.com/jupyter/oauthenticator@011f6ea25c6bafca087d94a6c73d24dcbb0bf80e#egg=oauthenticator \
      -e git+https://github.com/jtyberg/dockerspawner@host-port#egg=dockerspawner
  ```

1. override the `/usr/local/bin/start-singleuser.sh` script to support setting port in spawned container:

  ```
  #!/bin/bash
  set -e

  notebook_arg=""
  if [ -n "${NOTEBOOK_DIR:+x}" ]
  then
      notebook_arg="--notebook-dir=${NOTEBOOK_DIR}"
  fi

  : "${JPY_PORT:=8888}"

  exec jupyterhub-singleuser \
    --port=$JPY_PORT \
    --ip=0.0.0.0 \
    --user=$JPY_USER \
    --cookie-name=$JPY_COOKIE_NAME \
    --base-url=$JPY_BASE_URL \
    --hub-prefix=$JPY_HUB_PREFIX \
    --hub-api-url=$JPY_HUB_API_URL \
    ${notebook_arg} \
    $@
  ```

1. Run JupyterHub with `docker-compose.yml` in this directory:

  ```
  ./hub.sh -f examples/host-network/docker-compose.yml up -d
  ```

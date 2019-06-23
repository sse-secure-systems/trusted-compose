# trusted-compose: Docker Content Trust for docker-compose

`trusted-compose` is a wrapper for `docker-compose`, adding some extra logistics to support Docker Content Trust (DCT, a.k.a. image signing) with `docker-compose`.

`docker-compose` does not have any support for image signing or image signature verification. To enable this functionality, `trusted-compose` wraps or replaces the relevant actions by helper functions. This affects the following commands:

- `trusted-compose build`
- `trusted-compose push`
- `trusted-compose pull`

`build` and `pull` are modified such that images are downloaded and their signatures verified using the standalone `docker` command (with DCT support enabled). Pulled images are then propagated to a local registry. Images stored in this registry are known to have passed the signature verification. `push` is overridden such that images are uploaded using `docker push` which ensures DCT handling along the way.

Subsequent operations (such as the actual building or running) are handled by vanilla `docker-compose`, but are based upon the images stored in the local registry. The local registry is expected to run at `localhost:5000`.

## Install

`trusted-compose` assumes the presence of a few Python packages. You can install them like this:

```bash
git clone git@github.com:sse-secure-systems/trusted-compose.git
cd trusted-compose/
python3.7 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Make sure to activate the Python virtual environment before using `trusted-compose`.

## How to run

### Prerequisites

First, make sure that all your Dockerfiles begin with lines like this:

```
ARG DOCKER_REGISTRY
FROM ${DOCKER_REGISTRY}debian:stable
```

instead of the usual `FROM debian:stable`. Further, the `docker-compose.yml` file needs to have all `image` directives prefixed with `${DOCKER_REGISTRY}`, e.g. `${DOCKER_REGISTRY}repository/image:tag`. This is required to enable `trusted-compose` to inject its logic.

As all variables default to empty string, these changes are backwards-compatible. It remains possible to use `docker-compose` (but no DCT capabilities, of course).

### Run

1. Fire up a local registry: `DOCKER_CONTENT_TRUST=1 docker run -d -p 5000:5000 --name registry registry:2`
2. Enable the Python virtual environment (see previous section)
3. Run `trusted-compose help` or use any other `trusted-compose` command

## Limitations

- The command line options that `docker-compose` offers for the `push` and `pull` command are not supported by `trusted-compose`.
- When using `trusted-compose`, all commands of `docker-compose` are considered reserved words. They are only allowed *once* on the command line. In other words, you cannot have these words as part of other arguments or flags. For example, trying to explicitly build a service whose name is one of the keywords causes an error: `trusted-compose build config` leads to the message "Multiple reserved words encountered: build, config".

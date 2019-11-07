# Concourse Docker-Compose-in-Docker

Optimized for use with [Concourse CI](http://concourse.ci/).

The image is Alpine based, and includes Docker, Docker Compose, and Docker Squash, as well as Bash, maven and openJDK11.

Image published to Docker Hub: [andibraeu/concourse-dcind-maven](https://hub.docker.com/r/andibraeu/concourse-dcind-maven/).

Inspired by [meAmidos/dcind](https://github.com/meAmidos/dcind),  [concourse/docker-image-resource](https://github.com/concourse/docker-image-resource/blob/master/assets/common.sh), [karkkfi/concourse-dcind](https://github.com/karlkfi/concourse-dcind), and [mesosphere/mesos-slave-dind](https://github.com/mesosphere/mesos-slave-dind).

## Features

Unlike meAmidos/dcind, this image...

- Does not require the user to manually start docker.
- Uses errexit, pipefail, and nounset.
- Configures timeout (`DOCKERD_TIMEOUT`) on dockerd start to account for mis-configuration (docker log will be output).
- Accepts arbitrary dockerd arguments via optional `DOCKER_OPTS` environment variable.
- Passes through `--garden-mtu` from the parent Gardian container if `--mtu` is not specified in `DOCKER_OPTS`.
- Sets `--data-root /scratch/docker` to bypass the graph filesystem if `--data-root` is not specified in `DOCKER_OPTS`.

## Build

```
docker build -t andibraeu/concourse-dcind-maven .
```

## Example

Here is an example of a Concourse [job](http://concourse.ci/concepts.html) that uses ```andibraeu/concourse-dcind-maven``` image to run a bunch of containers in a task, and then runs the integration test suite. You can find a full version of this example in the [```example```](example) directory.

```yaml
jobs:
- name: integration
  plan:
  - get: code
    params:
      depth: 1
    passed:
    - unit-tests
    trigger: true
  - task: integration-tests
    privileged: true
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: andibraeu/concourse-dcind-maven
      inputs:
      - name: code
      run:
        path: entrypoint.sh
        args:
        - bash
        - -ceux
        - |
          # build maven project that starts some docker containers during integration tests 
          mvn -f code/pom.xml clean very
```

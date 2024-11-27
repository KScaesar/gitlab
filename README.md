# gitlab

## configure `/etc/hosts`

```bash
echo "127.0.0.1 gitlab.example.com" | sudo tee -a /etc/hosts
```

## Docker-in-Docker

```
variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_TLS_VERIFY: 1
  DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"

job:
  services:
    - docker:27.3.1-dind
```

<https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-in-docker>

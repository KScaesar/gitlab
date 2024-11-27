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
  image: golang:1.23.2
  services:
    - docker:27.3.1-dind

  script:
    - |
      if [[ "$UT_TAGS" == "intg" ]]; then
        curl -fsSL https://get.docker.com -o get-docker.sh
        sh get-docker.sh /dev/null
      fi

    - go mod tidy -v -x
```

image 裡應該也要有 docker，因為是 call docker cli，接 service 的 docker engine

<https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-in-docker>

## Docker daemon in rootless mode

To run Docker as a non-privileged user, consider setting up the  Docker daemon in rootless mode for your user:  
    dockerd-rootless-setuptool.sh install  

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.

To run the Docker daemon as a fully privileged service, but granting non-root  
users access, refer to https://docs.docker.com/go/daemon-access/  


<https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script>

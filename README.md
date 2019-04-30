
## HOW-TO

wake up, gitlab!

```shell
docker-compose up -d

#proxy_ip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' traefik)
#sudo echo "${proxy_ip} gitlab.example.com traefik.example.com registry.example.com" | sudo tee -a /etc/hosts;

sudo echo "127.0.0.1 gitlab.example.com traefik.example.com registry.example.com" | sudo tee -a /etc/hosts;
```

- Log into gitlab.example.com


### Gitlab-runner
https://docs.gitlab.com/runner/configuration/advanced-configuration.html

- Get runner 'token api' in CI/CD settings

```shell
export GITLAB_RUNNER_TOKEN=BGfzQ2NGWEBwrymJKsNK # example token

docker exec gitlabrunner gitlab-runner register -n \
--url http://gitlab:80 \
--registration-token ${GITLAB_RUNNER_TOKEN} \
--clone-url http://gitlab:80 \
--executor docker \
--docker-image "docker:latest" \
--docker-privileged \
--docker-network-mode gitlab \
--docker-volumes ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
```

### Gitlab Registry

* Check gitlab-registry login

```shell
DOCKER_USERNAME=username # gitlab credentials
DOCKER_PASSWORD=password

docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD} registry.example.com
```

### Snippets

* gitlab runner example config

```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "a45488195e3c"
  url = "http://gitlab:80"
  token = "vjxcoQcQyawwZ4oJ9kkP"
  executor = "docker"
  clone_url = "http://gitlab:80"
  environment = ["DOCKER_DRIVER=overlay2"]
  [runners.custom_build_dir]
  [runners.docker]
    tls_verify = false
    image = "docker:latest"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    cache_dir = "cache"
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    network_mode = "gitlabcompose_gitlab"
    shm_size = 0
  [runners.cache]
    Insecure = false
    [runners.cache.s3]
    [runners.cache.gcs]
```

### Traefik example config

```toml
defaultEntryPoints = ["https","http"]

[entryPoints]
  [entryPoints.dashboard]
  address = ":8080"
#    [entryPoints.dashboard.auth]
#      [entryPoints.dashboard.auth.basic]
#        users = ["username:password"]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
  [entryPoints.https.tls]

[api]
entrypoint="dashboard"
dashboard = true
debug = true

[docker]
domain = "example.com"
watch = true
network = "gitlab"
exposedbydefault = false

[acme]
email = "notifications@example.com"
storage = "acme.json"
entryPoint = "https"
OnHostRule = true
  [acme.httpChallenge]
  entryPoint = "http"
```

### Related docs

https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose
https://docs.gitlab.com/ce/administration/container_registry.html#configure-container-registry-under-an-existing-gitlab-domain

https://docs.traefik.io/user-guide/docker-and-lets-encrypt/
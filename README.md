
## docs

https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose
https://github.com/sameersbn/docker-gitlab
https://medium.com/@rukeith/how-to-use-docker-to-build-a-self-host-gitlab-and-gitlab-runner-781981dc4d03


## gitlab runner

### register runner

- get token in CI/CD settings

docker exec -ti gitlab-runner /bash

gitlab-runner register -n --url http://gitlab:80 --registration-token _GITLAB_RUNNER_TOKEN_ --clone-url http://gitlab:80 --executor docker --docker-image "docker:latest" --docker-privileged
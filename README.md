
# About "nginx" Docker image
https://hub.docker.com/_/nginx/

## How to run it on Docker
Need to specify ports like `docker container run -d -p 81:80 --name <container_name> nginx:latest`
- We cannot access to nginx via Web Browser without this port configuration.
- p from:to
- Can check `docker port <container_name>` like `80/tcp -> 0.0.0.0:81`

## Commands
- Can use
  - cat
  - more
  - bash
- Cannot use
  - vi
  - less
  - ps

# Docker Image
https://hub.docker.com/_/nginx/
- nginx version : `1.15.6`
- Configuration file : `/etc/nginx/nginx.conf`
- OS : `debian`
  - We can use `apt-get`.
- Default port : `80`.


# Commands We Can Use in This Image
- sh
- bash
- pwd
- ls
- clear
- alias
- cat
- more
- hostname


# Commands We Can NOT Use in This Image
- Cannot use
- vi
- less
- ps
- lsof
- netstat
- ss
- nmap
- curl


# How to Run It with Docker
1. `docker container run -d -p <from-port>:<to-port> --name <container_name> nginx:latest`
2. Access to http://<host-ip>:<from-port>/index.html


# [Tips] How to Mange nginx
- `nginx` or `service nginx start` to start
- `nginx -s (stop|quit|reopen|reload)`

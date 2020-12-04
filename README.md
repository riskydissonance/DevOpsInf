# DevOps Inf

Docker compose file to host:

* Gitlab
* Jenkins
* Sonar
* Artifactory
* Nginx reverse proxy for all the above.
* Postgres for DB

Just running `docker-compose --build up` should start all the above infrastructure and allow them to communicate.


Volumes on WSL are mounted at: `/mnt/wsl/docker-desktop-mount-points`

## Config-as-code

* Config-as-code is a WIP, but some examples are provided.
* Example (redacted) config-as-code in jenkins/config
* Example Jenkinsfile that would be committed to Seatbelt repository in gitlab/examples/Seatbelt/Jenkinsfile

## Setup

### Sonar

#### Setting VM Max on host (e.g. WSL or Linux)
https://stackoverflow.com/questions/51445846/elasticsearch-max-virtual-memory-areas-vm-max-map-count-65530-is-too-low-inc

### Dev environment

Locally we're setting up infrastructure that doesn't have DNS entries nor a valid certificate, so there are a few hacks:
#### Nginx

##### Setting up a Self-signed cert

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx/cert/cert.key -out ./nginx/cert/cert.crt
```

https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04#:~:text=%20How%20To%20Create%20a%20Self-Signed%20SSL%20Certificate,made%20our%20changes%20and%20adjusted%20our...%20More%20

* Jenkins Dockerfile is set to turn of SSL verify for git
* Similar actions will have to be taken for Windows agents for Jenkins

#### DNS

In this example, set the DNS for the hosts set in the nginx.conf file.

E.g. For local access, in `/etc/hosts`:

```
127.0.0.1 gitlab.jmpesp.local
127.0.0.1 jenkins.jmpesp.local
127.0.0.1 artifactory.jmpesp.local
127.0.0.1 sonar.jmpesp.local
127.0.0.1 gitlab
127.0.0.1 jenkins
127.0.0.1 artifactory
127.0.0.1 sonar
```
## TODO

* Fix local dev hacks from having no cert/dns
    - internal & external IP resolving for local dev (hacked in compose with extra_hosts)
    - Jenkins docker image has ssl verify off
* Add config-as-code for all services where possible

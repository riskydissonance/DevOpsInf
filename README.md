# DevOps Inf

Docker compose file to host:

* Gitlab
* Jenkins
* Sonar
* Artifactory
* Nginx reverse proxy for all the above.
* Postgres for DB

Just running `docker compose --build up` should start all the above infrastructure and allow them to communicate.

Config-as-code is a WIP, but examples are provided.

Volumes on WSL are mounted at: `/mnt/wsl/docker-desktop-mount-points`
### Nginx

##### Setting up a Self-signed cert

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx/cert/cert.key -out ./nginx/cert/cert.crt
```

https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04#:~:text=%20How%20To%20Create%20a%20Self-Signed%20SSL%20Certificate,made%20our%20changes%20and%20adjusted%20our...%20More%20

### Sonar

##### Setting VM Max on host (e.g. WSL)
https://stackoverflow.com/questions/51445846/elasticsearch-max-virtual-memory-areas-vm-max-map-count-65530-is-too-low-inc

### DNS

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

### Jenkins

* Example (redacted) config-as-code in jenkins/config
* Example Jenkinsfile that would be committed to Seatbelt repository in gitlab/examples/Seatbelt/Jenkinsfile

## TODO

* Fix local dev hacks from having no cert/dns
    - internal & external IP resolving for local dev (hacked in compose with extra_hosts)
    - Jenkins docker image has ssl verify off
* Add config-as-code for all services where possible

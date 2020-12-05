# DevOps Inf

Docker compose file to host:

* Gitlab
* Jenkins
* Sonar
* Artifactory
* Nginx reverse proxy for all the above.
* Postgres for DB

Just running `docker-compose up` should start all the above infrastructure and allow them to communicate.

Volumes on WSL are mounted at: `/mnt/wsl/docker-desktop-mount-points`

## Creds

* Gitlab: `root`:`o1%5BcpHK%I@TQ8!mUxi6@$gPQ^4KV`
* Artifactory: `admin`: `3Cb0pAqbaqA*7%sXTUXWk*z5KTzesj`
* Jenkins: `admin` : `3kDCH%4ULu9BVlHp!2@&7jAZqRf2q!`
* Sonar: `admin` : `Kc4%%22cxsTAlhVibNrFj1I3Wpb4sE`

## Config-as-code

* Config-as-code is a WIP
* Example Jenkinsfiles for repos that would be committed to each repo are in gitlab/examples
* Artifactory: https://www.jfrog.com/confluence/display/JFROG/Artifactory+Configuration+Descriptors

## Setup

### Sonar

#### Setting VM Max on host (e.g. WSL or Linux)

`sysctl -w vm.max_map_count=262144`

https://stackoverflow.com/questions/51445846/elasticsearch-max-virtual-memory-areas-vm-max-map-count-65530-is-too-low-inc

### Dev environment

* In prod you're not going to use :latest images but a specified version until new versions have been tested (obviously)
* Locally we're setting up infrastructure that doesn't have DNS entries nor a valid certificate, so there are a few hacks:
#### Nginx

##### Setting up a Self-signed cert

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./nginx/cert/cert.key -out ./nginx/cert/cert.crt
```

https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04#:~:text=%20How%20To%20Create%20a%20Self-Signed%20SSL%20Certificate,made%20our%20changes%20and%20adjusted%20our...%20More%20

* Jenkins Dockerfile is set to turn of SSL verify for git
* Sonar scanner doesn't have the ability to accept the self signed cert so sonar is also exposed over http in nginx for the scanner - this wouldn't be necessary with a legitimate certificate.

#### DNS

In this example, set the DNS for the hosts set in the nginx.conf file.

E.g. For local access, in `/etc/hosts` set the hostnames to your host's Docker/WSL IP:

```
172.22.64.1 gitlab.jmpesp.local
172.22.64.1 jenkins.jmpesp.local
172.22.64.1 artifactory.jmpesp.local
172.22.64.1 sonar.jmpesp.local
172.22.64.1 gitlab
172.22.64.1 jenkins
172.22.64.1 artifactory
172.22.64.1 sonar
```

#### Jenkins

##### SSL

* Jenkins docker image has ssl verify off as SSL is self signed (in Dockerfile)

##### Windows slave

For a Windows slave all you need to do is set it up in Jenkins, download the jnlp file and ensure docker (for Sonar) & visual studio (for MSBuild) is installed on the windows host.

##### Webhooks for Git pushing

https://github.com/jenkinsci/gitlab-plugin/issues/375

## TODO

* Add config-as-code for all services where possible
* Play with Windows containers for MSBuild

# DevOps Inf

Example inf setup using Kubernetes and config-as-code to quickly host and use:

* Gitlab
* Jenkins
* Sonar
* Artifactory
* Nginx reverse proxy for all the above.
* Postgres for DB

Just running `kubectl apply -f . --recursive` should start all the above infrastructure and allow them to communicate.

Config-as-code is a WIP.
## Creds

Before anyone tries anything these are all unique to this particular infrastructure example setup, with nothing sensitive in the config etc.

* Gitlab: `root`:`o1%5BcpHK%I@TQ8!mUxi6@$gPQ^4KV`
* Artifactory: `admin`: `3Cb0pAqbaqA*7%sXTUXWk*z5KTzesj`
* Jenkins: `admin` : `3kDCH%4ULu9BVlHp!2@&7jAZqRf2q!`
* Sonar: `admin` : `Kc4%%22cxsTAlhVibNrFj1I3Wpb4sE`

## Config-as-code

* Config-as-code is a WIP
    * Working for Jenkins
        * https://www.jenkins.io/projects/jcasc
    * Working for Artifactory
        * https://www.jfrog.com/confluence/display/JFROG/Artifactory+Configuration+Descriptors
    * TODO for Sonar
    * TODO for Gitlab

* Example `Jenkinsfile`s for repos are in `gitlab/examples`.
* See https://github.com/m0rv4i/RT-Jenkins-Piplines-Common for the Jenkins shared library.

## Setup

### Dev environment

* In prod you're not going to use `:latest` images but a specified version until new versions have been tested (obviously)
* Locally we're setting up infrastructure that doesn't have DNS entries nor a valid certificate, so there are a few hacks:
#### Ingress Reverse Proxy

##### Setting up the Self-signed certs

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout gitlab.key -out gitlab.crt  -subj "/CN=gitlab.jmpesp.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout jenkins.key -out jenkins.crt  -subj "/CN=jenkins.jmpesp.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout artifactory.key -out artifactory.crt  -subj "/CN=artifactory.jmpesp.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout sonar.key -addext "subjectAltName = DNS:sonar.jmpesp.local" -out sonar.crt  -subj "/CN=sonar.jmpesp.local"
```
**NOTE: Sonar SSL checks fail if the DNS name isn't also an Alt name hence the different command above.**

The below will base64 all the files in the current dir and their names for easy creation of the tls yaml files.
```
find . -type f -exec echo {} \; -exec base64 -w 0 {}  \; -exec echo \;
```

https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04#:~:text=%20How%20To%20Create%20a%20Self-Signed%20SSL%20Certificate,made%20our%20changes%20and%20adjusted%20our...%20More%20

Base64 these and add them to the `k8s-*-tls.yaml` files.

Add them to the Host **Trusted Root CA** (double click the crt on Windows) and to the Java Keystore:

```
& "C:\Program Files\Java\jre1.8.0_271\bin\keytool.exe" -import -file sonar.crt -keystore "C:\Program Files\Java\jre1.8.0_271\lib\security\cacerts" -alias sonar-ingress -storepass changeit -noprompt
```

**NOTE BEST TO USE JAVA 8 AS JAVA 11+ FUCKS STUFF UP.**
* https://issues.jenkins.io/browse/JENKINS-61212

Note:
* Jenkins Dockerfile is set to turn off SSL verify for git
* Sonar scanner doesn't have the ability to accept the self signed cert so sonar is also exposed over http in nginx for the scanner - this wouldn't be necessary with a legitimate certificate.

#### DNS

Grab the Ingress external IP using `kubectl get ingress` (note on minikube this needs enabling first) and add that to your DNS/hosts file.

E.g. For local access, in `/etc/hosts` set the hostnames to your host's Docker/WSL IP:

```
172.22.64.1 gitlab.jmpesp.local
172.22.64.1 jenkins.jmpesp.local
172.22.64.1 artifactory.jmpesp.local
172.22.64.1 sonar.jmpesp.local
```

#### Jenkins

##### SSL

* Jenkins docker image has ssl verify off as SSL is self signed (in Dockerfile)

##### Windows agent

For a Windows agent all you need to do is set it up in Jenkins, download the jnlp file and ensure docker (for Sonar) & visual studio (for MSBuild) is installed on the windows host.

On Kubernetes you can use the `websockets` agent to avoid having to expose additional agent ports and route them. This is a checkbox
in the agent config.

##### Webhooks for Git pushing

https://github.com/jenkinsci/gitlab-plugin/issues/375

### Minikube Notes

* Minikube ideally needs `minikube start --cpus 4 --memory 12000`
    * E.g. `minikube start --cpus 4 --memory 12000 --driver=hyperv --mount-string .:/tmp/devopsinf --mount`
* Don't forget to enable ingress
    * `minikube addons enable ingress`
* Each time you start Minikube you may get a new ingress IP - need to set this in your hosts file
    * `kubectl get ingress`
* When building Docker images you need to point to the Minikube docker registry
    * `& minikube -p minikube docker-env | Invoke-Expression`
* LoadBalancer type isn't supported and just gets a NodePort - maybe can use MetalLB but it's in beta and a bit flaky
    * At present we use a fixed NodePort LoadBalancer for Ingress to expose Gitlab SSH on port 30000
    * SSH URLs and then e.g. `ssh://git@gitlab.jmpesp.local:30000/modules/SafetyDump.git`
* Can connect to minikube with `minikube ssh`
* On Windows hosts, can hit an error mounting directoryies:
    ```
    X mount failed: mount: /mount-dir: mount(2) system call failed: Connection timed out. : Process exited with status 32
    ```
    Check that minikube doesn't have any Windows Firewall inbound rules outright blocking connections. Update the rules for the **Public** profile even on **Private** networks, that's the one the fixes the issue. `¯\_(ツ)_/¯`

## TODO

* Add config-as-code for all services where possible
* Play with Windows containers for MSBuild on Server 2019

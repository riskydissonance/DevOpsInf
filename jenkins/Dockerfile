FROM jenkins/jenkins:latest
USER root
RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable"
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN git config --global http.sslVerify false
RUN jenkins-plugin-cli --plugins blueocean:latest configuration-as-code:latest \
       msbuild:latest gitlab-plugin:latest material-theme:latest sonar:latest \
       slack:latest artifactory:latest pipeline-model-definition \
       built-on-column:latest envinject:latest email-ext:latest timestamper:latest \
       build-timeout:latest git-parameter:latest

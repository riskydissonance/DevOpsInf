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
COPY *.crt $JAVA_HOME/jre/lib/security
RUN cd $JAVA_HOME/jre/lib/security \
    && keytool -keystore cacerts -storepass changeit -noprompt -trustcacerts -importcert -alias gitlabcert -file gitlab.crt \
    && keytool -keystore cacerts -storepass changeit -noprompt -trustcacerts -importcert -alias artifactorycert -file artifactory.crt \
    && keytool -keystore cacerts -storepass changeit -noprompt -trustcacerts -importcert -alias sonarcert -file sonar.crt
COPY *.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates
USER jenkins
RUN jenkins-plugin-cli --plugins blueocean:latest configuration-as-code:latest \
       msbuild:latest gitlab-plugin:latest material-theme:latest sonar:latest \
       slack:latest artifactory:latest pipeline-model-definition \
       built-on-column:latest timestamper:latest \
       build-timeout:latest git-parameter:latest ws-cleanup:latest

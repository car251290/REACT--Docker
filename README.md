# REACT--Docker  Whalecome 🐳 
React- Docker
This is a fully functional Jenkins server, based on the weekly and LTS releases.

logo

To use the latest LTS: docker pull jenkins/jenkins:lts-jdk11
To use the latest weekly: docker pull jenkins/jenkins:jdk11




## Docker React 
eliminates the need to manage host systems and instead lets us specify a Docker file that then lists all the dependencies of the project
<div>
  <img   style=width:20%;height:20%  src="https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2014/05/homepage-docker-logo.png?itok=zx0e-vcP"/>

</div>



Official Jenkins Docker image

Docker Stars Docker Pulls Join the chat at https://gitter.im/jenkinsci/docker

The Jenkins Continuous Integration and Delivery server is available on Docker Hub.

This is a fully functional Jenkins server. https://jenkins.io/.

Usage

NOTE: Avoid using a bind mount from a folder on the host machine into /var/jenkins_home, as this might result in file permission issues (the user used inside the container might not have rights to the folder on the host machine). If you really need to bind mount jenkins_home, ensure that the directory on the host is accessible by the jenkins user inside the container (jenkins user - uid 1000) or use -u some_other_user parameter with docker run.

docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins:lts-jdk11
This will run Jenkins in detached mode with port forwarding and volume added. You can access logs with command 'docker logs CONTAINER_ID' in order to check first login token. ID of container will be returned from output of command above.

Backing up data

If you bind mount in a volume - you can simply back up that directory (which is jenkins_home) at any time.

This is highly recommended. Treat the jenkins_home directory as you would a database - in Docker you would generally put a database on a volume.

If your volume is inside a container - you can use docker cp $ID:/var/jenkins_home command to extract the data, or other options to find where the volume data is. Note that some symlinks on some OSes may be converted to copies (this can confuse jenkins with lastStableBuild links, etc)

For more info check Docker docs section on Use volumes

In order to connect agents through an inbound TCP connection, map the port: -p 50000:50000. That port will be used when you connect agents to the controller.

If you are only using SSH (outbound) build agents, this port is not required, as connections are established from the controller. If you connect agents using web sockets (since Jenkins 2.217), the TCP agent port is not used either.

Passing JVM parameters

You might need to customize the JVM running Jenkins, typically to adjust system properties or tweak heap memory settings. Use the JAVA_OPTS or JENKINS_JAVA_OPTS environment variables for this purpose :

docker run --name myjenkins -p 8080:8080 -p 50000:50000 --restart=on-failure --env JAVA_OPTS=-Dhudson.footerURL=http://mycompany.com jenkins/jenkins:lts-jdk11
JVM options specifically for the Jenkins controller should be set through JENKINS_JAVA_OPTS, as other tools might also respond to the JAVA_OPTS environment variable.

Configuring logging

Jenkins logging can be configured through a properties file and java.util.logging.config.file Java property. For example:

mkdir data
cat > data/log.properties <<EOF
handlers=java.util.logging.ConsoleHandler
jenkins.level=FINEST
java.util.logging.ConsoleHandler.level=FINEST
EOF
docker run --name myjenkins -p 8080:8080 -p 50000:50000 --restart=on-failure --env JAVA_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties" -v `pwd`/data:/var/jenkins_home jenkins/jenkins:lts-jdk11
Configuring reverse proxy

If you want to install Jenkins behind a reverse proxy with a prefix, example: mysite.com/jenkins, you need to add environment variable JENKINS_OPTS="--prefix=/jenkins" and then follow the below procedures to configure your reverse proxy, which will depend if you have Apache or Nginx:

Apache
Nginx
Passing Jenkins launcher parameters

Arguments you pass to docker running the Jenkins image are passed to jenkins launcher, so for example you can run:

docker run jenkins/jenkins:lts-jdk11 --version
This will show the Jenkins version, the same as when you run Jenkins from an executable war.

You can also define Jenkins arguments via JENKINS_OPTS. This is useful for customizing arguments to the jenkins launcher in a derived Jenkins image. The following sample Dockerfile uses this option to force use of HTTPS with a certificate included in the image.

FROM jenkins/jenkins:lts-jdk11

COPY --chown=jenkins:jenkins certificate.pfx /var/lib/jenkins/certificate.pfx
COPY --chown=jenkins:jenkins https.key /var/lib/jenkins/pk
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8083 --httpsKeyStore=/var/lib/jenkins/certificate.pfx --httpsKeyStorePassword=Password12
EXPOSE 8083
You can also change the default agent port for Jenkins by defining JENKINS_SLAVE_AGENT_PORT in a sample Dockerfile.

FROM jenkins/jenkins:lts-jdk11
ENV JENKINS_SLAVE_AGENT_PORT 50001
or as a parameter to docker,

docker run --name myjenkins -p 8080:8080 -p 50001:50001 --restart=on-failure --env JENKINS_SLAVE_AGENT_PORT=50001 jenkins/jenkins:lts-jdk11
Note: This environment variable will be used to set the system property jenkins.model.Jenkins.slaveAgentPort.

If this property is already set in JAVA_OPTS or JENKINS_JAVA_OPTS, then the value of JENKINS_SLAVE_AGENT_PORT will be ignored.
Installing more tools

You can run your container as root - and install via apt-get, install as part of build steps via jenkins tool installers, or you can create your own Dockerfile to customise, for example:

FROM jenkins/jenkins:lts-jdk11
# if we want to install via apt
USER root
RUN apt-get update && apt-get install -y ruby make more-thing-here
# drop back to the regular jenkins user - good practice
USER jenkins
In such a derived image, you can customize your jenkins instance with hook scripts or additional plugins. For this purpose, use /usr/share/jenkins/ref as a place to define the default JENKINS_HOME content you wish the target installation to look like :

FROM jenkins/jenkins:lts-jdk11
COPY --chown=jenkins:jenkins custom.groovy /usr/share/jenkins/ref/init.groovy.d/custom.groovy
Preinstalling plugins

Install plugins script

You can rely on the plugin manager CLI to pass a set of plugins to download with their dependencies. This tool will perform downloads from update centers, and internet access is required for the default update centers.

Setting update centers

During the download, the script will use update centers defined by the following environment variables:

JENKINS_UC - Main update center. This update center may offer plugin versions depending on the Jenkins LTS Core versions. Default value: https://updates.jenkins.io
JENKINS_UC_EXPERIMENTAL - Experimental Update Center. This center offers Alpha and Beta versions of plugins. Default value: https://updates.jenkins.io/experimental
JENKINS_INCREMENTALS_REPO_MIRROR - Defines Maven mirror to be used to download plugins from the Incrementals repo. Default value: https://repo.jenkins-ci.org/incrementals
JENKINS_UC_DOWNLOAD - Download url of the Update Center. Default value: $JENKINS_UC/download
It is possible to override the environment variables in images.

Installing Custom Plugins

Installing prebuilt, custom plugins can be accomplished by copying the plugin HPI file into /usr/share/jenkins/ref/plugins/ within the Dockerfile:

COPY --chown=jenkins:jenkins path/to/custom.hpi /usr/share/jenkins/ref/plugins/
Usage

You can run the CLI manually in Dockerfile:

FROM jenkins/jenkins:lts-jdk11
RUN jenkins-plugin-cli --plugins pipeline-model-definition github-branch-source:1.8
Furthermore, it is possible to pass a file that contains this set of plugins (with or without line breaks).

FROM jenkins/jenkins:lts-jdk11
COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli -f /usr/share/jenkins/ref/plugins.txt
When jenkins container starts, it will check JENKINS_HOME has this reference content, and copy them there if required. It will not override such files, so if you upgraded some plugins from UI they won't be reverted on next start.

In case you do want to override, append '.override' to the name of the reference file. E.g. a file named /usr/share/jenkins/ref/config.xml.override will overwrite an existing config.xml file in JENKINS_HOME.

Also see JENKINS-24986

Here is an example to get the list of plugins from an existing server:

JENKINS_HOST=username:password@myhost.com:port
curl -sSL "http://$JENKINS_HOST/pluginManager/api/xml?depth=1&xpath=/*/*/shortName|/*/*/version&wrapper=plugins" | perl -pe 's/.*?<shortName>([\w-]+).*?<version>([^<]+)()(<\/\w+>)+/\1 \2\n/g'|sed 's/ /:/'
Example Output:
Updating plugins file
The plugin-installation-manager-tool supports updating the plugin file for you.







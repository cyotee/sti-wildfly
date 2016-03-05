# This image provides a base for building and running WildFly applications.
# It builds using maven and runs the resulting artifacts on WildFly 8.1.

FROM openshift/base-centos7

MAINTAINER Ben Parees <bparees@redhat.com>

EXPOSE 8080

ENV WILDFLY_VERSION 8.1
ENV WILDFLY_VERSION_BIN 8.1.0.Final
ENV MAVEN_VERSION 3.3.3

LABEL io.k8s.description="Platform for building and running JEE applications on WildFly 8.1" \
      io.k8s.display-name="WildFly 8.1" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,wildfly,wildfly81" \
      io.openshift.s2i.destination="/opt/s2i/destination"

ENV JBOSS_HOME=/wildfly

# Install Maven, Wildfly 8
RUN INSTALL_PKGS="tar unzip bc which lsof java-1.8.0-openjdk java-1.8.0-openjdk-devel" && \
    yum install -y --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    (curl -0 http://www.us.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | \
    tar -zx -C /usr/local) && \
    ln -sf /usr/local/apache-maven-$MAVEN_VERSION/bin/mvn /usr/local/bin/mvn && \
    mkdir -p /wildfly && \
    (curl -0 http://download.jboss.org/wildfly/$WILDFLY_VERSION_BIN/wildfly-$WILDFLY_VERSION_BIN.tar.gz | tar -zx --strip-components=1 -C $JBOSS_HOME) && \
    mkdir -p /opt/s2i/destination

# Add sti wildfly customizations
ADD ./contrib/wfmodules/ $JBOSS_HOME/modules/
ADD ./contrib/wfbin/standalone.conf $JBOSS_HOME/bin/standalone.conf
ADD ./contrib/wfcfg/standalone.xml $JBOSS_HOME/standalone/configuration/standalone.xml

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

RUN chown -R 1001:0 $JBOSS_HOME && \
    chown -R 1001:0 $JBOSS_HOME/.pki && \
    chmod -R ug+rw $JBOSS_HOME && \
    chmod -R g+rw /opt/s2i/destination && \


ADD installs/ /opt/

ENV JAVA_AGENT_HOME=/opt/appdynamics/java_server_agent/

ENV JBOSS_MODULES_SYSTEM_PKGS="org.jboss.byteman,com.singularity,org.jboss.logmanager" \
    JAVA_OPTS_EXT="-javaagent:${JAVA_AGENT_HOME}/javaagent.jar -Xbootclasspath/p:$(ls ${JBOSS_HOME}/modules/system/layers/base/org/jboss/logmanager/main/jboss-logmanager-*.jar)" \
    APPDYNAMICS_AGENT_APPLICATION_NAME=loadtest \
    APPDYNAMICS_AGENT_TIER_NAME=test \
    APPDYNAMICS_AGENT_NODE_NAME="loadtest-docker"

ENV APPD_LOGS="$JAVA_AGENT_HOME/ver4.1.7.1/logs/$APPDYNAMICS_AGENT_NODE_NAME"

RUN mkdir -p $APPD_LOGS

RUN chown -R 1001:0 $APPD_LOGS && \
    chmod -R ug+rw $APPD_LOGS && \
    chmod -R g+rw $APPD_LOGS

USER 1001

CMD $STI_SCRIPTS_PATH/usage

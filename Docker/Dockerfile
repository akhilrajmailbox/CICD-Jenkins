FROM openjdk:8-jdk
USER root
RUN apt-get update \
    && apt-get install -y sshpass git curl wget gettext jq zip unzip tar \
    software-properties-common python3-pip openssh-server openssh-client ldap-utils rsync
RUN mkdir /run/sshd
RUN pip3 install jenkins-job-builder jmespath ansible
WORKDIR /tmp
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install \
    && rm -rf aws awscliv2.zip
WORKDIR /var/jenkins_agent
ENTRYPOINT [ "/usr/sbin/sshd", "-D" ]
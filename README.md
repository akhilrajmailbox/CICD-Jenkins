# Jenkins Server

## Prerequisites

* We are using `devops-network` docker network for all devops deployment.
* You need at least Docker Engine 17.06.0  and docker-compose 1.18 for this to work.

```bash
docker --version
docker-compose --version
```

## Create Docker image from Dockerfile (optional, docker-compose will create docker image for you)

```bash
docker build -t akhilrajmailbox/jenkins:agent . -f Dockerfile
```

## Create custom Docker network before deploying container (If the network is already created then ignore it)

*Note : Make sure that the subnet won't conflict with any other subnets*

```bash
docker network create --driver=bridge --subnet=172.31.0.0/16 devops-network
```

## Deploy Jenkins as Docker container

```bash
docker-compose -f docker-compose.yml up -d
```

## Deployment services

* jenkins-master

    * running as jenkins master to manage all jobs (must make executor 0 to ensure that non of the jobs are execute on this master nodes)

* jenkins-agent

    * give label `MASTER` and all the jobs that are going to triggered on master node will execute in this docker sidecar (agent node)


## Working with Jenkins


* Login to jenkins server

```bash
docker exec -it jenkins /bin/bash
```

* Restarting Jenkins

```bash
docker-compose -f docker-compose.yml restart
```

* stopping/starting jenkins

```
docker-compose -f docker-compose.yml stop/start
```

* check the logs of jenkins server

```bash
docker logs -f jenkins
```

### Jenkins side car ssh configuration (root user)

```shell
ssh-keygen
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

*Note : add this private key in jenkins credentials*


### Jenkins slave on Remote machine - ssh configuration (root user)

```shell
# On Remote VM
mkdir /var/lib/jenkins-slave # copy the compose file docker-slave-compose.yml
docker network create --driver=bridge --subnet=172.31.0.0/16 devops-network
# Within slave container
ssh-keygen
> ~/.ssh/id_rsa.pub
> ~/.ssh/id_rsa
vim ~/.ssh/id_rsa.pub
vim ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/*
```

*Note : add this private key in jenkins credentials*


## LDAP Auth Configuration on Jenkins

* Configure Jenkins with [Role-Based Strategy](https://plugins.jenkins.io/role-strategy/) and configure the roles as follows..

* Pattern Example

    * `SOME_JOB-SSO(/.*)?` : All jobs under `SOME_JOB-SSO` folder

    * `SOME_JOB.*(/.*)?` : All jobs starting with `SOME_JOB`

    * `SOME_JOB.*(/.*)?|DevOps(/.*)?` : All jobs starting with `SOME_JOB` and All jobs under `DevOps` folder



Configuration | Value
-------------|-------------
Server | `ldap://ldap-server`
root DN | `dc=my,dc=devops,dc=example,dc=com`
User search base | `ou=People`
User search filter | `uid={0}`
Group search base | `ou=Groups`
Group search filter | `(&(cn={0})(objectclass=posixGroup))`
Group membership filter | `(memberUid={1})`
Manager DN | `cn=Query Admin,ou=DevOps,ou=People,dc=my,dc=devops,dc=example,dc=com`
Manager Password | `SecurePassword`
Display Name LDAP attribute | `cn`
Email Address LDAP attribute | `mail`

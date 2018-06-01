https://hub.docker.com/r/bitnami/phabricator/
https://hub.docker.com/r/bitnami/jenkins/

sudo docker-compose up -d

Watch the Phabricator container log and wait a couple minutes until the
overseer deamon starts.

logins:
  - phabricator, epc:epcpower
  - jenkins, epc:epcpower

Note that the image names are prefixed by the project name.  This is derived from the
directory containing the docker-compose.yml file unless overridden on the cli.

---- Phabricator:
create jenkins bot user

create repository
git
name: grid-tied
callsign: GRIDTIED
set existing uri's to io type read only
create a new uri to git@github.com:epcpower/grid-tied with io type observe
set credential, add new, username git, private key from some github user with access
activate repository

create a build plan: Build Commit
add a build step
make http request
uri: http://jenkins:8080/jenkins/job/${repository.callsign}/buildWithParameters?DIFF_ID=${buildable.diff}&PHID=${target.phid}&COMMIT=${buildable.commit}
http method: get
add credential: name phabricator@jenkins, phabricator:epcpower (to be added to jenkins later)
when complete: wait for message

create herald rule
commits
global
rule name: Build Commit
when > repository > is any of > grid-tied (etc)
action: run build plans (and select Build Commit)

---- Jenkins:

install locally built phabricator plugin

install "Docker" plugin
manage > configure > add new cloud > docker
name: Host Docker
host uri: unix://var/run/docker.sock
enabled: check it
go ahead and test the connection

new item
GRIDTIED (to match phabricator repo callsign)
pipeline
ok
pipeline script from scm
scm git
ssh://git@phabricator/diffusion/GRIDTIED/grid-tied.git
additional behaviours > add > advanced sub-modules
use credentials from default remote of parent repository


add ssh user name with private key credentials
generate a keypair in phabricator for the 'jenkins' bot user
username: git
enter directly and paste the private key

branch specifier: ${COMMIT}
lightweight checkout: uncheck

advanced
name: grid-tied (to match phabricator repository name)

this project is parametrized: check
add
  string: DIFF_ID
  string: PHID
  string: COMMIT

save

add user: phabricator

jenkins > manage > configure

default phabricator credentials
add
phabricator conduit key
url: https://phabricator
description: jenkins@phabricator
conduit token: generate for jenkins@phabricator user

phabricator notifications
select the https://phabricator credentials

---- test it:

find a commit and run the build plan manually

----

https://secure.phabricator.com/book/phabricator/article/diffusion_hosting/#sshd-setup

cd /opt/bitnami/phabricator
bin/config set diffusion.ssh-port 22
bin/config set diffusion.ssh-user phab
sed -e 's;/path/to/phabricator;/opt/bitnami/phabricator;' -e 's/vcs-user/phab/' resources/sshd/phabricator-ssh-hook.sh > /opt/phabricator-ssh-hook.sh
sed -e 's/2222/22/' -e 's/vcs-user/phab/' -e 's;/usr/libexec/;/opt/;' resources/sshd/sshd_config.phabricator.example > /etc/ssh/sshd_config.phabricator
chmod 755 /opt/phabricator-ssh-hook.sh
apt update
apt install -y openssh-server
sed -i 's;SSHD_OPTS=$;SSHD_OPTS="-f /etc/ssh/sshd_config.phabricator";' /etc/default/ssh
service ssh start


cd /opt/bitnami/phabricator
bin/config set diffusion.ssh-port 22
bin/config set diffusion.ssh-user git

# echo 'export PATH=/opt/bitnami/php/bin:$PATH' >> /etc/profile

# sed -e 's;/path/to/phabricator;/opt/bitnami/phabricator;' -e 's/vcs-user/git/' resources/sshd/phabricator-ssh-hook.sh > /opt/phabricator-ssh-hook-actual.sh
# chmod 755 /opt/phabricator-ssh-hook-actual.sh
sed -e 's;/path/to/phabricator;/opt/bitnami/phabricator;' -e 's/vcs-user/git/' resources/sshd/phabricator-ssh-hook.sh > /opt/phabricator-ssh-hook.sh
chmod 755 /opt/phabricator-ssh-hook.sh
# echo '#!/bin/bash' > /opt/phabricator-ssh-hook.sh
# echo 'id >> /git/sshlog2' >> /opt/phabricator-ssh-hook.sh
# echo 'export PATH=/opt/bitnami/php/bin:$PATH' >> /opt/phabricator-ssh-hook.sh
# echo '/opt/phabricator-ssh-hook-actual.sh "$@" 2>&1 | tee -a /git/sshlog2' >> /opt/phabricator-ssh-hook.sh
# echo 'exit' >> /opt/phabricator-ssh-hook.sh
# chmod 755 /opt/phabricator-ssh-hook.sh
sed -e 's/2222/22/' -e 's/vcs-user/git/' -e 's;/usr/libexec/;/opt/;' resources/sshd/sshd_config.phabricator.example > /etc/ssh/sshd_config.phabricator
echo 'PermitUserEnvironment yes' >> /etc/ssh/sshd_config.phabricator
apt update
apt install -y openssh-server
sed -i 's;^\(git:.*\):/home/phabricator:;\1:/opt/bitnami/phabricator:;' /etc/passwd
sed -i 's/git:!:/git:*:/' /etc/shadow
# sed -i 's;SSHD_OPTS=.*;SSHD_OPTS="-f /etc/ssh/sshd_config.phabricator -ddd -E /sshlog";' /etc/default/ssh
sed -i 's;SSHD_OPTS=.*;SSHD_OPTS="-f /etc/ssh/sshd_config.phabricator";' /etc/default/ssh
# echo 'export PATH=/opt/bitnami/php/bin:$PATH' >> /etc/bash.bashrc
# sed -i 's;#!.*;#!/opt/bitnami/php/bin/php;' /opt/bitnami/phabricator/bin/ssh-auth
sed -i 's;\(\$root = \)dirname;\1;' /opt/bitnami/phabricator/bin/ssh-auth
mkdir -p ~git/.ssh
echo 'PATH=/usr/bin:/opt/bitnami/php/bin' >> ~git/.ssh/environment
chown git:git ~git/.ssh/environment

service ssh start


----- phabricator

apt update

apt install -y openssh-server

cd /opt/bitnami/phabricator
bin/config set diffusion.ssh-port 22
bin/config set diffusion.ssh-user git
sed -e 's;/path/to/phabricator;/opt/bitnami/phabricator;' -e 's/vcs-user/git/' resources/sshd/phabricator-ssh-hook.sh > /opt/phabricator-ssh-hook.sh
chmod 755 /opt/phabricator-ssh-hook.sh
sed -e 's/2222/22/' -e 's/vcs-user/git/' -e 's;/usr/libexec/;/opt/;' resources/sshd/sshd_config.phabricator.example > /etc/ssh/sshd_config.phabricator
echo 'PermitUserEnvironment yes' >> /etc/ssh/sshd_config.phabricator
sed -i 's;^\(git:.*\):/home/phabricator:;\1:/opt/bitnami/phabricator:;' /etc/passwd
sed -i 's/git:!:/git:*:/' /etc/shadow
sed -i 's;SSHD_OPTS=.*;SSHD_OPTS="-f /etc/ssh/sshd_config.phabricator";' /etc/default/ssh
sed -i 's;\(\$root = \)dirname;\1;' /opt/bitnami/phabricator/bin/ssh-auth
mkdir -p ~git/.ssh
echo 'PATH=/usr/bin:/opt/bitnami/php/bin' >> ~git/.ssh/environment
chown git:git ~git/.ssh/environment
service ssh start

# ----- jenkins
# 
# apt update
# 
# apt install -y docker


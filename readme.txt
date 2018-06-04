https://hub.docker.com/r/bitnami/phabricator/
https://hub.docker.com/r/bitnami/jenkins/

sudo docker-compose up -d

Watch the Phabricator container log and wait a couple minutes until the
overseer deamon starts.

logins:
  - phabricator.local, epc:epcpower
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
ssh://git@phabricator.local/diffusion/GRIDTIED/grid-tied.git
additional behaviours > add
  - wipeout repository and force clone
  - advanced submodule
    - check recursively update
    - check use credentials from parent

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

disable csrf protection (yeah, this is bad but i haven't figured it out yet)

default phabricator credentials
add
phabricator conduit key
url: http://phabricator.local (should be https, but no ssl setup here yet)
description: jenkins@phabricator.local
conduit token: generate for jenkins@phabricator.local user

phabricator notifications
select the https://phabricator.local credentials

---- test it:

find a commit and run the build plan manually

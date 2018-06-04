FROM bitnami/phabricator:2018.20.0

RUN install_packages openssh-server acl

# acl for: https://github.com/bitnami/bitnami-docker-phabricator/issues/77#issuecomment-331167933

WORKDIR /opt/bitnami/phabricator

COPY packages/phabricatormod /tmp/bitnami/pkg/cache/phabricatormod-linux-x64-debian-8

#RUN adduser --quiet --home /opt/bitnami/phabricator git

RUN bin/config set diffusion.ssh-port 22
RUN bin/config set diffusion.ssh-user git
RUN sed -e 's;/path/to/phabricator;/opt/bitnami/phabricator;' -e 's/vcs-user/git/' resources/sshd/phabricator-ssh-hook.sh > /opt/phabricator-ssh-hook.sh
RUN chmod 755 /opt/phabricator-ssh-hook.sh
RUN sed -e 's/2222/22/' -e 's/vcs-user/git/' -e 's;/usr/libexec/;/opt/;' resources/sshd/sshd_config.phabricator.example > /etc/ssh/sshd_config.phabricator
RUN echo 'PermitUserEnvironment yes' >> /etc/ssh/sshd_config.phabricator
RUN sed -i 's/git:!:/git:*:/' /etc/shadow
RUN sed -i 's;SSHD_OPTS=.*;SSHD_OPTS="-f /etc/ssh/sshd_config.phabricator";' /etc/default/ssh
RUN sed -i 's;\(\$root = \)dirname;\1;' /opt/bitnami/phabricator/bin/ssh-auth
RUN mkdir -p /home/phabricator/.ssh
RUN echo 'PATH=/usr/bin:/opt/bitnami/php/bin' >> /home/phabricator/.ssh/environment

RUN sed -i 's;\(exec .*\);usermod --password \\* git\nusermod --unlock git\nchown git:git ~git/.ssh/environment\nservice ssh start\nsetfacl -Rm d:u:phabricator:rwX,u:phabricator:rwX /bitnami/phabricator/data/\n\1;' /app-entrypoint.sh

EXPOSE 22

ENV PHABRICATOR_HOST="phabricator.local" \
    PHABRICATOR_USERNAME="epc" \
    PHABRICATOR_PASSWORD="epcpower"

WORKDIR /

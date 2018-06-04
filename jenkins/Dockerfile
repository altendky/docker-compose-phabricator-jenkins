FROM bitnami/jenkins:2.107.3

RUN sed -i 's;\(exec .*\);chown tomcat:tomcat /var/run/docker.sock\nchmod a+rw /var/run/docker.sock\n\1;' /app-entrypoint.sh

RUN install_packages git php5-cli

# this doesn't seem to help so...  maybe don't need it
WORKDIR /opt
RUN git clone https://github.com/phacility/libphutil.git
RUN git clone https://github.com/phacility/arcanist.git
ENV PATH="${PATH}:/opt/arcanist/bin"

WORKDIR /

RUN curl -fsSL get.docker.com -o get-docker.sh
RUN sh get-docker.sh

ENV JENKINS_USERNAME="user" \
    JENKINS_PASSWORD="password"

WORKDIR /

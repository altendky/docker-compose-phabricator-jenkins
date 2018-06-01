FROM bitnami/jenkins:2.107.3

RUN sed -i 's;\(exec .*\);chown tomcat:tomcat /var/run/docker.sock\nchmod a+rw /var/run/docker.sock\n\1;' /app-entrypoint.sh

#RUN apt-get update
#RUN apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
#RUN apt-get install -y lsb-release
#RUN curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
#RUN add-apt-repository \
#   "deb [arch=amd64] https://download.docker.com/linux/debian \
#   $(lsb_release -cs) \
#   stable"
#RUN apt-get update
#RUN apt-get install -y docker-ce

#RUN curl https://get.docker.com/ > install_docker.sh
#RUN chmod +x install_docker.sh
#RUN ./install_docker.sh

RUN curl -fsSL get.docker.com -o get-docker.sh
RUN sh get-docker.sh

ENV JENKINS_USERNAME="epc" \
    JENKINS_PASSWORD="epcpower"

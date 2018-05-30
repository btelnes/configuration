# To build this Dockerfile:
#
# From the root of configuration:
#
# docker build -f docker/build/edxapp/Dockerfile .
#
# This allows the dockerfile to update /edx/app/edx_ansible/edx_ansible
# with the currently checked-out configuration repo.

ARG BASE_IMAGE_TAG=latest
FROM edxops/xenial-common:${BASE_IMAGE_TAG}
MAINTAINER edxops
USER root
CMD ["/edx/app/supervisor/venvs/supervisor/bin/supervisord", "-n", "--configuration", "/edx/app/supervisor/supervisord.conf"]

ADD . /edx/app/edx_ansible/edx_ansible
WORKDIR /edx/app/edx_ansible/edx_ansible/docker/plays

COPY docker/build/edxapp/ansible_overrides.yml /
COPY docker/build/edxapp/devstack.yml /
COPY docker/build/devstack/ansible_overrides.yml /devstack/ansible_overrides.yml

ARG OPENEDX_RELEASE=master
ENV OPENEDX_RELEASE=${OPENEDX_RELEASE}
RUN sudo /edx/app/edx_ansible/venvs/edx_ansible/bin/ansible-playbook edxapp.yml \
    -c local -i '127.0.0.1,' \
    -t 'install,assets,devstack' \
    --extra-vars="edx_platform_version=${OPENEDX_RELEASE}" \
    --extra-vars="@/ansible_overrides.yml" \
    --extra-vars="@/devstack.yml" \
    --extra-vars="@/devstack/ansible_overrides.yml"

# Add mongo
WORKDIR /edx/app/edx_ansible/edx_ansible/docker/plays
RUN /edx/app/edx_ansible/venvs/edx_ansible/bin/ansible-playbook mongo.yml \
    -i '127.0.0.1,' -c local \
    -t 'install' \
    -e@/ansible_overrides.yml

# Add sshd
RUN apt-get update
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd

RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN mkdir /root/.ssh
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

EXPOSE 18000 18010 27017 22

CMD ["/usr/sbin/sshd", "-D"]

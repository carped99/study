FROM ubuntu:22.04

# install sshd
RUN apt update && apt install -y openssh-server vim; \
    apt clean;

RUN mkdir /run/sshd; \
    sed -i 's/^#\(PermitRootLogin\) .*/\1 yes/' /etc/ssh/sshd_config; \
    sed -i 's/^\(UsePAM yes\)/# \1/' /etc/ssh/sshd_config;

RUN echo 'root:root' | chpasswd

EXPOSE 22

CMD    ["/usr/sbin/sshd", "-D", "-e"]

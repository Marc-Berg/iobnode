FROM node:22-bookworm-slim

RUN apt-get update && apt-get install -y \
    openssh-server gcc g++ make git vim curl sudo libcap2-bin \
    && rm -rf /var/lib/apt/lists/*

RUN npm install --global @iobroker/dev-server

RUN useradd -ms /bin/bash iobdev

RUN mkdir /var/run/sshd

EXPOSE 22
EXPOSE 8081

COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

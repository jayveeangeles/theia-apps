ARG NODE_VERSION=12.18.3

FROM node:${NODE_VERSION}
RUN apt-get update && apt-get install -y libsecret-1-dev
ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean

FROM openvino/ubuntu20_dev:2021.4

USER root

RUN sh -c "curl -fsSL https://deb.nodesource.com/setup_12.x | bash " && \
    apt update && \
    apt-get install -y nodejs libsecret-1-0 yarn sudo \
    && apt-get clean \
    && apt-get auto-remove -y \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

RUN echo "openvino ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers

RUN mkdir -p /home/theia \
    && mkdir -p /home/project 
ENV HOME /home/theia
WORKDIR /home/theia
COPY --from=0 /home/theia /home/theia

RUN chown -R 1000:0 /home/theia /home/project

EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins
ENV USE_LOCAL_GIT true
ENV INTEL_OPENVINO_DIR /opt/intel/openvino

RUN echo "source ${INTEL_OPENVINO_DIR}/bin/setupvars.sh" > /home/theia/.bashrc

ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
USER 1000

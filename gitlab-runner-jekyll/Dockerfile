# Updated from https://gitlab.com/gitlab-org/gitlab-runner/-/tree/main/dockerfiles/runner/ubuntu?ref_type=heads
ARG BASE_IMAGE=ubuntu:24.04
FROM $BASE_IMAGE AS base
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        curl \
        git git-lfs \
        wget \
        tzdata \
        openssh-client \
    && rm -rf /var/lib/apt/lists/*

ARG DOCKER_MACHINE_VERSION
ARG DUMB_INIT_VERSION=1.2.5

RUN wget -O /tmp/gitlab-runner_amd64.deb https://gitlab-runner-downloads.s3.amazonaws.com/v17.4.0/deb/gitlab-runner_amd64.deb
COPY --chmod=755 install-deps install-gitlab-runner /tmp/
RUN /tmp/install-deps "${TARGETPLATFORM}" "${DOCKER_MACHINE_VERSION}" "${DUMB_INIT_VERSION}"
RUN rm -rf /tmp/* /etc/gitlab-runner/.runner_system_id

FROM $BASE_IMAGE AS gitlab-runner

COPY --from=base / /
COPY --chmod=755 entrypoint /

STOPSIGNAL SIGQUIT
VOLUME ["/etc/gitlab-runner", "/home/gitlab-runner"]
ENTRYPOINT ["/usr/bin/dumb-init", "/entrypoint"]
CMD ["run", "--user=gitlab-runner", "--working-directory=/home/gitlab-runner"]

### CUSTOM IMAGE with RUBY

ARG RUBY_INSTALLER_VERSION=0.9.3
ARG RUBY_VERSION=3.2.2

FROM gitlab-runner

RUN echo "Installing needed dependencies" \
  && apt-get update && apt-get install -y --no-install-recommends \
  locales \
  imagemagick \
  build-essential \
  zlib1g-dev \
  jupyter-nbconvert \
  npm \
  python3-pip && \
  apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

RUN echo "Fixing locales" \
  && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && locale-gen

ENV LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV JEKYLL_ENV=production

ARG RUBY_INSTALLER_VERSION
ARG RUBY_VERSION
# https://github.com/postmodern/ruby-install/releases/download/v$RUBY_INSTALLER_VERSION/ruby-install-$RUBY_INSTALLER_VERSION.tar.gz
RUN wget -O ruby-install.tar.gz "https://github.com/postmodern/ruby-install/releases/download/v${RUBY_INSTALLER_VERSION}/ruby-install-${RUBY_INSTALLER_VERSION}.tar.gz" && \
    tar -xzf ruby-install.tar.gz && \
    cd "ruby-install-${RUBY_INSTALLER_VERSION}/" && \
    apt-get update && \
    make install && \
    ruby-install --system ruby "${RUBY_VERSION}" && \
    cd .. && \
    rm -rf ruby-install* && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

RUN pip3 install --upgrade jupyter
RUN npm install -g purgecss

# Updated from https://gitlab.com/gitlab-org/gitlab-runner/-/tree/main/dockerfiles/runner/ubuntu?ref_type=heads
FROM ubuntu:24.04 AS gitlab-runner

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

ARG DUMB_INIT_VERSION=1.2.5
ARG GITLAB_RUNNER_VERSION=17.4.0

RUN wget -O /tmp/gitlab-runner.deb https://gitlab-runner-downloads.s3.amazonaws.com/v${GITLAB_RUNNER_VERSION}/deb/gitlab-runner_amd64.deb && \
    dpkg -i /tmp/gitlab-runner.deb && \ 
    apt-get update && \
    apt-get -f install -y && \
    rm -rf /var/lib/apt/lists/* && \
    gitlab-runner --version && \
    mkdir -p /etc/gitlab-runner/certs && \
    chmod -R 700 /etc/gitlab-runner && \
    curl -L "https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_x86_64" -o /tmp/dumb-init && \
    install -m 555 -o 0 -g 0 /tmp/dumb-init /usr/bin/dumb-init && \
    dumb-init --version && \
    rm -rf /tmp/* /etc/gitlab-runner/.runner_system_id

COPY --chmod=755 entrypoint /

STOPSIGNAL SIGQUIT
VOLUME ["/etc/gitlab-runner", "/home/gitlab-runner"]
ENTRYPOINT ["/usr/bin/dumb-init", "/entrypoint"]
CMD ["run", "--user=gitlab-runner", "--working-directory=/home/gitlab-runner"]

### CUSTOM IMAGE with RUBY

ARG RUBY_INSTALLER_VERSION=0.9.3
ARG RUBY_VERSION=3.2.2
ARG LYCHEE_VERSION=0.15.1

FROM gitlab-runner

RUN echo "Installing needed dependencies" \
  && apt-get update && apt-get install -y --no-install-recommends \
  locales \
  imagemagick \
  build-essential \
  zlib1g-dev \
  jupyter-nbconvert \
  npm && \
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

RUN npm install -g purgecss

RUN echo "Installing lychee" && \
    wget -O lychee.tar.gz "https://github.com/lycheeverse/lychee/releases/download/v${LYCHEE_VERSION}/lychee-v${LYCHEE_VERSION}-x86_64-unknown-linux-gnu.tar.gz" && \
    tar -xzf lychee.tar.gz && \
    mv lychee /usr/bin/lychee && \
    rm -rf lychee.tar.gz


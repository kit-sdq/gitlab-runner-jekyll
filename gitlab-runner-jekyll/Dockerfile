ARG RUBY_INSTALLER_VERSION=0.9.3
ARG RUBY_VERSION=3.2.2

FROM gitlab/gitlab-runner:latest

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

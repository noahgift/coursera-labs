FROM ubuntu:19.10

ENV DEBIAN_FRONTEND=noninteractive

# Common sdk or packages required to install languages
RUN apt-get update && apt-get install --no-install-recommends -y \
    gpg \
    curl \
    wget \
    lsb-release \
    add-apt-key \
    ca-certificates \
    dumb-init \
    git \
    sudo \
    gdb \
    build-essential \
    ucommon-utils \
    zip \
    unzip \
    openssh-server

# Node SDK
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install --no-install-recommends -y nodejs \
    && npm install -g typescript

# Golang SDK
ENV GO_VERSION="1.12.4"
RUN curl -sL https://dl.google.com/go/go${GO_VERSION}.linux-amd64.tar.gz | tar -xz -C /usr/local

# Python SDK
RUN apt-get install --no-install-recommends -y \
    python3 \
    python-dev \
    python3-pip

RUN python3 -m pip install --upgrade setuptools \
    && python3 -m pip install wheel \
    && python3 -m pip install pylint \
    && python3 -m pip install requests

# Java 11 SDK
RUN apt-get install --no-install-recommends -y \
    default-jre-headless \
    default-jdk-headless \
    maven \
    gradle

# Scala SDK
RUN apt-get remove scala-library scala \
    && wget -q http://scala-lang.org/files/archive/scala-2.13.1.deb \
    && dpkg -i scala-2.13.1.deb \
    && apt-get install scala --no-install-recommends -y

# SBT
RUN echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list \
    && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 \
    && apt-get update \
    && apt-get install sbt --no-install-recommends -y

# C and C++ SDK
RUN apt-get install build-essential --no-install-recommends -y \
    valgrind \
    gdb

# C# SDK
RUN apt-get install mono-complete --no-install-recommends -y

# Ruby SDK
RUN apt-get install ruby-full --no-install-recommends -y

# Rust SDK (note, this requires build-essential to exist)
ENV RUSTUP_HOME "/bin/rustup"
ENV CARGO_HOME "/bin/cargo"
RUN curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh -s -- -y

# PHP SDK
RUN apt-get install php --no-install-recommends -y

# Chromium SDK
RUN curl -sLO https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
RUN apt-get install -y ./google-chrome-stable_current_amd64.deb
RUN rm google-chrome-stable_current_amd64.deb

# Code-Server
RUN apt-get install --no-install-recommends -y \
    libarchive-tools \
    locales

RUN localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8
ENV DISABLE_TELEMETRY true

# actual install of code server
ENV CODE_VERSION="3.5.0"
RUN curl -fsSL https://code-server.dev/install.sh | bash /dev/stdin --version $CODE_VERSION

# Setup User
RUN groupadd -r coder \
    && useradd -m -r coder -g coder -s /bin/bash \
    && echo "coder ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd
USER coder

# Setup User Go Environment
RUN mkdir /home/coder/go
ENV GOPATH "/home/coder/go"
ENV PATH "${PATH}:/usr/local/go/bin:/home/coder/go/bin"

# Setup User Visual Studio Code Extensions
ENV VSCODE_USER "/home/coder/.local/share/code-server/User"
ENV VSCODE_EXTENSIONS "/home/coder/.local/share/code-server/extensions"

RUN mkdir -p ${VSCODE_USER}

# Setup Code Runner
RUN code-server --install-extension formulahendry.code-runner --extensions-dir=$VSCODE_EXTENSIONS

# Setup Test Runner
RUN code-server --install-extension hbenl.vscode-test-explorer --extensions-dir=$VSCODE_EXTENSIONS

# Setup Go Extension
RUN code-server --install-extension golang.go --extensions-dir=$VSCODE_EXTENSIONS
RUN go get -u \
    github.com/mdempsky/gocode \
    github.com/uudashr/gopkgs/cmd/gopkgs \
    github.com/ramya-rao-a/go-outline \
    github.com/acroca/go-symbols \
    golang.org/x/tools/cmd/guru \
    golang.org/x/tools/cmd/gorename \
    github.com/go-delve/delve/cmd/dlv \
    github.com/stamblerre/gocode \
    github.com/rogpeppe/godef \
    github.com/sqs/goreturns \
    golang.org/x/lint/golint

# Setup Python Extension
RUN code-server --install-extension ms-python.python --extensions-dir=$VSCODE_EXTENSIONS

# Setup Java Extension
RUN code-server --install-extension redhat.java --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension vscjava.vscode-java-debug --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension vscjava.vscode-java-test --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension vscjava.vscode-maven --extensions-dir=$VSCODE_EXTENSIONS

# Setup environment variables required for Scala to work (without cosmetic errors)
RUN echo 'export JAVA_HOME=$(readlink -f /usr/bin/javac | sed "s:/bin/javac::")' >> /home/coder/.bashrc
RUN echo "export TERM=xterm" >> /home/coder/.bashrc

# Setup Scala Extension
RUN code-server --install-extension scala-lang.scala --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension scalameta.metals --extensions-dir=$VSCODE_EXTENSIONS

# Setup Groovy Extension
RUN code-server --install-extension marlon407.code-groovy --extensions-dir=$VSCODE_EXTENSIONS

# Setup C/C++ Extension: coder-server downloads the wrong binary https://github.com/cdr/code-server/issues/2120
COPY extensions/ /tmp/
RUN mkdir -p $VSCODE_EXTENSIONS/ms-vscode.cpptools-1.0.1 \
  && bsdtar --strip-components=1 -xf /tmp/cpptools-linux-1.0.1.vsix -C $VSCODE_EXTENSIONS/ms-vscode.cpptools-1.0.1 extension
RUN code-server --install-extension danielpinto8zz6.c-cpp-compile-run --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension matepek.vscode-catch2-test-adapter --extensions-dir=$VSCODE_EXTENSIONS

# Setup C# Extension (disabled since it's always trying to install packages)
#RUN code-server --install-extension ms-dotnettools.csharp --extensions-dir=$VSCODE_EXTENSIONS

# Setup HTML/CSS Extension
RUN code-server --install-extension ecmel.vscode-html-css --extensions-dir=$VSCODE_EXTENSIONS

# Setup Rust alias
RUN cat /usr/bin/cargo/env >> /home/coder/.bash_aliases

# Setup Ruby Extension
RUN code-server --install-extension rebornix.Ruby --extensions-dir=$VSCODE_EXTENSIONS

# Setup Rust Extension
RUN code-server --install-extension rust-lang.rust --extensions-dir=$VSCODE_EXTENSIONS

# Setup JavaScript/Node Extensions
RUN code-server --install-extension waderyan.nodejs-extension-pack --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension xabikos.JavaScriptSnippets --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension ms-vscode.node-debug2 --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension octref.vetur --extensions-dir=$VSCODE_EXTENSIONS

# Setup PHP Extension
RUN code-server --install-extension bmewburn.vscode-intelephense-client --extensions-dir=$VSCODE_EXTENSIONS

# Setup Chrome Extension
RUN code-server --install-extension msjsdiag.debugger-for-chrome --extensions-dir=$VSCODE_EXTENSIONS
RUN code-server --install-extension ritwickdey.LiveServer --extensions-dir=$VSCODE_EXTENSIONS

# Setup Chrome Viewer (can't be found on the code-server marketplace currently as of 07/20/2020)
# We don't want to download the vsix file here because it fails too often.
RUN mkdir -p $VSCODE_EXTENSIONS/vscode-browser-preview \
  && bsdtar --strip-components=1 -xf /tmp/auchenberg.vscode-browser-preview-0.6.7.vsix -C $VSCODE_EXTENSIONS/vscode-browser-preview extension

# This fixes the health check bug
COPY index.html /home/travis/build/codercom/code-server/packages/server/build/web/

# Setup action buttons extension for submit button
RUN code-server --install-extension seunlanlege.action-buttons --extensions-dir=$VSCODE_EXTENSIONS

# Remove extra notifications
RUN find /home/coder/.local/share/code-server/extensions -wholename "*action-buttons*/out/src/init.js" \
    | xargs sed -i --expression='/showInformationMessage/d'
RUN find /home/coder/.local/share/code-server/extensions -wholename "*ritwickdey.liveserver*/out/src/announcement/index.js" \
    | xargs sed -i --expression='/showInformationMessage/d'

ENV SHELL /bin/bash

USER root

# Add python scripts to add submit button
COPY --chown=coder:coder coursera-submit-button/ /home/coder/.local/share/code-server/User/coursera/

# Turns off the welcome page
COPY --chown=coder:coder settings.json /home/coder/.local/share/code-server/User/

# Makes terminal opens bash by default
COPY start.sh /usr/local/bin/
COPY config.yaml /tmp/config/code-server/
RUN chmod 755 /usr/local/bin/start.sh

# Put coursera submission scripts in python lib
COPY coursera-submission /tmp/coursera
RUN for d in $(ls /usr/lib | grep "^python*") ; do cp -r /tmp/coursera /usr/lib/$d; done

USER coder

# Setup User Workspace
RUN mkdir -p /home/coder/project
WORKDIR /home/coder/project

ENTRYPOINT ["dumb-init", "--"]
CMD ["start.sh"]

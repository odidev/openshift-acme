FROM centos:centos7

RUN INSTALL_PKGS="which git tar wget hostname sysvinit-tools util-linux bsdtar epel-release \
      socat ethtool device-mapper iptables tree findutils nmap-ncat e2fsprogs xfsprogs lsof" && \
    yum install -y $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all && \
    mkdir -p /var/lib/origin

FROM base_image:latest

ENV VERSION=1.4.3 \
    GOARM=5 \
    GOPATH=/go \
    GOROOT=/usr/local/go \
    OS_VERSION_FILE=/go/src/github.com/openshift/origin/os-version-defs \
    TMPDIR=/openshifttmp
ENV PATH=$PATH:$GOROOT/bin:$GOPATH/bin
RUN mkdir $TMPDIR && \
    INSTALL_PKGS="make gcc zip mercurial krb5-devel bsdtar" && \
    yum install -y $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all && \
    if [ `uname -m` = "aarch64" ] ; then \
    curl -L https://github.com/protocolbuffers/protobuf/releases/download/v3.17.3/protoc-3.17.3-linux-aarch_64.zip | bsdtar -C /usr/local -xf - && \
    curl  https://dl.google.com/go/go1.13.linux-arm64.tar.gz | tar -C /usr/local -xzf - ; \
    else \
    curl -L https://github.com/google/protobuf/releases/download/v3.0.0-beta-4/protoc-3.0.0-beta-4-linux-x86_64.zip | bsdtar -C /usr/local -xf - && \
    curl https://storage.googleapis.com/golang/go$VERSION.linux-amd64.tar.gz | tar -C /usr/local -xzf - ; \
    fi && \
    chmod ug+x /usr/local/bin/protoc && \
    touch /os-build-image

WORKDIR /go/src/github.com/openshift/origin

FROM golang_image:latest as builder
WORKDIR /go/src/github.com/tnozicka/openshift-acme
COPY . .
RUN make build --warn-undefined-variables

FROM registry.access.redhat.com/ubi8/ubi-minimal:latest
COPY --from=builder /go/src/github.com/tnozicka/openshift-acme/openshift-acme-controller /usr/bin/openshift-acme-controller
ENTRYPOINT ["/usr/bin/openshift-acme-controller"]

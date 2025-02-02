FROM docker.io/library/debian:bullseye-slim

ARG JITSI_RELEASE=stable
ARG TARGETPLATFORM
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

RUN groupadd --gid 1000 noroot; \
  useradd --uid 1000 --gid 1000 -m noroot

COPY --chown=noroot:noroot rootfs /

RUN case ${TARGETPLATFORM} in \
         "linux/amd64")  TPL_ARCH=amd64  ;; \
         "linux/arm64")  TPL_ARCH=arm64  ;; \
    esac && \
    case ${TARGETPLATFORM} in \
         "linux/amd64")  S6_ARCH=x86_64  ;; \
         "linux/arm64")  S6_ARCH=aarch64  ;; \
    esac && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y apt-transport-https apt-utils ca-certificates gnupg wget xz-utils && \
    wget -qO /usr/bin/tpl https://github.com/jitsi/tpl/releases/download/v1.0.4/tpl-linux-${TPL_ARCH} && \
    wget -qO - https://github.com/just-containers/s6-overlay/releases/download/v3.1.5.0/s6-overlay-${S6_ARCH}.tar.xz | tar -xJp -C / && \
    wget -qO - https://github.com/just-containers/s6-overlay/releases/download/v3.1.5.0/s6-overlay-noarch.tar.xz | tar -xJp -C / && \
    wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | gpg --dearmour > /etc/apt/trusted.gpg.d/jitsi.gpg && \
    echo "deb https://download.jitsi.org $JITSI_RELEASE/" > /etc/apt/sources.list.d/jitsi.list && \
    echo "deb http://ftp.debian.org/debian bullseye-backports main" > /etc/apt/sources.list.d/backports.list && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get dist-upgrade -y && \
    apt-cleanup && \
    chown -R noroot:noroot /usr/bin/tpl && \
    chown -R noroot:noroot /etc/cont-init.d && \
    chown -R noroot:noroot /etc/services.d && \
    chown -R noroot:noroot /etc/timezone && \
    chmod +x /usr/bin/tpl && \
    chmod +x /etc/cont-init.d/*
    
RUN [ "$JITSI_RELEASE" = "unstable" ] && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y jq procps curl vim iputils-ping net-tools && \
    apt-cleanup || \
    true

ENTRYPOINT [ "/init" ]
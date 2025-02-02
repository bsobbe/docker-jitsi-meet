ARG JITSI_REPO=jitsi
ARG BASE_TAG=latest

FROM ${JITSI_REPO}/base:${BASE_TAG} as builder

RUN apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y \
      build-essential \
      lua5.4 \
      liblua5.4-dev \
      libreadline-dev \
      git \
      unzip \
      wget && \
    mkdir /tmp/luarocks && \
    wget -qO - https://luarocks.github.io/luarocks/releases/luarocks-3.8.0.tar.gz | tar xfz - --strip-components 1 -C /tmp/luarocks && \
    cd /tmp/luarocks && ./configure && make && make install && cd - && \
    luarocks install basexx 0.4.1-1 && \
    luarocks install lua-cjson 2.1.0-1 && \
    luarocks install net-url 0.9-1

FROM ${JITSI_REPO}/base:${BASE_TAG}

LABEL org.opencontainers.image.title="Prosody IM"
LABEL org.opencontainers.image.description="XMPP server used for signalling."
LABEL org.opencontainers.image.url="https://prosody.im/"
LABEL org.opencontainers.image.source="https://github.com/jitsi/docker-jitsi-meet"
LABEL org.opencontainers.image.documentation="https://jitsi.github.io/handbook/"

ARG VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN="1.8.0"

RUN wget -qO /etc/apt/trusted.gpg.d/prosody.gpg https://prosody.im/files/prosody-debian-packages.key && \
    echo "deb http://packages.prosody.im/debian bullseye main" > /etc/apt/sources.list.d/prosody.list && \
    apt-dpkg-wrap apt-get update && \
    apt-dpkg-wrap apt-get install -y \
      lua5.4 \
      prosody \
      libldap-common \
      sasl2-bin \
      libsasl2-modules-ldap \
      lua-cyrussasl \
      lua-inspect \
      lua-ldap \
      lua-luaossl \
      lua-sec \
      lua-unbound && \
    apt-dpkg-wrap apt-get -d install -y jitsi-meet-prosody && \
    dpkg -x /var/cache/apt/archives/jitsi-meet-prosody*.deb /tmp/pkg && \
    mv /tmp/pkg/usr/share/jitsi-meet/prosody-plugins /prosody-plugins && \
    rm -rf /tmp/pkg /var/cache/apt && \
    apt-cleanup && \
    rm -rf /etc/prosody && \
    mv /usr/share/lua/5.3/inspect.lua /usr/share/lua/5.4/ && \
    rm -rf /usr/lib/lua/{5.1,5.2,5.3} && \
    rm -rf /usr/share/lua/{5.1,5.2,5.3} && \
    wget -qO /prosody-plugins/mod_auth_cyrus.lua https://hg.prosody.im/prosody-modules/raw-file/65438e4ba563/mod_auth_cyrus/mod_auth_cyrus.lua && \
    wget -qO /prosody-plugins/sasl_cyrus.lua https://hg.prosody.im/prosody-modules/raw-file/65438e4ba563/mod_auth_cyrus/sasl_cyrus.lua  && \
    wget https://github.com/matrix-org/prosody-mod-auth-matrix-user-verification/archive/refs/tags/v$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN.tar.gz && \
    tar -xf v$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN.tar.gz && \
    mv prosody-mod-auth-matrix-user-verification-$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN/mod_auth_matrix_user_verification.lua /prosody-plugins && \
    mv prosody-mod-auth-matrix-user-verification-$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN/mod_matrix_power_sync.lua /prosody-plugins && \
    rm -rf prosody-mod-auth-matrix-user-verification-$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN v$VERSION_MATRIX_USER_VERIFICATION_SERVICE_PLUGIN.tar.gz

COPY --chown=noroot:noroot rootfs/ /

COPY --chown=noroot:noroot --from=builder /usr/local/lib/lua/5.4 /usr/local/lib/lua/5.4
COPY --chown=noroot:noroot --from=builder /usr/local/share/lua/5.4 /usr/local/share/lua/5.4

RUN mkdir -pm777 /var/run/saslauthd && \
    mkdir -p /config/noroot_data && \
    mkdir -p /config/certs && \
    mkdir -p /config/conf.d && \
    mkdir -p /prosody-plugins-custom

RUN chown -R noroot:noroot /etc/cont-init.d && \
    chown -R noroot:noroot /etc/services.d && \
    chown -R noroot:noroot /etc/ldap && \
    chown -R noroot:noroot /run && \
    chown -R noroot:noroot /var && \
    chown -R noroot:noroot /defaults && \
    chown -R noroot:noroot /prosody-plugins && \
    chown -R noroot:noroot /prosody-plugins-custom && \
    mkdir -p /config/saslauthd

RUN chmod +x /etc/cont-init.d/* && \
    chmod +x /etc/services.d/10-saslauthd/* && \
    chmod +x /etc/services.d/prosody/*

RUN cp -r /defaults/* /config
RUN chown -R noroot:noroot /config

EXPOSE 5222 5280

USER noroot

VOLUME ["/config", "/prosody-plugins-custom"]

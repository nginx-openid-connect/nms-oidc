FROM debian:bullseye-slim

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

# Define NGINX versions for NGINX Plus and NGINX Plus modules
# Uncomment this block and the versioned nginxPackages block in the main RUN
# instruction to install a specific release
ENV NGINX_VERSION   28
ENV NJS_VERSION     0.7.9
ENV PKG_RELEASE     1~bullseye

RUN  mkdir -p /etc/ssl/nginx
COPY docker/build-context/ssl/*.crt /etc/ssl/nginx/
COPY docker/build-context/ssl/*.key /etc/ssl/nginx/

# Download certificate and key from the customer portal (https://account.f5.com)
# and copy to the build context
RUN set -x \
# Create nginx user/group first, to be consistent throughout Docker variants
    && addgroup --system --gid 101 nginx \
    && adduser --system --disabled-login --ingroup nginx --no-create-home --home /nonexistent --gecos "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y \
                        ca-certificates \
                        gnupg1 \
                        lsb-release \
                        vim-tiny \
    && \
    NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
    found=''; \
    for server in \
        hkp://keyserver.ubuntu.com:80 \
        pgp.mit.edu \
    ; do \
        echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
        apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
    done; \
    test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
    apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
# Install the latest release of NGINX Plus and/or NGINX Plus modules
# Uncomment individual modules if necessary
# Use versioned packages over defaults to specify a release
    && nginxPackages=" \
        nginx-plus \
        # nginx-plus=${NGINX_VERSION}-${PKG_RELEASE} \
        # nginx-plus-module-xslt \
        # nginx-plus-module-xslt=${NGINX_VERSION}-${PKG_RELEASE} \
        # nginx-plus-module-geoip \
        # nginx-plus-module-geoip=${NGINX_VERSION}-${PKG_RELEASE} \
        # nginx-plus-module-image-filter \
        # nginx-plus-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE} \
        # nginx-plus-module-perl \
        # nginx-plus-module-perl=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-plus-module-njs \
        nginx-plus-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${PKG_RELEASE} \
    " \
    && echo "Acquire::https::pkgs.nginx.com::Verify-Peer \"true\";" > /etc/apt/apt.conf.d/90nginx \
    && echo "Acquire::https::pkgs.nginx.com::Verify-Host \"true\";" >> /etc/apt/apt.conf.d/90nginx \
    && echo "Acquire::https::pkgs.nginx.com::SslCert     \"/etc/ssl/nginx/nginx-repo.crt\";" >> /etc/apt/apt.conf.d/90nginx \
    && echo "Acquire::https::pkgs.nginx.com::SslKey      \"/etc/ssl/nginx/nginx-repo.key\";" >> /etc/apt/apt.conf.d/90nginx \
    && printf "deb https://pkgs.nginx.com/plus/debian `lsb_release -cs` nginx-plus\n" > /etc/apt/sources.list.d/nginx-plus.list \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y \
                        $nginxPackages \
                        curl \
                        gettext-base \
    && apt-get remove --purge -y lsb-release \
    && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx-plus.list \
    && rm -rf /etc/apt/apt.conf.d/90nginx \
# Forward request logs to Docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

RUN    mkdir -p /etc/nginx/conf.d                           \
    && mkdir -p /etc/nginx/sample                           \
    && mkdir -p /usr/share/nginx/html/css                   \
    && mkdir -p /var/lib/nginx/state                        \
    && mkdir -p /etc/controller-agent/configurator/auxfiles \
    && mkdir -p /var/run/nms                                \
    && mkdir -p /etc/nms/nginx/                             \
    && mkdir -p /etc/nms/nginx/locations/                   \
    && mkdir -p /etc/nms/nginx/oidc/                        \
    && mkdir -p /var/www/nms/ui/css/                        \
    && mkdir -p /var/www/nms/json/

VOLUME /usr/share/nginx/html
VOLUME /var/www/nms/ui/
VOLUME /etc/nginx
VOLUME /etc/nms/nginx

RUN    chmod -R 644 /etc/nginx
RUN    chmod -R 644 /etc/nginx/conf.d
RUN    chmod -R 644 /etc/nms/nginx/oidc/
RUN    chmod -R 644 /etc/nginx/sample
RUN    chmod -R 666 /usr/share/nginx/html/css
RUN    chmod -R 777 /var/www/nms/ui/
RUN    chmod -R 777 /var/www/nms/ui/css
RUN    chmod -R 777 /var/run/nms/

# COPY (Nginx configuration) directory
COPY  docker/build-context/nginx/             /etc/nginx/
COPY  docker/build-context/nms/               /etc/nms/
COPY  docker/build-context/content/           /var/www/nms/ui/
COPY  docker/build-context/var/www/nms/json   /var/www/nms/json/
COPY  nms-http-*.conf                         /etc/nginx/conf.d/
COPY  openid_*.conf                           /etc/nms/nginx/oidc/
COPY  openid_connect.js                       /etc/nms/nginx/oidc/
COPY  config-oidc-var.sh                      /etc/nms/nginx/oidc/

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]

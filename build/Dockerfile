FROM nginx:alpine AS builder

LABEL maintainer="Paul Bargewell <paul.bargewell@opusvl.com>"

ENV NAXSI_VERSION=1.3
ENV GEO_DB_RELEASE=2021-02

RUN apk add --no-cache --virtual .build-deps \
    gcc \
    geoip-dev \
    libc-dev \
    make \
    openssl-dev \
    pcre-dev \
    python3-dev \
    py3-pip \
    zlib-dev

WORKDIR /usr/local

# RUN wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
#     wget https://github.com/nbs-system/naxsi/archive/${NAXSI_VERSION}.tar.gz && \
#     mkdir -p /usr/src && \
#     tar -zxC /usr/src -f nginx-${NGINX_VERSION}.tar.gz && \
#     tar -zxC /usr/src -f ${NAXSI_VERSION}.tar.gz

RUN wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    mkdir -p /usr/src && \
    tar -zxC /usr/src -f nginx-${NGINX_VERSION}.tar.gz

COPY naxsi-${NAXSI_VERSION} /usr/src/naxsi-${NAXSI_VERSION}

#     ./configure --with-cc-opt="-Wno-error" \
# RUN cd /usr/local/nginx-${NGINX_VERSION}/ && \
#     ./configure \
#     --conf-path=/etc/nginx/nginx.conf --add-dynamic-module=../naxsi-${NAXSI_VERSION}/naxsi_src/ \
#     --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body \
#     --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log \
#     --http-proxy-temp-path=/var/lib/nginx/proxy --lock-path=/var/lock/nginx.lock \
#     --pid-path=/var/run/nginx.pid --with-http_ssl_module \
#     --without-mail_pop3_module --without-mail_smtp_module \
#     --without-mail_imap_module --without-http_uwsgi_module \
#     --without-http_scgi_module --with-ipv6 --prefix=/usr && \
#     make && \
#     make modules && \
#     make install && \
#     mkdir -p /etc/nginx/modules && \
#     cp objs/ngx_http_naxsi_module.so /etc/nginx/modules/ngx_http_naxsi_module.so

RUN CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') \
    cd /usr/src/nginx-$NGINX_VERSION && \
    ./configure --with-compat $CONFARGS --add-dynamic-module=/usr/src/naxsi-${NAXSI_VERSION}/naxsi_src/ && \
    make && \
    make modules && \
    make install && \
    mkdir -p /usr/local/nginx/modules/ && \
    cp objs/ngx_http_naxsi_module.so /usr/local/nginx/modules/ngx_http_naxsi_module.so

RUN cd /usr/src/naxsi-${NAXSI_VERSION} && \
    pip install --upgrade pip && \
    cp naxsi_config/naxsi_core.rules /etc/nginx/ && \
    cd nxapi && \
    pip install -r requirements.txt && \
    python3 setup.py install

FROM nginx:alpine

LABEL maintainer="Paul Bargewell <paul.bargewell@opusvl.com>"

ENV GEO_DB_RELEASE=2021-02

COPY --from=builder  /usr/local/nginx/modules/ngx_http_naxsi_module.so /usr/local/nginx/modules/ngx_http_naxsi_module.so
COPY --from=builder /etc/nginx/naxsi_core.rules /etc/nginx/naxsi_core.rules
# COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
# COPY --from=builder /etc/nginx /etc/nginx
# COPY --from=builder /usr/lib/python2.7/site-packages /usr/lib/python2.7/site-packages

RUN mkdir -p /etc/nginx/geoip && \
    wget -O - https://download.db-ip.com/free/dbip-city-lite-${GEO_DB_RELEASE}.mmdb.gz | gzip -d > /etc/nginx/geoip/dbip-city-lite.mmdb && \
    wget -O - https://download.db-ip.com/free/dbip-country-lite-${GEO_DB_RELEASE}.mmdb.gz | gzip -d > /etc/nginx/geoip/dbip-country-lite.mmdb

# COPY --from=builder /etc/nginx/ /etc/nginx/

COPY nginx.conf /etc/nginx/
COPY default.conf /etc/nginx/conf.d/default.conf

ADD naxsi/naxsi.rules /etc/nginx/naxsi.rules
ADD naxsi/nxapi.json /usr/local/etc/nxapi.json
# COPY --from=BUILDER /usr/lib/nginx/modules/ /usr/lib/nginx/modules/

# USER www-data

WORKDIR /usr/share/nginx/html

EXPOSE 8080 8443

VOLUME [ "/usr/share/nginx/html" ]

ENTRYPOINT [ "/usr/sbin/nginx", "-c", "/etc/nginx/nginx.conf" ]
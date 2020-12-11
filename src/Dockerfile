FROM nginx
LABEL maintainer="JB Bernard <email@jbbernard.com>"
ARG LICENSE_KEY="XXXXX"
VOLUME /etc/letsencrypt
EXPOSE 80
EXPOSE 443
# Do this apt/pip stuff all in one RUN command to avoid creating large
# intermediate layers on non-squashable docker installs

RUN apt update && \
    apt install -y gcc libpcre3-dev zlib1g-dev libbsd-dev libssl-dev libxslt-dev geoip-database libgeoip-dev  && \
    apt install -y libmaxminddb0 libmaxminddb-dev mmdb-bin git wget tar make && \
    apt install -y python3 python3-dev libffi6 libffi-dev  curl build-essential procps && \
    curl -L 'https://bootstrap.pypa.io/get-pip.py' | python3 && \
    pip install -U cffi certbot  && \
    cd /tmp && git clone https://github.com/masonicboom/ipscrub && \
    git clone https://github.com/leev/ngx_http_geoip2_module && \
    echo $(echo "$(nginx -v 2>&1)" |grep -Po '(?<=nginx/)[0-9|\.]+') > /tmp/nginx_version.txt && \
    cat /tmp/nginx_version.txt && \
    wget "https://nginx.org/download/nginx-$(cat /tmp/nginx_version.txt).tar.gz" && \
    tar -xzvf "nginx-$(cat /tmp/nginx_version.txt).tar.gz" && \
    nginx -V 2>&1 | egrep  "^configure" | cut -d: -f2 > /tmp/nginx_build_options.txt && \
    cd "nginx-$(cat /tmp/nginx_version.txt)" && sh -c "./configure  $(cat /tmp/nginx_build_options.txt) --add-dynamic-module=../ipscrub/ipscrub --add-dynamic-module=../ngx_http_geoip2_module" && \
    make modules && \
    cp objs/ngx_ipscrub_module.so /etc/nginx/modules && \
    cp objs/ngx_http_geoip2_module.so /etc/nginx/modules && \
    cd /tmp && wget "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=${LICENSE_KEY}&suffix=tar.gz"  -O  GeoLite2Country.tar.gz && \
    wget "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-City&license_key=${LICENSE_KEY}&suffix=tar.gz"  -O  GeoLite2City.tar.gz && \
    tar -xvf GeoLite2Country.tar.gz && \
    cd $(ls -d GeoLite2-Country*/|head -n 1) && mv GeoLite2-Country.mmdb /etc/nginx/GeoLite2-Country.mmdb && cd .. && rm  GeoLite2Country.tar.gz && rm -rf $(ls -d GeoLite2-Country*/|head -n 1) && \
    tar -xvf GeoLite2City.tar.gz && \
    cd $(ls -d GeoLite2-City*/|head -n 1) && mv GeoLite2-City.mmdb /etc/nginx/GeoLite2-City.mmdb && cd .. && rm  GeoLite2City.tar.gz && rm -rf $(ls -d GeoLite2-City*/|head -n 1) && \
    rm -rf /tmp/* && \
    ls /tmp/ && \
    apt remove --purge -y  python3-dev build-essential libffi-dev libssl-dev curl && \
    apt remove --purge -y  gcc libpcre3-dev zlib1g-dev libbsd-dev libssl-dev libxslt-dev geoip-database libgeoip-dev  && \
    apt remove --purge -y  git wget make  && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# Copy in scripts for certbot
COPY ./scripts/ /scripts
RUN chmod +x /scripts/*.sh

# Add /scripts/startup directory to source more startup scripts
RUN mkdir -p /scripts/startup

# Copy in default nginx configuration (which just forwards ACME requests to
# certbot, or redirects to HTTPS, but has no HTTPS configurations by default).
RUN rm -f /etc/nginx/conf.d/*
COPY nginx_conf.d/ /etc/nginx/conf.d/
COPY nginx.conf /etc/nginx/nginx.conf

ENTRYPOINT []
CMD ["/bin/bash", "/scripts/entrypoint.sh"]

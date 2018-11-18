FROM debian:stretch-slim

RUN apt-get -y update && \
    apt-get -y install gnupg2 lsb-release software-properties-common wget && \
    wget -qO - https://openresty.org/package/pubkey.gpg | apt-key add - && \
    add-apt-repository -y "deb http://openresty.org/package/debian $(lsb_release -sc) openresty" && \
    apt-get update && \
    apt-get -y install openresty && \
    apt-get remove -y --purge gnupg2 lsb-release software-properties-common wget && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin"

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 8080

CMD ["/usr/bin/openresty", "-g", "daemon off;"]

FROM debian:stretch-slim

RUN apt-get -y update && \
    apt-get -y install software-properties-common ca-certificates gnupg2 lsb-release wget && \
    wget -qO - https://openresty.org/package/pubkey.gpg | apt-key add - && \
    add-apt-repository -y "deb http://openresty.org/package/debian $(lsb_release -sc) openresty" && \
    apt-get update && \
    apt-get -y install openresty && \
    apt-get remove -y --purge software-properties-common gnupg2 lsb-release wget && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin"

COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

EXPOSE 8080

CMD ["/usr/bin/openresty", "-g", "daemon off;"]

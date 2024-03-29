FROM centos:7
# version 21.0

#MAINTAINER MW1-4@tmax.co.kr

# 0) Install necessary packages through yum in CentOS
RUN set -eux; \
    yum update -y; \
    yum install -y \
        make \
        gcc \
        gcc-c++ \
        wget \
        perl \
        expat-devel \
        ca-certificates \
    ; \rm -rf /var/lib/apt/lists/*

# 1) Httpd env set
ENV SOURCE_PATH /usr/local/src
ENV HTTPD_PREFIX /usr/local/apache2
ENV HTTPD_CONF_PATH $HTTPD_PREFIX/conf
ENV SSL_PATH $HTTPD_CONF_PATH
ENV LICENSE_PATH $HTTPD_PREFIX/license

ENV LIBRARY_DOWNLOAD_PATH https://github.com/TmaxSoftOfficial/HyperFrame-Apache-Image/raw/master/lib

ENV APR_VERSION 1.7.0
ENV APR_UTIL_VERSION 1.6.1
ENV PCRE_VERSION 8.45
ENV OPENSSL_VERSION 1.1.1s
ENV APACHE_MAJOR_VERSION 2.4
ENV APACHE_MINOR_VERSION 48

ENV HTTP_PORT 80
ENV HTTPS_PORT 443

# environment PATH set
ENV PATH $HTTPD_PREFIX:$HTTPD_PREFIX/bin:$PATH

# 2) copy sample_key to use OpenSSL
COPY ./ssl/server.crt $SSL_PATH/
COPY ./ssl/server.key $SSL_PATH/
# copy shell script to set password for OpenSSL sample_key
COPY ./ssl/ssl_passwd.sh $SSL_PATH/
# comment out 2) if you do not want to use OpenSSL

# copy start.sh to run Apache
COPY ./start.sh $HTTPD_PREFIX/

# copy Apache license
COPY ./license/ $LICENSE_PATH

# 3) library build
# 3_1) apr  
RUN set -eux; \
    \
    cd $SOURCE_PATH; \
    wget $LIBRARY_DOWNLOAD_PATH/apr-${APR_VERSION}.tar.gz; \
    tar -zxvf apr-${APR_VERSION}.tar.gz; \
    cd apr-${APR_VERSION}; \
    ./configure --prefix=/usr/local/apr; \
    make; \
    make install
   
# 3_2) apr-util
RUN set -eux; \
    cd $SOURCE_PATH; \ 
    wget $LIBRARY_DOWNLOAD_PATH/apr-util-${APR_UTIL_VERSION}.tar.gz; \
    tar -zxvf apr-util-${APR_UTIL_VERSION}.tar.gz; \
    cd apr-util-${APR_UTIL_VERSION}; \
    ./configure --with-apr=/usr/local/apr; \
    make; \
    make install
    
# 3_3) pcre
RUN set -eux; \
    cd $SOURCE_PATH; \  
    wget $LIBRARY_DOWNLOAD_PATH/pcre-${PCRE_VERSION}.tar.gz; \
    tar -zxvf pcre-${PCRE_VERSION}.tar.gz; \
    cd pcre-${PCRE_VERSION}; \
    ./configure --prefix=/usr/local/pcre; \
    make; \
    make install
    
# 3_4) OpenSSL
RUN set -eux; \
    cd $SOURCE_PATH; \ 
    wget $LIBRARY_DOWNLOAD_PATH/openssl-${OPENSSL_VERSION}.tar.gz; \
    tar -zxvf openssl-${OPENSSL_VERSION}.tar.gz; \
    cd openssl-${OPENSSL_VERSION}; \
    ./config --prefix=/usr/local/ssl; \
    # source-based compilation and generate installation binaries
    make; \
    make install; \
    echo /usr/local/ssl/lib >> /etc/ld.so.conf; \
    /sbin/ldconfig
    
# 3_5) httpd
RUN set -eux; \
    cd $SOURCE_PATH; \  
    wget $LIBRARY_DOWNLOAD_PATH/httpd-${APACHE_MAJOR_VERSION}.${APACHE_MINOR_VERSION}.tar.gz; \
    tar -zxvf httpd-${APACHE_MAJOR_VERSION}.${APACHE_MINOR_VERSION}.tar.gz; \
    cd httpd-${APACHE_MAJOR_VERSION}.${APACHE_MINOR_VERSION}; \
    ./configure \
        --prefix=$HTTPD_PREFIX \
        --enable-mods-shared=all \
        --enable-ssl=shared \
        --with-pcre=/usr/local/pcre \
        --with-ssl=/usr/local/ssl \
    ; \
    # source-based compilation and generate installation binaries
    make; \
    make install

RUN set -eux; \
    rm -rf $SOURCE_PATH/*; \
    \
    # permission settings
    chmod 755 $HTTPD_CONF_PATH/ssl_passwd.sh; \
    chmod 755 $HTTPD_PREFIX/start.sh; \
    \
    # Apache version check
    httpd -v

# 4) copy configuration file
COPY ./conf/httpd.conf $HTTPD_CONF_PATH/httpd.conf
COPY ./conf/httpd-ssl.conf $HTTPD_CONF_PATH/extra/httpd-ssl.conf
COPY ./conf/httpd-vhosts.conf $HTTPD_CONF_PATH/extra/httpd-vhosts.conf
# DocumentRoot in httpd.conf : "/usr/local/apache2/htdocs"

# http port number: 80 https port number : 443
EXPOSE 80 443

# run Apache 
CMD start.sh



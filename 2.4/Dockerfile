FROM debian:jessie

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
#RUN groupadd -r www-data && useradd -r --create-home -g www-data www-data

ENV HTTPD_PREFIX /usr/local/apache2
ENV PATH $PATH:$HTTPD_PREFIX/bin
RUN mkdir -p "$HTTPD_PREFIX" \
	&& chown www-data:www-data "$HTTPD_PREFIX"
WORKDIR $HTTPD_PREFIX

# install httpd runtime dependencies
# https://httpd.apache.org/docs/2.4/install.html#requirements
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		libapr1 \
		libaprutil1 \
		libpcre++0 \
	&& rm -r /var/lib/apt/lists/*

# see https://httpd.apache.org/download.cgi#verify
RUN gpg --keyserver pgp.mit.edu --recv-keys A93D62ECC3C8EA12DB220EC934EA76E6791485A8

ENV OPEN_SSL_VERSION 1.0.1j
ENV OPEN_DDL_GZ_URL https://www.openssl.org/source/openssl-$OPEN_SSL_VERSION.tar.gz
ENV HTTPD_VERSION 2.4.10
ENV HTTPD_BZ2_URL https://www.apache.org/dist/httpd/httpd-$HTTPD_VERSION.tar.bz2

RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		build-essential \
		ca-certificates \
		curl \
		gcc \
		libapr1-dev \
		libaprutil1-dev \
		libpcre++-dev \
		make \
		openssl \
	&& rm -r /var/lib/apt/lists/* \
	&& curl -SL "$OPEN_DDL_GZ_URL" -o openssl.tar.gz \
	&& curl -SL "$OPEN_DDL_GZ_URL.sha1" -o openssl.tar.gz.sha1 \
	&& sha1sum openssl.tar.gz | awk '{print $1;}' | cmp - openssl.tar.gz.sha1 \
	&& mkdir -p src/openssl \
	&& tar -xzvf openssl.tar.gz -C src/openssl --strip-components=1 \
	&& rm openssl.tar.gz* \
	&& mkdir -p /usr/local/openssl \
	&& cd src/openssl \
	&& ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl \
	&& make \
	&& make install \
	&& cd $HTTPD_PREFIX \
	&& curl -SL "$HTTPD_BZ2_URL" -o httpd.tar.bz2 \
	&& curl -SL "$HTTPD_BZ2_URL.asc" -o httpd.tar.bz2.asc \
	&& gpg --verify httpd.tar.bz2.asc \
	&& mkdir -p src/httpd \
	&& tar -xvf httpd.tar.bz2 -C src/httpd --strip-components=1 \
	&& rm httpd.tar.bz2* \
	&& cd src/httpd \
	&& ./configure --enable-so --enable-ssl --with-ssl=/usr/local/openssl --enable-ssl-staticlib-deps --enable-mods-static=ssl --prefix=$HTTPD_PREFIX \
	&& make -j"$(nproc)" \
	&& make install \
	&& cd ../../ \
	&& rm -r src/httpd \
	&& sed -ri ' \
		s!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; \
		s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g; \
		' /usr/local/apache2/conf/httpd.conf \
	&& apt-get purge -y --auto-remove \
		build-essential \
		ca-certificates \
		curl \
		gcc \
		libapr1-dev \
		libaprutil1-dev \
		libpcre++-dev \
		make

EXPOSE 80
CMD ["httpd", "-DFOREGROUND"]

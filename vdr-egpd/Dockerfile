FROM uatschitchun/stretch-s6-base

WORKDIR /tmp
ENV ARCH=amd64

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main'

RUN apt-get update -q \
 && apt-get install -y --no-install-recommends \
	build-essential \
	git \
	unzip \
	wget \
	libarchive13 libarchive-dev \
	zlib1g zlib1g-dev \
	libcurl3-nss libcurl4-nss-dev \
	libxml2 libxml2-dev \
	libxslt1.1 libxslt1-dev \
	libmariadbclient18 libmariadb-dev libmariadb-dev-compat \
	libjpeg62-turbo libjpeg-dev \
	uuid uuid-dev \
	libjansson4 libjansson-dev \
	libimlib2 libimlib2-dev \
	libpython2.7 libpython-dev python-dev \
	libssl-dev \
	libmicrohttpd12 libmicrohttpd-dev

RUN mkdir -p /usr/lib/mysql/plugin \
 && mkdir -p /config \
 && git clone git://projects.vdr-developer.org/vdr-epg-daemon.git \
 && cd vdr-epg-daemon* \
 && sed -i 's/# include <my_global.h>/#/' epglv/src/epglv.h \
 && sed -i 's/# include <my_sys.h>/#/' epglv/src/epglv.h \
 && sed -i 's/#include <m_ctype.h>/#/' epglv/src/epglv.h \
 && sed -i 's/#include <m_string.h>/#/' epglv/src/epglv.h \
 && git clone https://github.com/3PO/epgd-plugin-tvm.git ./PLUGINS/tvm \
 && git clone https://github.com/chriszero/epgd-plugin-tvsp.git ./PLUGINS/tvsp \
 && sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen \
 && make && make plugins && make install


RUN cd /tmp \
 && wget https://github.com/FrodoVDR/channellogos/archive/master.tar.gz \
 && tar xzf master.tar.gz \
 && mkdir -p /var/epgd/www/channellogos \
 && cp -r channellogos-master/logos-orig/* /var/epgd/www/channellogos/

RUN apt-get remove -y \
	build-essential \
        git \
        libarchive-dev \
        zlib1g-dev \
        libcurl4-nss-dev \
        libxml2-dev \
        libxslt1-dev \
        libmariadb-dev \
        libmariadb-dev-compat \
        libjpeg-dev \
        uuid-dev \
        libjansson-dev \
        libimlib2-dev \
        libpython-dev python-dev \
        libssl-dev \
        libmicrohttpd-dev \
	wget \
 && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /epgd/config

COPY configs/* /epgd/config/

# install runit scripts
COPY epgd.sh /etc/services.d/epgd/run
COPY epghttpd.sh /etc/services.d/epghttpd/run

EXPOSE 8888 9999


################################################################################################################
# Target: builder                                                                                              #
# Build _builder_ image from stretch-s6-base for building vdr-epg-daemon*.deb used by vdr-epgd & mariadb-epglv #
################################################################################################################

FROM uatschitchun/stretch-s6-base AS builder
WORKDIR /tmp
ENV ARCH=amd64


RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main' \
 && apt-get update -q \
 && apt-get install -y --no-install-recommends \
        build-essential \
        nano \
        less \
        unzip \
        git \
        wget \
# VDR-EPGD
        libarchive-dev \
        zlib1g-dev \
#        libcurl4-nss-dev \
        libxml2-dev \
        libxslt1-dev \
        libmariadb-dev libmariadb-dev-compat \
        libjpeg-dev \
        uuid-dev \
        libjansson-dev \
        libimlib2-dev \
        libpython-dev python-dev \
        libssl-dev \
        libmicrohttpd-dev \
        debhelper \
        libcurl4-openssl-dev \
        dh-systemd \
        libsystemd-dev \
        dh-exec


#########################################################
# Target: build-epgd                                    #
# Using _builder_ image to build vdr-epg-daemon's debs  #
# Is beeing used as delivery container for debs & logos #
# targets: vdr-epgd && mariadb-epglv                    #
#########################################################

FROM builder AS build-epgd
WORKDIR /tmp
ENV ARCH=amd64

RUN mkdir -p vdr-epg-daemon/ \
 && sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

# Copy debian source package from host
COPY deb-source/* vdr-epg-daemon/

# Build debs
RUN cd vdr-epg* \
 && dpkg-source -x vdr-*.dsc \
 && cd vdr-* \
 && dpkg-buildpackage -uc -us -j`nproc`

# Get channel logos
RUN cd /tmp \
 && wget https://github.com/FrodoVDR/channellogos/archive/master.tar.gz \
 && tar xzf master.tar.gz \
 && mkdir -p /var/epgd/www/channellogos \
 && cp -r channellogos-master/logos-orig/* /var/epgd/www/channellogos/

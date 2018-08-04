FROM debian:stretch-slim

ENV ARCH=amd64

ENV DEBIAN_FRONTEND noninteractive
ENV _clean="rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*"
ENV _apt_clean="eval apt-get clean && $_clean"

# Install needed
RUN apt-get update \
  && apt-get install --no-install-recommends -y \
	apt-utils \
	locales \
	tzdata \
        software-properties-common \
        dirmngr \
        apt-transport-https \
        gnupg \
  && $_apt_clean

# Set your local timezone
RUN echo "Europe/Berlin" > /etc/timezone && \
	dpkg-reconfigure -f noninteractive tzdata

# fix locale.
RUN sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

ENV LANG de_DE.UTF-8
ENV LANGUAGE de_DE:de
ENV LC_ALL de_DE.UTF-8

# Install s6-overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v1.21.4.0/s6-overlay-amd64.tar.gz /tmp/
RUN tar zxf /tmp/s6-overlay-amd64.tar.gz -C / && $_clean

ENTRYPOINT [ "/init" ]

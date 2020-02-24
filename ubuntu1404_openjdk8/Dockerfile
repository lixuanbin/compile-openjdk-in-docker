FROM ubuntu:14.04
MAINTAINER lixuanbin <lxbjkben@163.com>
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY ./sources.list /etc/apt/
RUN apt-get update \
    && apt-get install -y  curl ssh zip unzip vim ant git mercurial build-essential ccache cpio g++ gcc curl libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libasound2-dev libelf-dev openjdk-7-jdk gdb
RUN mkdir -p /opt/openjdk 
COPY ./Hello.java /opt/openjdk/
# some other ways to get the source of openjdk8
# RUN git clone --depth 1 -b master git@github.com:AdoptOpenJDK/openjdk-jdk8u.git
# RUN hg clone http://hg.openjdk.java.net/jdk8u/jdk8u/
RUN cd /opt/openjdk && wget https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-src-b04-14_jan_2020.zip && unzip openjdk-8*.zip && mv openjdk openjdk8 && cd openjdk8 && sed -i 's/3%/3% 4%/g' hotspot/make/linux/Makefile && chmod a+x configure && ./configure --with-debug-level=slowdebug --with-target-bits=64 --with-freetype-include=/usr/include/freetype2 --with-freetype-lib=/usr/lib/x86_64-linux-gnu && make clean && make images
RUN mkdir -p /app
WORKDIR /app
ADD . /app

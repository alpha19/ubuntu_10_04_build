# This dockerfile is meant to be dynamic in that it downloads packages (other than via apt-get) via the internet
FROM ubuntu:10.04

# This Ubuntu doesn't have its packages on the normal server anymore
RUN sed -i -e "s/archive.ubuntu.com/old-releases.ubuntu.com/g" /etc/apt/sources.list

# Get some starter things
RUN apt-get update
RUN apt-get install tar wget gcc g++ make nano libc6-dev-i386 python-pip python-dev -y
RUN apt-get install python-argparse build-essential ia32-libs gcc-multilib g++-multilib -y
RUN apt-get install git-core python libcurl4-openssl-dev libz-dev gettext zlib1g-dev -y
RUN apt-get install checkinstall libgnutls-dev curl autoconf libtool -y

# Build new OpenSSL (needed for newer git and https to work)
ADD ./dependencies/openssl-1.0.2o.tar.gz /usr/local
ADD ./dependencies/openssl-1.0.2o.tar.gz.sha1 /usr/local
RUN cd /usr/local/openssl-1.0.2o && \
	./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl shared zlib && \
	make -j4 && \
	make install && \
	make clean && \
	make distclean && \
	echo "/usr/local/openssl/lib" >> /etc/ld.so.conf.d/openssl.conf && \
	ldconfig -v && \
	/usr/local/openssl/bin/openssl version

RUN rm /usr/local/openssl-1.0.2o.tar.gz.sha1 && \
	rm -r /usr/local/openssl-1.0.2o && \

# Build new Git
ADD ./dependencies/git-2.9.5.tar.gz /usr/local
RUN cd /usr/local/git-2.9.5 && \
	./configure --with-openssl=/usr/local/openssl && \
	make -j4 && \
	make install && \
	make clean && \
	make distclean && \
	cd /usr/local && \
	rm -r /usr/local/git-2.9.5 && \
	git --version && \
	echo "Host github.com\n\tStrictHostKeyChecking no\n" >> /etc/ssh/ssh_config

# Build new curl with new OpenSSL for Git
# Is the commit id below safe enough to say SSL_NO_VERIFY is ok?
RUN cd /usr/local && \
	GIT_SSL_NO_VERIFY=true git clone https://github.com/curl/curl.git && \
	cd /usr/local/curl && \
	git checkout 432eb5f5c254ee8383b2522ce597c9219877923e && \
	./buildconf && \
	LIBS="-ldl" ./configure --with-ssl=/usr/local/openssl --disable-shared && \
	make -j4 && \
	make install && \
	make clean && \
	make distclean && \
	cd /usr/local && \
	curl --version

# Download the latest .pem file for https connections via curl
RUN curl https://curl.haxx.se/ca/cacert.pem -o /cacert.pem && \
	echo "[http]" >> ~/.gitconfig && \
	echo "sslCAinfo = /cacert.pem" >> ~/.gitconfig

RUN rm /usr/local/openssl-1.0.2o.tar.gz.sha1 && \
	rm -r /usr/local/openssl-1.0.2o
# Let's go!
WORKDIR /
CMD "/bin/bash"

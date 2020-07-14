FROM openjdk:8-alpine

ENV SPARK_MASTER_PORT 7087
ENV SPARK_MASTER_WEBUI_PORT 8090
ENV SPARK_MASTER_LOG /spark/logs
ENV SPARK_WORKER_LOG /spark/logs
ENV SPARK_VERSION 2.4.5

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# change the source of alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 工具类
RUN apk add --no-cache ca-certificates
RUN apk --update --no-cache add \
        wget tar bash

ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
ENV PYTHON_VERSION 3.6.10

RUN set -ex \
	&& apk add --no-cache --virtual .fetch-deps \
		gnupg \
		tar \
		xz \
	\
	&& wget -O python.tar.xz "https://mirrors.huaweicloud.com/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://mirrors.huaweicloud.com/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm python.tar.xz \
	\
	&& apk add --no-cache --virtual .build-deps  \
		bluez-dev \
		bzip2-dev \
		coreutils \
		dpkg-dev dpkg \
		expat-dev \
		findutils \
		gcc \
		gdbm-dev \
		libc-dev \
		libffi-dev \
		libnsl-dev \
		libtirpc-dev \
		linux-headers \
		make \
		ncurses-dev \
		openssl-dev \
		pax-utils \
		readline-dev \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		xz-dev \
		zlib-dev \
# add build deps before removing fetch deps in case there's overlap
	&& apk del --no-network .fetch-deps \
	\
	&& cd /usr/src/python \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-loadable-sqlite-extensions \
		--enable-optimizations \
		--enable-option-checking=fatal \
		--enable-shared \
		--with-system-expat \
		--with-system-ffi \
		--without-ensurepip \
	&& make -j "$(nproc)" \
# set thread stack size to 1MB so we don't segfault before we hit sys.getrecursionlimit()
# https://github.com/alpinelinux/aports/commit/2026e1259422d4e0cf92391ca2d3844356c649d0
		EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
		LDFLAGS="-Wl,--strip-all" \
# setting PROFILE_TASK makes "--enable-optimizations" reasonable: https://bugs.python.org/issue36044 / https://github.com/docker-library/python/issues/160#issuecomment-509426916
		PROFILE_TASK='-m test.regrtest --pgo \
			test_array \
			test_base64 \
			test_binascii \
			test_binhex \
			test_binop \
			test_bytes \
			test_c_locale_coercion \
			test_class \
			test_cmath \
			test_codecs \
			test_compile \
			test_complex \
			test_csv \
			test_decimal \
			test_dict \
			test_float \
			test_fstring \
			test_hashlib \
			test_io \
			test_iter \
			test_json \
			test_long \
			test_math \
			test_memoryview \
			test_pickle \
			test_re \
			test_set \
			test_slice \
			test_struct \
			test_threading \
			test_time \
			test_traceback \
			test_unicode \
		' \
	&& make install \
	\
	&& find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec scanelf --needed --nobanner --format '%n#p' '{}' ';' \
		| tr ',' '\n' \
		| sort -u \
		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
		| xargs -rt apk add --no-cache --virtual .python-rundeps \
	&& apk del --no-network .build-deps \
	\
	&& find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' + \
	&& rm -rf /usr/src/python \
	\
	&& python3 --version

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
	&& ln -s idle3 idle \
	&& ln -s pydoc3 pydoc \
	&& ln -s python3 python \
	&& ln -s python3-config python-config

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 20.1.1
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/eff16c878c7fd6b688b9b4c4267695cf1a0bf01b/get-pip.py
ENV PYTHON_GET_PIP_SHA256 b3153ec0cf7b7bbf9556932aa37e4981c35dc2a2c501d70d91d2795aa532be79

RUN echo '52.216.142.252  github-production-release-asset-2e65be.s3.amazonaws.com' >> /etc/hosts
RUN echo '151.101.108.133  raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.228.133  raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.196.133  raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.128.133  raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.40.133 raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.76.133 raw.githubusercontent.com' >> /etc/hosts
RUN echo '151.101.24.133 raw.githubusercontent.com' >> /etc/hosts

RUN cat /etc/hosts

RUN set -ex; \
	\
	echo '52.216.142.252  github-production-release-asset-2e65be.s3.amazonaws.com' >> /etc/hosts; \
	echo '151.101.108.133  raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.228.133  raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.196.133  raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.128.133  raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.40.133 raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.76.133 raw.githubusercontent.com' >> /etc/hosts; \
	echo '151.101.24.133 raw.githubusercontent.com' >> /etc/hosts; \
	\
	wget -O get-pip.py "$PYTHON_GET_PIP_URL" --user-agent="Mozilla/5.0 (X11;U;Linux i686;en-US;rv:1.9.0.3) Geco/2008092416 Firefox/3.0.3" --no-check-certificate; \
	echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum -c -; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PYTHON_PIP_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py

# Spark 压缩包下载
RUN wget https://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop2.7.tgz

# 解压并删除多余压缩包
RUN tar -xzf spark-${SPARK_VERSION}-bin-hadoop2.7.tgz && \
    mv spark-${SPARK_VERSION}-bin-hadoop2.7 /spark && \
    rm spark-${SPARK_VERSION}-bin-hadoop2.7.tgz

# 复制环境变量
COPY bashrc /root/.bashrc
# 复制启动脚本(包括启动Master和Worker)到容器根目录
COPY scripts/* /

# 暴露端口
#EXPOSE 8080 7077 6066
EXPOSE 8090 7087 6076

# 默认启动 Spark-shell 暂不开启
# ENTRYPOINT ["/start-shell.sh"]

## Dockerfile
FROM ubuntu:24.04

LABEL maintainer="sshrise206516@163.com" \
      version="1.0" \
      description="\
      FastDFS is a lightweight object storage solution. If you need a general distributed file system for databases,\
      K8s and virtual machines (such as KVM), you can learn about FastCFS which achieves strong data consistency and high performance."


ENV FASTDFS_PATH=/opt/fdfs \
  FASTDFS_BASE_PATH=/var/fdfs \
  LIBFASTCOMMON_VERSION="V1.0.73" \
  LIBSERVERFRAME_VERSION="V1.2.3" \
  FASTDFS_NGINX_MODULE_VERSION="V1.24" \
  FASTDFS_VERSION="V6.12.1" \
  FREENGINX_VERSION="1.25.4" \
  TENGINE_VERSION="3.1.0" \
  PORT= \
  GROUP_NAME= \
  TRACKER_SERVER= \
  CUSTOM_CONFIG="false"

# update aliyon.com

# get all the dependences
# RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys  40976EAF437D05B5 && \
#     apt-key adv --keyserver keyserver.ubuntu.com --recv-keys  3B4FE6ACC0B21F32 && \
#     echo '\
#     deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse \n\
#     deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse \n\
#     deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse \n\
#     deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse \n\
#     ' | tee -a /etc/apt/sources.list && \
RUN apt-get update  && apt-get install gnupg curl git gcc make wget openssl libssl-dev -y  && rm -rf /var/lib/apt/lists/*

# create the dirs to store the files downloaded from internet
RUN mkdir -p ${FASTDFS_PATH}/libfastcommon \
  && mkdir -p ${FASTDFS_PATH}/fastdfs \
  && mkdir -p ${FASTDFS_PATH}/fastdfs-nginx-module \
  && mkdir ${FASTDFS_BASE_PATH} \
  && mkdir /nginx_conf && mkdir -p /usr/local/nginx/conf/conf.d

WORKDIR ${FASTDFS_PATH}

## compile the libfastcommon
RUN git clone -b $LIBFASTCOMMON_VERSION https://github.com/happyfish100/libfastcommon.git libfastcommon \
  && cd libfastcommon \
  && ./make.sh \
  && ./make.sh install \
  && rm -rf ${FASTDFS_PATH}/libfastcommon

## compile the libserverframe
RUN git clone -b $LIBSERVERFRAME_VERSION https://github.com/happyfish100/libserverframe.git libserverframe \
  && cd libserverframe \
  && ./make.sh \
  && ./make.sh install \
  && rm -rf ${FASTDFS_PATH}/libserverframe

## compile the fastdfs
RUN git clone -b $FASTDFS_VERSION https://github.com/happyfish100/fastdfs.git fastdfs \
  && cd fastdfs \
  && ./make.sh \
  && ./make.sh install \
  && rm -rf ${FASTDFS_PATH}/fastdfs

RUN useradd -m -s /bin/bash www

## comile nginx
# nginx url: https://freenginx.org/download/freenginx-${NGINX_VERSION}.tar.gz
# tengine url: http://tengine.taobao.org/download/tengine-${TENGINE_VERSION}.tar.gz
RUN git clone -b $FASTDFS_NGINX_MODULE_VERSION https://github.com/happyfish100/fastdfs-nginx-module.git fastdfs-nginx-module \
  && wget https://freenginx.org/download/freenginx-${FREENGINX_VERSION}.tar.gz \
  && tar -zxf freenginx-${FREENGINX_VERSION}.tar.gz \
  && cd freenginx-${FREENGINX_VERSION} \
  && ./configure --prefix=/usr/local/nginx \
      --user=www \
      --add-module=${FASTDFS_PATH}/fastdfs-nginx-module/src/ \
      --with-stream=dynamic \
  && make \
  && make install \
  && ln -s /usr/local/nginx/sbin/nginx /usr/bin/ \
  && rm -rf ${FASTDFS_PATH}/freenginx-* \
  && rm -rf ${FASTDFS_PATH}/fastdfs-nginx-module

EXPOSE 22122 23000 8080 8888 80
VOLUME ["$FASTDFS_BASE_PATH","/etc/fdfs","/usr/local/nginx/conf/conf.d"]

COPY conf/*.* /etc/fdfs/
COPY nginx_conf/ /nginx_conf/
COPY nginx_conf/nginx.conf /usr/local/nginx/conf/
COPY entrypoint.sh /usr/bin/

RUN chmod a+x /usr/bin/entrypoint.sh

WORKDIR ${FASTDFS_PATH}

ENTRYPOINT ["/usr/bin/entrypoint.sh"]
CMD ["tracker"]

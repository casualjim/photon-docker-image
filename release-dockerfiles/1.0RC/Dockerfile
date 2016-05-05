FROM photon:1.0TP2
MAINTAINER Fabio Rapposelli <fabio@vmware.com>

# Create temporary chroot environment
ENV TEMP_CHROOT /temp_chroot

RUN mkdir /data &&\
    mkdir $TEMP_CHROOT &&\
    mkdir -p $TEMP_CHROOT/var/lib/rpm &&\
    rpm --root $TEMP_CHROOT/ --initdb


# Add RC pkg source to tdnf/yum
COPY photon-rc.repo /etc/yum.repos.d/photon.repo

RUN tdnf upgrade -y tdnf &&\ 
    tdnf install -y tar xz &&\
    tdnf --installroot $TEMP_CHROOT/ install -y --nogpgcheck bash \
    coreutils filesystem findutils glibc grep photon-release photon-repos tdnf util-linux vim which &&\
    tdnf --installroot=$TEMP_CHROOT/ clean all

RUN cp /etc/resolv.conf $TEMP_CHROOT/etc/

# Cleanup
RUN cd $TEMP_CHROOT && rm -rf usr/src/ && rm -rf home/* && rm -rf var/log/*

# Build rootfs
RUN cd $TEMP_CHROOT && cp -pr etc/skel/. root/. && tar cpJf /data/rootfs.tar.xz .

RUN echo $'FROM scratch\n\
MAINTAINER Fabio Rapposelli <fabio@vmware.com>\n\
ADD rootfs.tar.xz /\n\
\n\
CMD ["/bin/bash"]' >> /data/Dockerfile

VOLUME /data

CMD [ "/usr/bin/tar", "-cf", "-", "-C", "/data", "." ]

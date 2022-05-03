FROM i386/debian

ENV DEBIAN_FRONTEND noninteractive

RUN apt update && \
    apt upgrade -y && \
    apt --yes --no-install-recommends install \
        linux-image-686 grub2 systemd initramfs-tools \
        python python3 python3-pip libc-l10n locales \
        libterm-readline-perl-perl \
        build-essential make cmake \
        unzip bzip2 xz-utils ca-certificates \
        strace file xterm vim nano apt-file \
        dhcpcd5 wget curl net-tools netcat \
    && \
    # this can save some startup time
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen && \
    # install 32 bit podman
    apt --yes --no-install-recommends install podman crun fuse-overlayfs && \
    mkdir -p ~/.config/containers && \
    echo 'unqualified-search-registries=["docker.io", "quay.io"]' > ~/.config/containers/registries.conf && \
    echo "alias podman='podman --storage-driver=vfs --events-backend=file'" >> ~/.bashrc && \
    podman pull --storage-driver=vfs --events-backend=file docker.io/hello-world && \
    # install 32 bit go
    wget https://go.dev/dl/go1.18.1.linux-386.tar.gz && \
    rm -rf /usr/local/go && tar -C /usr/local -xzf go*.linux-386.tar.gz && \
    rm go*.linux-386.tar.gz && \
    echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc && \
    export PATH=$PATH:/usr/local/go/bin && \
    go version && \
    # install 32 bit node
    wget https://unofficial-builds.nodejs.org/download/release/v16.9.1/node-v16.9.1-linux-x86.tar.gz && \
    tar -xf node-*-linux-x86.tar.gz && \
    rm node-*-linux-x86.tar.gz && \
    mv node-*-linux-x86 node && \
    echo "export PATH=$PATH:/node/bin" >> ~/.bashrc && \
    export PATH=$PATH:/node/bin && \
    npm install -g npm@* && \
    node --version && npm --version && \
    # change shell to bash and change root password
    chsh -s /bin/bash && \
    echo "root:root" | chpasswd && \
    # setup for serial access
    mkdir -p /etc/systemd/system/serial-getty@ttyS0.service.d/ && \
    systemctl enable serial-getty@ttyS0.service && \
    # disable "normal" services that aren't needed
    rm /lib/systemd/system/getty.target.wants/getty-static.service && \
    systemctl disable systemd-timesyncd.service && \
    systemctl disable apt-daily.timer && \
    systemctl disable apt-daily-upgrade.timer && \
    systemctl disable dhcpcd.service && \
    # make sure fstab is configured with tmpfs
    echo "tmpfs /tmp tmpfs nodev,nosuid 0 0" >> /etc/fstab && \
    # clean up as much as possible to reduce image space
    apt autoremove -y && \
    apt-get --yes clean && \
    rm -r /var/lib/apt/lists/* && \
    rm -r /usr/share/doc/* && \
    rm -r /usr/share/man/* && \
    rm -r /usr/share/locale/?? && \
    rm /var/log/*.log /var/log/lastlog /var/log/wtmp /var/log/apt/*.log /var/log/apt/*.xz

COPY getty-noclear.conf getty-override.conf /etc/systemd/system/getty@tty1.service.d/
COPY getty-autologin-serial.conf /etc/systemd/system/serial-getty@ttyS0.service.d/
COPY logind.conf /etc/systemd/logind.conf
COPY networking.sh /root/
COPY boot-9p /etc/initramfs-tools/scripts/boot-9p

# this needs to be commented out in order to boot from hdd
RUN printf '%s\n' 9p 9pnet 9pnet_virtio virtio virtio_ring virtio_pci | tee -a /etc/initramfs-tools/modules && \
    echo 'BOOT=boot-9p' | tee -a /etc/initramfs-tools/initramfs.conf && \
    update-initramfs -u

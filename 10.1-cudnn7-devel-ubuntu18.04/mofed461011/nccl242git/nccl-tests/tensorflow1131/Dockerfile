FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

ENV MOFED_VERSION=4.6-1.0.1.1
# Pick up some MOFED and TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        perl \
        lsb-release && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu18.04-x86_64.tgz && \
        tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu18.04-x86_64.tgz && \
        yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu18.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv && \
	dpkg -i MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu18.04-x86_64/DEBS/openmpi_*_all.deb && \
        cd .. && \
        rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu18.04-x86_64 && \
        rm -rf *.tgz && \
    rm -rf /var/lib/apt/lists/*

ENV NCCL_VERSION 2.4.2

# Install NCCL2 v2.4.2-1
RUN apt-get purge -y --allow-change-held-packages \
        "*nccl*" && \
        apt-get update && apt-get install -y --no-install-recommends \
        git \
        devscripts && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
        git clone https://github.com/NVIDIA/nccl.git && \
        cd nccl && git reset --hard v$NCCL_VERSION-1 && \
	make pkg.debian.build -j && \
        dpkg -i build/pkg/deb/libnccl*.deb && \
        cd .. && \
        rm -rf nccl 

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

# Download and install nccl-tests
RUN mkdir /nccl-tests -p && cd /nccl-tests && \
    git clone https://github.com/NVIDIA/nccl-tests.git . && \
    make MPI=1 NCCL_HOME=/usr/lib/x86_64-linux-gnu CUDA_HOME=/usr/local/cuda-10.1 MPI_HOME=/usr/mpi/gcc/openmpi-4.0.2a1/ DST_DIR=/ompi4tests/NCCL2 -j $(nproc) && \
    cd / && rm -fr /nccl-tests

RUN apt-get update && apt-get install -y --no-install-recommends \
        python-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip install setuptools wheel && pip install tensorflow-gpu==1.13.1

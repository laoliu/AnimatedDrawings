# syntax = docker/dockerfile:1.2

FROM continuumio/miniconda3:24.1.2-0

# install os dependencies
RUN mkdir -p /usr/share/man/man1
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    ca-certificates \
    curl \
    vim \
    sudo \
    default-jre \
    git \
    gcc \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

RUN conda install python=3.8.13 -y


# install python dependencies
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
RUN pip install openmim
ENV MMCV_WITH_OPS=1
ENV FORCE_CUDA=0
RUN pip install torch==2.1.2 torchvision==0.16.2 --index-url https://download.pytorch.org/whl/cpu
# 使用预编译的 mmcv wheel 包，避免编译问题
RUN pip install mmcv==2.1.0 -f https://download.openmmlab.com/mmcv/dist/cpu/torch2.1/index.html
RUN pip install mmdet==3.2.0
RUN pip install torchserve

# bugfix for xtcocoapi, an mmpose dependency
RUN git clone https://github.com/jin-s13/xtcocoapi.git
WORKDIR /xtcocoapi
RUN pip install -r requirements.txt
RUN python setup.py install
WORKDIR /
RUN pip install mmpose==1.0.0
RUN pip install numpy==1.24.4  #solve numpy version problem

# prep torchserve
ARG HTTP_PROXY=http://host.docker.internal:7890
ARG HTTPS_PROXY=http://host.docker.internal:7890
RUN mkdir -p /home/torchserve/model-store
RUN http_proxy=${HTTP_PROXY} https_proxy=${HTTPS_PROXY} wget https://github.com/facebookresearch/AnimatedDrawings/releases/download/v0.0.1/drawn_humanoid_detector.mar -P /home/torchserve/model-store/
RUN http_proxy=${HTTP_PROXY} https_proxy=${HTTPS_PROXY} wget https://github.com/facebookresearch/AnimatedDrawings/releases/download/v0.0.1/drawn_humanoid_pose_estimator.mar -P /home/torchserve/model-store/
COPY config.properties /home/torchserve/config.properties

# Force CPU usage
ENV CUDA_VISIBLE_DEVICES=""

# starting command
CMD /opt/conda/bin/torchserve --start --disable-token-auth --ts-config /home/torchserve/config.properties && sleep infinity

FROM continuumio/miniconda3:24.1.2-0

# 定义构建参数用于代理设置
ARG HTTP_PROXY=""
ARG HTTPS_PROXY=""

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
RUN pip install torch==1.13.1 torchvision==0.14.1 --index-url https://download.pytorch.org/whl/cpu
# 使用与mmpose 0.29.0兼容的mmcv版本
RUN pip install mmcv-full==1.7.0 -f https://download.openmmlab.com/mmcv/dist/cpu/torch1.13/index.html
RUN pip install mmdet==2.28.2
RUN pip install torchserve

# bugfix for xtcocoapi, an mmpose dependency
# 使用代理克隆xtcocoapi，如果代理不可用则直接尝试
RUN if [ -n "$HTTP_PROXY" ]; then \
        git config --global http.proxy $HTTP_PROXY && \
        git config --global https.proxy $HTTPS_PROXY; \
    fi && \
    git clone https://github.com/jin-s13/xtcocoapi.git && \
    git config --global --unset http.proxy || true && \
    git config --global --unset https.proxy || true
WORKDIR /xtcocoapi
RUN pip install -r requirements.txt
RUN python setup.py install
WORKDIR /
RUN pip install mmpose==0.29.0
RUN pip install numpy==1.24.4  #solve numpy version problem

# prep torchserve
RUN mkdir -p /home/torchserve/model-store
COPY drawn_humanoid_detector.mar /home/torchserve/model-store/
COPY drawn_humanoid_pose_estimator.mar /home/torchserve/model-store/
COPY config.properties /home/torchserve/config.properties

# Force CPU usage
ENV CUDA_VISIBLE_DEVICES=""
ENV FORCE_CPU=1

# starting command
CMD /opt/conda/bin/torchserve --start --disable-token-auth --ts-config /home/torchserve/config.properties && sleep infinity

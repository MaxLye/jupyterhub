FROM nvidia/cuda

LABEL maintainer="ahkui <ahkui@outlook.com>"

# Pick up some TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cuda-command-line-tools-10-0 \
        cuda-cublas-10-0 \
        cuda-cufft-10-0 \
        cuda-curand-10-0 \
        cuda-cusolver-10-0 \
        cuda-cusparse-10-0 \
        curl \
        wget \
        libcudnn7=7.3.1.20-1+cuda10.0 \
        libnccl2=2.3.5-2+cuda10.0 \
        libfreetype6-dev \
        libhdf5-serial-dev \
        libpng-dev \
        libzmq3-dev \
        pkg-config \
        python \
        python-dev \
        python3 \
        python3-dev \
        rsync \
        software-properties-common \
        unzip \
        && \
        apt-get install nvinfer-runtime-trt-repo-ubuntu1804-5.0.0-rc-cuda10.0 && \
        apt-get update && \
        apt-get install libnvinfer5=5.0.0-1+cuda10.0 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.10.0/tini && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
    python3 get-pip.py && \
    python2 get-pip.py && \
    rm get-pip.py

ARG PIP="Pillow h5py ipykernel jupyter notebook keras keras_applications keras_preprocessing matplotlib numpy pandas scipy sklearn Flask gunicorn pymongo redis requests ipyparallel bs4"
RUN python2 -m pip --no-cache-dir install \
        ${PIP} && \
    python3 -m pip --no-cache-dir install \
        jupyterhub \
        ${PIP} && \
    python2 -m ipykernel.kernelspec && \
    python3 -m ipykernel.kernelspec

RUN if [ ${JUPYTERHUB_ENABLE_NVIDIA} = true ]; then \
    # run the install
    python2 -m pip --no-cache-dir install \
        tensorflow-gpu \
    && \
    python3 -m pip --no-cache-dir install \
        tensorflow-gpu \
;else \
    python2 -m pip --no-cache-dir install \
        tensorflow \
    && \
    python3 -m pip --no-cache-dir install \
        tensorflow \
;fi

RUN ln -s -f /usr/bin/python3 /usr/bin/python

COPY jupyter_notebook_config.py /root/.jupyter/

COPY notebooks /notebooks

COPY run_jupyter.sh /

ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888

WORKDIR "/notebooks"

COPY start.sh /usr/local/bin/
COPY start-notebook.sh /usr/local/bin/
COPY start-singleuser.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/start.sh
RUN chmod +x /usr/local/bin/start-notebook.sh
RUN chmod +x /usr/local/bin/start-singleuser.sh

ENTRYPOINT ["tini", "--"]

CMD ["start-notebook.sh"]
#CMD ["/run_jupyter.sh", "--allow-root"]
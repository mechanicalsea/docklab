# docklab
Dockerfile for groups

```
- dockerlab
| - dockerfile1: SpeechLab
| - dockerfile2: RecSysLab
| - dockerfile3: Speechlab-JupyterLab-Cuda10.0
```

## Base Image: SpeechLab

```yaml
language:
  - python
version:
  python: 3.7.6
  pip: 20.0.1
deeplearning framework:
  torch: 1.4.0
  tensorflow-gpu: 2.1.0
port:
  ssh: 22
  tensorboard: 6006
  jupyter lab: 8888
service:
  ssh: root@ip -p port
  jupyter lab: ip:port/?token=...
workdir:
  - /workspace
recommend pypi source:
  - https://pypi.tuna.tsinghua.edu.cn/simple
ssh:
  default password: 422
docker relative:
  build: docker build -t speechlab .
  run: docker run -d -p 9000:22 --gpus '"device=2"' speechlab
```

## Current Dockerfile

Dockerfile: [Speechlab-JupyterLab-Cuda10.0](./docklab/Speechlab-JupyterLab-Cuda10.0)

- Nodejs 12.x
- Jupyterlab + extension
  - git
  - ploty
  - markup
  - lsp
  - manager
  - drawio
  - bokeh
  - spreadsheet
- Cuda:10.0

```dockerfile
From nvidia/cuda:10.0-devel-ubuntu18.04
MAINTAINER Wang Rui <rwang@tong.edu.cn>
ENV DEBIAN_FRONTEND=noninteractive
RUN rm /etc/apt/sources.list.d/* && apt-get update && apt-get -y upgrade && apt-get install -y software-properties-common unzip zip wget git && add-apt-repository -y ppa:deadsnakes/ppa
# Add ssh on port 22
RUN apt-get update && apt-get install -y curl openssh-server net-tools iputils-ping
RUN echo 'root:1' | chpasswd && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && sed -i 's/UsePAM yes/UsePAM no/' /etc/ssh/sshd_config
# Install nodejs
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - && apt-get install -y nodejs && rm -rf /var/lib/apt/lists/*
# Add python 3.7, pip, and library
RUN apt-get update && apt-get install -y python3.7 python3-setuptools python3.7-dev libblas-dev libatlas-base-dev build-essential libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev libsndfile1 automake autoconf sox gfortran libtool subversion libsox-dev libsox-fmt-all openjdk-8-jdk virtualenv swig
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
RUN python3.7 get-pip.py && rm get-pip.py
RUN rm /usr/bin/python && ln -s /usr/bin/python3.7 /usr/bin/python && ln -s /usr/bin/pip3 /usr/bin/pip
RUN pip --no-cache-dir install -i https://pypi.tuna.tsinghua.edu.cn/simple h5py matplotlib numpy scipy sklearn pandas future
# Install bazel
RUN apt-get -y install openjdk-8-jdk && echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list && curl https://bazel.build/bazel-release.pub.gpg | apt-key add - && apt-get update && apt-get -y install bazel && apt-get -y upgrade bazel
# Install jupyterlab
RUN pip install --upgrade pip && pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade jupyterlab==2.2.0 ipywidgets plotly bokeh numexpr patsy ipython sympy nose jupyterlab-git && jupyter labextension install @jupyter-widgets/jupyterlab-manager jupyterlab-drawio jupyterlab-plotly @bokeh/jupyter_bokeh @jupyterlab/git jupyterlab-spreadsheet @jupyterlab/toc @agoose77/jupyterlab-markup
# Install pytorch
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip install torch==1.7.1+cu92 torchvision==0.8.2+cu92 torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple librosa seaborn scikit-learn SoundFile sounddevice tqdm blockdiag pyyaml tensorboard thop gpustat
# Read to go
RUN mkdir /workspace && apt-get clean && apt-get autoclean && rm -rf /var/lib/apt/lists/*
WORKDIR /workspace
EXPOSE 22 6006 8888
CMD ["/bin/bash", "-c", "/etc/init.d/ssh restart && jupyter lab --ip=0.0.0.0 --allow-root --no-browser --port=8888"]
```

## WARNING

- debconf: delaying package configuration, since apt-utils is not installed
- WARNING: The directory '/root/.cache/pip' or its parent directory is not owned or is not writable by the current user. The cache has been disabled. Check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.

## ERROR

- E: GPG error: https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu1804/x86_64 Release: Signed file isn't valid, got 'NODATA' (does the network require authentication?)
  
  - Try Again Woks.
  
- E: The repository 'https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 Release' is not signed.
  
  - Try Again Works
  
- error: command 'x86_64-linux-gnu-gcc' failed
  
  - apt-get install -y python-dev/python3-dev/python3.7-dev (select a version corresponding to python)
  
- WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED
  - ssh verification error，modify client ~/.ssh/known_hosts
  - a simple way：delete content corresponding to specific ip:port in ~/.ssh/known_hosts
  
- Error: CUDA Source Downloading：

  - ```
    E: Failed to fetch https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu1804/x86_64/Packages.gz File has unexpected size (132475 != 140969). Mirror sync in progress? [IP: 119.90.56.230 443]
    
      Hashes of expected file:
    
      - Filesize:140969 [weak]
    
      - SHA256:9d931d646cab6a4eef0611967d29063db26b3b61485957e07bfc09b4af695ce8
    
      - SHA1:68fe75a7c159ea88b3f3d55c23c8543cce056e47 [weak]
    
      - MD5Sum:5721c0752c247920ae60c5b0db38509e [weak]
    
      Release file created at: Fri, 28 Feb 2020 22:31:40 +0000
    
    E: Some index files failed to download. They have been ignored, or old ones used instead.
    ```

  - A simple way: delete cuda source, e.g., `rm /etc/apt/sources.list.d/*`

## TIMEOUT

- Tensorflow-gpu
  - Try Again Works


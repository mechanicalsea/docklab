# docklab
Dockerfile for groups

## Image 1: SpeechLab

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

## TIMEOUT

- Tensorflow-gpu
  - Try Again Works


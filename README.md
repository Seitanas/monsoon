# Monsoon - main repository

This repository contains 

1. SONIC exporter

    * `$ docker run --name sonic_monitoring --network=host --pid=host --privileged --restart=always -d -e REDIS_AUTH=$(cat /run/redis/auth/passwd) -v /var/run/redis:/var/run/redis -v /usr/bin/vtysh:/usr/bin/vtysh -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock  stordis/sonic_monitoring:latest`

<!-- 2. Node Exporter
    * `$ docker run --name node-exporter --network=host --pid=host --privileged --restart=always -v /proc:/host/proc:ro -v /sys:/host/sys:ro -v /:/rootfs:ro prom/node-exporter:v1.3.0` -->


## Details:

1. src/ folder has below subfolders
- sonic_exporter - this folder contains - exporter script
- sonic-py-swsssdk - this is a git submodule pulled from [github](https://github.com/Azure/sonic-py-swsssdk) as a redis connector used by python_exporter.

2. Dockerfile - This file can be used to build docker image.


## Building

```console
$ export VERSION="0.1.2"
$ podman build -t sonic_monitoring:${VERSION} .
[1/2] STEP 1/7: FROM python:3.10-bullseye
Resolving "python" using unqualified-search registries (/etc/containers/registries.conf)
Trying to pull docker.io/library/python:3.10-bullseye...
Getting image source signatures
Copying blob 461bb1d8c517 done  
Copying blob 724cfd2dc19b done  
Copying blob e6d3e61f7a50 done  
Copying blob 412caad352a3 done  
Copying blob 808edda3c2e8 done  
Copying blob 0c6b8ff8c37e done  
Copying blob 8bd4965a24ab done  
Copying blob fccd5fa208a8 done  
Copying blob af1ca64a0eec done  
Copying config e2e732b795 done  
Writing manifest to image destination
Storing signatures
[1/2] STEP 2/7: COPY . .
--> b1bb5809caf
[1/2] STEP 3/7: RUN pip3 install poetry
--> 1f58ca76511
[1/2] STEP 4/7: RUN poetry export -f requirements.txt -o /home/requirements.txt
--> aafc87b688d
[1/2] STEP 5/7: RUN cd src/sonic-py-swsssdk && python setup.py build sdist && cd ../..
--> b056c2dad7e
[1/2] STEP 6/7: RUN poetry build
--> 7a0b271d272
[1/2] STEP 7/7: RUN cp dist/sonic_exporter*.tar.gz /home/ && cp src/sonic-py-swsssdk/dist/swsssdk-*.tar.gz /home
--> c1af04863c0
[2/2] STEP 1/5: FROM python:3.10-slim-bullseye
[2/2] STEP 2/5: COPY --from=0 /home/requirements.txt /home/requirements.txt
--> 9d98422dfc2
[2/2] STEP 3/5: COPY --from=0 /home/*.tar.gz /home/
--> bfdf47151b8
[2/2] STEP 4/5: RUN pip3 install --pre -r /home/requirements.txt && pip3 install /home/*.tar.gz && mkdir -p /src
--> a9de129245a
[2/2] STEP 5/5: CMD sonic_exporter
[2/2] COMMIT sonic_monitoring:0.1.2
--> f71e7b8de82
Successfully tagged localhost/sonic_monitoring:0.1.2
f71e7b8de82e5eabfe66c803538f19d1fb3c44b3b0edf9725e9eb61943d4a093
$ podman save --format docker-archive localhost/sonic_monitoring:${VERSION} | gzip  > sonic-monitoring_${VERSION}.tar.gz
$ ls
sonic-monitoring_0.1.2.tar.gz
```

## Loading the image on a switch

```console
$ docker load -i sonic-monitoring_${VERSION}.tar.gz
# 编译starrocks

- 源码地址: /datadisk/starrocks-build/starrocks
- build env docker image: starrocks/dev-env-ubuntu:3.2-latest

mkdir -p /datadisk/starrocks-build/.m2

docker run -it -d \
  -v /datadisk/starrocks-build/.m2:/root/.m2 \
  -v /datadisk/starrocks-build/starrocks:/root/starrocks \
  --name starrocks-3.2 \
  starrocks/dev-env-ubuntu:3.2-latest

docker exec -it starrocks-3.2 /bin/bash

## start build
cd /root/starrocks
./build.sh

or

export PARALLEL=4
./build.sh

## 编译结果
/datadisk/starrocks-build/starrocks/output/
├── fe/
└── be/

# 打all-in-one镜像

cd /datadisk/starrocks-build/starrocks

## 编译 broker
cd fs_brokers/apache_hdfs_broker/
./build.sh

## 返回根目录重新构建镜像
cd /datadisk/starrocks-build/starrocks

DOCKER_BUILDKIT=1 docker build \
  --build-arg ARTIFACT_SOURCE=local \
  --build-arg LOCAL_REPO_PATH=. \
  -f docker/dockerfiles/allin1/allin1-ubuntu.Dockerfile \
  -t starrocks-allin1-ubuntu:custom-3.2.16 \
  .

## 运行all-in-one
docker run -d \
  --name starrocks-test \
  -p 19031:9030 \
  -p 18031:8030 \
  -p 18041:8040 \
  --cpus=2 \
  --memory=8g \
  starrocks-allin1-ubuntu:custom-3.2.16


# 构建be镜像	
cd /datadisk/starrocks-build/starrocks

DOCKER_BUILDKIT=1 docker build \
  --build-arg ARTIFACT_SOURCE=local \
  --build-arg LOCAL_REPO_PATH=. \
  -f docker/dockerfiles/be/be-ubuntu.Dockerfile \
  -t starrocks-be-ubuntu:custom-3.2.16 \
  .

## push到阿里云
docker login username addr 
docker tag starrocks-be:custom your-repo:tag
docker push your-repo:tag


## 本地运行
docker run -it --rm \
--name starrocks-be-test \
--hostname starrocks-be-test \
--privileged \
your-repo:tag \
/bin/bash

cd /opt/starrocks/be
./bin/start_be.sh --daemon

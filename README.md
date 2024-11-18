# 前言
参考教程：[https://blog.gismore.com/archives/install-harbor-on-arm64](https://blog.gismore.com/archives/install-harbor-on-arm64)

github：[https://github.com/bitnami/containers](https://github.com/bitnami/containers)

github加速网站：[https://github.geekery.cn](https://github.geekery.cn) （来源网络）

docker hub镜像网站：[https://docker.fxxk.dedyn.io](https://docker.fxxk.dedyn.io)（来源网络）
# 1. 环境信息
```shell
系统版本：Kylin Linux Advanced Server V10 (Tercel)
CPU型号：Kunpeng-920
docker版本：18.09
docker-compose版本：1.22.0
安装harbor版本 bitnami/2.12-rc1
```
# 2. 下载配置文件
```shell
wget https://github.com/bitnami/containers/archive/refs/heads/main.zip
unzip -q containers-main.zip
mkdir bitnami-harbor
mv containers-main/bitnami/harbor-portal/2/debian-12/config/ bitnami-harbor/
mv containers-main/bitnami/harbor-portal/2/debian-12/docker-compose.yml bitnami-harbor/
rm -rf containers-main
```
# 3. 上传镜像
获取镜像可参考[[利用github下载镜像]]
```shell
docker load -i bitnami-harbor-arm64.tar
```
# 4. 获取redis配置
```shell
mkdir -p bitnami-harbor/config/redis
# 获取redis配置文件
docker run -d --rm --name=redis111 bitnami/redis:7.4 bash -c "sleep 120"
docker cp redis111:/opt/bitnami/redis/etc/redis.conf /opt/bitnami-harbor/config/redis/redis.conf

cd bitnami-harbor
vi config/redis/redis.conf
# 编辑redis.conf,取消redis.conf中如下内容的注释
ignore-warnings ARM64-COW-BUG
```
# 5. 修改配置
```shell
# 修改 docker-compose
vi docker-compose

# 首行添加如下内容
version: '3'

# 删除如下内容
volumes:
  registry_data:
    driver: local
  core_data:
    driver: local
  jobservice_data:
    driver: local
  postgresql_data:
    driver: local

# 修改以下内容
[root@harbor bitnami-harbor]# grep data: docker-compose.yml 
      - registry_data:/storage
      - registry_data:/storage
      - postgresql_data:/bitnami/postgresql
      - core_data:/data
      - jobservice_data:/var/log/jobs
为
[root@harbor bitnami-harbor]# grep data: docker-compose.yml 
      - ./data/registry_data:/storage
      - ./data/registry_data:/storage
      - ./data/postgresql_data:/bitnami/postgresql
      - ./data/core_data:/data
      - ./data/jobservice_data:/var/log/jobs

# redis部分添加volumes部分
  redis:
    image: docker.io/bitnami/redis:7.4
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
    # 添加如下两行
    volumes:
      - ./config/redis/redis.conf:/opt/bitnami/redis/etc/redis.conf

# 修改core部分
	  # 修改docker访问地址
      - EXT_ENDPOINT=http://193.169.22.28:83
      # 修改admin默认密码，默认为bitnami
      - HARBOR_ADMIN_PASSWORD=bitnami
      
```
# 6. 启动harbor
```shell
mkdir -p data/{registry_data,postgresql_data,core_data,jobservice_data}

chown -R 1001:1001 config
chown -R 1001:1001 data

docker-compose up -d
```

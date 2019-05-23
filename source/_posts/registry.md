---
title: docker.registry
date: 2019-05-10 15:00:09
tags: docker
---

# 安装

```
docker pull registry
```

<!---more-->

# 运行

```
docker run -d -p 5000:5000 -v $PWD/myregistry:/var/lib/registry registry
```

# 测试

```
docker tag hello-world 192.168.119.148:5000/test  # 复制
docker push 192.168.119.148:5000/test  # 提交
docker rmi 192.168.119.148:5000/test   # 删除原有
docker pull 192.168.119.148:5000/test  # 拉取
```

# 查看

```
curl http://192.168.2.193:5000/v2/_catalog
```


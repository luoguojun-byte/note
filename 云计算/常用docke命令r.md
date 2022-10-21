#  Docker

## 镜像命令

``` shell
docker images 查看本地运行的镜像
 Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
  
  -a 显示所有
  -q 只显示ID
  
  docker search 搜索镜像
  --filter=STARS=3000 显示大于3000的文件，用来过滤
  docker rmi 镜像ID 删除镜像
  docker rmi -f $(docker images -aq) 删除查出来的镜像
  docker inspect 容器ID 查看容器的元数据
```



## 容器命令

```  shell
docker pull 软件名 下载镜像
docker run [可选参数] image
	--name 容器名字
	-d 后台运行
	-it 交互方式运行
	-p 指定容器的端口
		-p 主机端口：映射的容器端口 8080:8080
		-p 容器端口
		-p ip:
docker run -it centos /bin/bash 启动并且进入容器
docker rm  容器ID 删除容器
ocker rm -f $(docker ps -aq) 删除查出来的容器
docker ps 列出正在运行的容器
	-a 列出正在运行和历史运行
	-n=? 显示最近创建的容器
	-q 容器ID	
	
docker start 启动
docker stop  停止
docker kill  强制停止
docker exec 容器ID 进入容器
 
```


# docker 安装 FastDFS

镜像包含nginx，配置文件位置：/etc/fdfs
```shell
sudo docker image pull delron/fastdfs

sudo docker run -d --name tracker --network=host -v /data/fastdfs/tracker:/var/fdfs delron/fastdfs tracker

sudo docker run -d --name storage --network=host -e TRACKER_SERVER=47.97.8.7:22122 -v /data/fastdfs/storage:/var/fdfs delron/fastdfs storage
```


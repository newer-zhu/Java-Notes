## Dockerfile容器编排

### 命令

1. FROM 基于哪个镜像构建
2. MAINTAINER 维护者名字
3. RUN 构建时命令，再docker build时执行
4. EXPOSE  暴露端口
5. WORKDIR 工作目录，终端 -it 进入的默认落地点
6. USER 以什么用户身份执行，默认root
7. COPY 将宿主机目录下文件的内容复制到镜像目录
8. ADD 宿主机目录下文件拷贝进镜像，自动处理url和tar压缩包
9. VOLUMNE 允许挂载的目录
10. ENV key value 环境变量
11. ENTRYPOINT 容器启动时指令，命令行的参数会传给ENTRYPOINT 指令指定的程序。 如果有了此命令，CMD会变成参数，如<ENTRYPOINT> "<CMD>"
12. CMD 默认参数，启动后执行命令。会被docker run之后的命令替换


## Docker

![image-20231203173946943](D:\typora_img\image-20231203173946943.png)

### Image

类似一个静态的文件系统（虚拟机的快照 snapshot），提供了运行时所需的

- 程序
- 库
- 资源
- 配置

#### Build a image

> from [docker docs]([Containerize an application | Docker Docs](https://docs.docker.com/get-started/02_our_app/))
>
> Github repo: [getting-started-app]([docker/getting-started-app: A simple application for the getting started guide in Docker's documentation (github.com)](https://github.com/docker/getting-started-app/tree/main))

Create a `Dockerfile` at `/path/to/getting-started-app`

```dockerfile
# syntax=docker/dockerfile:1

FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

Build the image in the terminal:

```shell
cd /path/to/getting-started-app

docker build -t getting-started .
```

> - `-t`: tag your container name, in this case: getting-started
> - `.`: the path where Docker should find the `Dockerfile`



### Container

Run your container using `docker run`:

```dockerfile
docker run -dp 127.0.0.1:3000:3000 getting-started
```

> - `-d`: `--detach` runs in the background
> - `-p`: `--publish` takes a string value in the format of `HOST:CONTAINER`
>     - `HOST`: the address on the host, in this case: `127.0.0.1:3000`
>     - `CONTAINER`: the port on the container, in this case: `3000`

then you open web browser to `http://localhost:3000` to access the app service

#### Filesystem

> - Each container has its own "scratch space" to create/update/remove files.
> - Other containerse won't see the changes

Start an `ubuntu` container that will create a file named `/data.txt` with a random number between 1 and 10000:

```shell
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

in CLI, use the `docker exec` command to access the container (use `docker ps` to get the container's ID):

```shell
docker exec <container-id> cat /data.txt
```

But in another container using same image, you can't see the `data.txt`

#### Volumes

> man page: [volumes]([Volumes | Docker Docs](https://docs.docker.com/storage/volumes/))
>
> [Volumes](https://docs.docker.com/storage/volumes/) provide the ability to connect specific filesystem paths of the container back to the host machine. If you mount a directory in the container, changes in that directory are also seen on the host machine. If you mount that same directory across container restarts, you'd see the same files.

![image-20231203180538210](D:\typora_img\image-20231203180538210.png)

##### Mount

Create a volume and start the container

```shell
docker volume create todo-db
```

> the specific location of volume on the host machine is implicitly set by the docker

Start the todo app container, but add the `--mount` option to specify a volume mount

```shell
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

> - `src`: the volume you want to mount
> - `target`: the file path in the container

Delve deeper to see where docker storing my data using volume

```shell
$ docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

> `Mountpoint`：the actual location of the data in the disk

##### Bind

Comparison with Mount

|                     | Mount                                                   | Bind                                                 |
| ------------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| Host location       | Docker chooses                                          | You chooses                                          |
| params in `--mount` | `type=volume,src=my-volume-name,target=/usr/local/data` | `type=bind,src=/path/to/data,target=/usr/local/data` |



### Repository



## Docker Compose


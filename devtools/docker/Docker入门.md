# Docker入门

#### Docker是什么？

#### Docker能做什么？

#### Docker的好处是什么？

#### Docker的使用

* ##### Docker下载

* ##### Docker安装zookeeper

  * docker拉取zookeeper

    ```shell
    sunwj@sunwjdeMacBook-Pro  ~/Documents/GitHub/EverydayLeetcode/gitbook_leetcode   main ±  docker pull zookeeper
    Using default tag: latest
    latest: Pulling from library/zookeeper
    f7ec5a41d630: Pull complete
    faf4c47c8c61: Pull complete
    810072571faf: Pull complete
    ca2026cde8de: Pull complete
    560b60c59d86: Pull complete
    48a7bbbfc8eb: Pull complete
    56ff45ef75e6: Pull complete
    e28096689586: Pull complete
    Digest: sha256:7b598403a79fddd39043702e401a3b6b976d670e442009978c526490c2ff904f
    Status: Downloaded newer image for zookeeper:latest
    docker.io/library/zookeeper:latest
    ```

  * 运行docker下的zookeeper

    ```
    docker run -d --name zookeeper -p 2181:2181 zookeeper
    ```

  * 查看docker下运行的进程

    ```
    sunwj@sunwjdeMacBook-Pro  ~/Documents/GitHub/JavaGitBook   main ±  docker ps
    CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                                                                     NAMES
    76457936aa08   zookeeper   "/docker-entrypoint.…"   27 seconds ago   Up 25 seconds   2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, :::2181->2181/tcp, 8080/tcp   zookeeper
    ```

  * 根据ps查询出来的容器id，可以用exec 命令进入zookeeper控制台

    ```
     sunwj@sunwjdeMacBook-Pro  ~/Documents/GitHub/JavaGitBook   main ±  docker exec -it 76457936aa08 zkCli.sh
    ```

    


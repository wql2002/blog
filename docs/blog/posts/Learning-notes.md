---
date:
  created: 2024-07-16
  updated: 2024-07-19
readtime: 5
---

My daily learning notes, including all aspects.
<!-- more -->

## 7.16.2024

### WSL2

Windows下的虚拟化方案

- [使用 vmware 还是 docker 来搭建 Linux 实验环境？](https://www.v2ex.com/t/812956)
- [[WSL 2] Multi WSL2 distributions use the same network namespace](https://github.com/microsoft/WSL/issues/4304)
- [无图形界面的 Linux 能装虚拟机么？](https://www.v2ex.com/t/142002)



### Linux内核格式与启动协议

https://jia.je/os/2023/10/01/linux-boot-protocol/#linuxx86-boot-protocol



### Docker

- [How is Docker different from a virtual machine?](https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-virtual-machine)

- [From inside of a Docker container, how do I connect to the localhost of the machine?](https://stackoverflow.com/questions/24319662/from-inside-of-a-docker-container-how-do-i-connect-to-the-localhost-of-the-mach)

- [How do I get into a Docker container's shell?](https://stackoverflow.com/questions/30172605/how-do-i-get-into-a-docker-containers-shell)




## 7.17.2024

### www

https://stackoverflow.com/questions/413490/what-is-the-point-of-www-in-web-urls



### Materials for MkDocs

udpate my homepage build kits

https://squidfunk.github.io/mkdocs-material/getting-started/#with-pip-latest



### 系统底层

V2EX讨论：https://www.v2ex.com/t/282778

Stanford Embedded OS: https://github.com/dddrrreee/cs140e-20win/tree/master/docs

network debugging: https://www.manjusaka.blog/posts/2024/05/11/where-are-my-package/#%E6%9C%80%E5%90%8E



### `WSLg`

https://silaoa.github.io/2021/2021-06-02-WSLg%EF%BC%9A%E4%B8%BAWSL%E5%A2%9E%E5%85%89%E6%B7%BB%E5%BD%A9.html



### git

change remote:

- official https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories
- https://stackoverflow.com/questions/18801147/changing-the-git-remote-push-to-default
- https://stackoverflow.com/questions/2432764/how-do-i-change-the-uri-url-for-a-remote-git-repository



### pip换源

https://juejin.cn/post/6988116489739960334



## 7.18.2024

### zsh autocompletion

https://gist.github.com/n1snt/454b879b8f0b7995740ae04c5fb5b7df



## 7.19.2024

### rust

`mut` in function parameter: 

- https://users.rust-lang.org/t/function-parameter-v-mut-t-or-mut-v-t/62410
- https://stackoverflow.com/questions/28587698/whats-the-difference-between-placing-mut-before-a-variable-name-and-after-the

> as [constant pointer vs pointer to constant](https://stackoverflow.com/questions/21476869/constant-pointer-vs-pointer-to-constant)

```rust
// Rust          C/C++
    a: &T     == const T* const a; // can't mutate either
mut a: &T     == const T* a;       // can't mutate what is pointed to
    a: &mut T == T* const a;       // can't mutate pointer
mut a: &mut T == T* a;             // can mutate both
```



`type` and `pattern`: https://users.rust-lang.org/t/function-parameter-v-mut-t-or-mut-v-t/62410



### Cloud Infra

A bottom-up approach by Nick Cao@TUNA: https://www.youtube.com/watch?v=-lrkJOXAqoY

#### restic

back up: https://restic.readthedocs.io/en/stable/010_introduction.html

#### knot

authoritative DNS: https://www.knot-dns.cz/



## 7.21.2024

### check alias

https://askubuntu.com/questions/102093/how-to-see-the-command-attached-to-a-bash-alias



### fs in linux

#### mount

mount a fs/partition (typically one partition has one fs, so used interchangeably): https://askubuntu.com/questions/20680/what-does-it-mean-to-mount-something

#### device

In Unix world, *everything is a file*. So device is stored at `/dev`.

https://askubuntu.com/questions/20680/what-does-it-mean-to-mount-something

- `lsblk`: check disk(/partition) info
- `df -h`: check fs-partition info



### project build

`./configure`, `make`, and `make install`: https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install



## 7.22.2024

### python shebang

`#!/usr/bin/env python` vs `#!\usr/local/bin/python`

https://mail.python.org/pipermail/tutor/2007-June/054816.html

https://stackoverflow.com/questions/1352922/why-is-usr-bin-env-python-supposedly-more-correct-than-just-usr-bin-pyt

TL; DR: The former will search `$PATH` for the actual place python installed, while the latter will use the path specified and fail if not find.




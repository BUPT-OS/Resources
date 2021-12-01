# RTL

## 基本介绍

RTL是Linux内核的一个下游分支，致力于增强Linux内核的实时性。目前Linux基金会成立了一个项目小组（The RTL Collaborative Project），目的是将RTL这个分支的代码提交到上游主线分支。

## RTL社区运作规则

1. RTL社区会维持一个和Linux主线分支保持一致的git仓库，在保证和Linux主线同步的情况下，改进Linux内核的实时性，这个仓库就是RTL的[dev分支](http://git.kernel.org/cgit/linux/kernel/git/rt/linux-rt-devel.git)。每当Linux主线移动到下一个稳定版本，RTL的开发也就随之移动到下一个版本，之前版本就会被移动到另一个git仓库中，作为RTL的[stable分支](http://git.kernel.org/cgit/linux/kernel/git/rt/linux-stable-rt.git)。

2. RTL还会定期发布[补丁集](https://cdn.kernel.org/pub/linux/kernel/projects/rt/)，可以直接打在对应版本的Linux内核中，使之成为RTL。在2.6.11后对Linux内核的每一个LTS版本一定会出一个补丁集，使用者可以自行下载对应版本的补丁集，然后打到Linux内核中；实际情况中，RTL的补丁集更为密集，几乎是每个Linux稳定版本都会出一个补丁集，然后针对这个稳定版本的每一个更新都会有补丁集。
   - RTL stable分支的更新有两个情况，一个是Linux的stable分支发生了变化，另一个是RTL的主线分支的补丁回合。
   - RTL stable的分支命名规则如下
     - 在Linux stable分支的后面加上-rt，-rt计数器作为一个标识符，区分不同的版本，在以下情况会发生更新
       - Linux的stable分支发生更新
       - Linux的stable分支更新造成和现有的RTL主线版本冲突
       - 回合RTL主线补丁
     - 根据上面的规则，某一个Linux的stable分支更新后，会产生两个-rt版本的tag
   - RTL也有LTS分支，RTL的[LTS分支](https://www.kernel.org/releases.html)的选择和维护时间与Linux的[LTS分支](https://www.kernel.org/releases.html)保持一致。

## 搭建开发环境

1. 获取RTL源码的方式有两种：直接从RTL的代码树中获取，或者使用RTL发布的补丁集外加Linux的一个stable的release，自行生成RTL。两种方式各有优劣
   - 直接需要下载整个git仓库，成本比较高；但是可以直接看到整个仓库的情况；

        ```shell
        git clone git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git
        ```

   - 自行生成RTL需要选择对应版本，并且自己打补丁，比较麻烦；但是胜在比较灵活；

2. 搭建qemu等必要工具

## 开发实时应用
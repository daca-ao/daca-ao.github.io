---
title: Hexo Blog 的部署
---

相信各位，特别是喜欢玩博客的朋友都有听过 Hexo 这个框架了，本人在用它来搭建自己的空间的时候遇到了不少的问题，现在就写一篇 blog 来记录一下搭建过程。

<!-- more -->

## Hexo 简介
Hexo 是一款基于 Node.js 的静态博客框架，依赖少，而且可以很方便地将静态页面托管在 Github 上。详细介绍或者 API 可参考 [Hexo 官网](https://hexo.io/zh-cn/)或者它的 [Github 工程页面](https://github.com/hexojs/hexo)。

## 搭建步骤

### 1. 准备 Github 账号与仓库

1. 在[ Github 官网](github.com)注册好账号，并记录下自己的【用户名】。
    * 这一步很重要，因为这与你将要创建的博客的**访问网址**直接相关。
    * 注：用户名可以修改，但是有的时候会带来[副作用](https://docs.github.com/en/github/setting-up-and-managing-your-github-user-account/managing-user-account-settings/changing-your-github-username)。
2. 创建以 [用户名].github.io 为名的 repository。


### 2. 建立本机与 Github 的 SSH 连接

从 [git 官网](https://git-scm.com/)下载并安装 git。安装完成后可通过以下命令验证：
```
git --version
```

完后需要生成 ssh key
```
ssh-keygen -t rsa -b 4096 -C your_email@example.com
[press enter] //默认位置~/.ssh
[press enter] //key不需要密码保护
[press enter] //确认不需要密码保护
```
随后复制 ssh key 到 [Github 页面中](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)。
```
$ pbcopy < ~/.ssh/id_rsa.pub
```

具体可参照 Github 提供的[方法](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。

验证方法：
```
ssh -T git@github.com
```


### 3. 安装 Hexo 及其依赖
* Node.js [安装](https://nodejs.org/en/)
    * 验证：
    ```
    node -v
    npm -v
    ```
    * 注：所安装的 Node.js 版本需在 12 以上。
* Hexo：
    ```
    npm install hexo-cli -g
    ```

* Hexo Git Developer：用于博客页面的更新与部署。
    ```
    npm install hexo-deployer-git --save
    ```


### 4. 开搞
* 在适当的位置初始化 Hexo 博客文件夹：
    ```
    hexo init myBlog
    ```

* 进入到新创建的文件夹中：
    ```
    cd myBlog
    ```
    此时能看见的文件与文件夹：
    * `node_modules`：依赖包
    * `public`：存放生成的页面
    * `scaffolds`：生成文章的模板
    * `source`：存放文章的位置
    * `themes`：博客主题
    * `_config.yml`：博客配置文件

* 配置页面
    ```yaml
    # _config.yml
    # 配置内容
    deploy:
    - type: git
      repo: git@github.com:[用户名]/[用户名].github.io.git  # 该【用户名】为之前步骤提到的需要记录的用户名
      branch: main  # 新创建的 Github repo 的 branch
    ```
    注：
    * 配置的用户名需保持一致，此时 Github Pages 会根据 repo 所配置的值生成页面的访问地址。如用户名为 zhangsan 的话，最终的博客访问网址为 https://zhangsan.github.io
    * repo 亦可配置按照 https 方式配置：https://github.com/[用户名]/[用户名].github.io.git。

* 清理 Hexo 博客文件夹：
    ```
    hexo clean
    ```

* 创建新的博客页面
    ```
    hexo new myPage
    ```
    * 此时文件 myPage.md 会在 `source/_posts` 下生成。

* 生成 Hexo 博客静态页面：
    ```
    hexo g
    ```

* 部署 Hexo 静态页面至 Github 中，由 [Github Pages](https://pages.github.com/) 保管并渲染、显示：
    ```
    hexo d
    ```
    注：如配置成 https 形式访问，则会有提示输入账号密码。

* 此时访问 *https://[用户名].github.io*，即可登入搭建的页面啦。

* 也可现在本地搭建 Hexo 服务，通过 http://localhost:4000 查看博客效果：
    ```
    hexo s
    ```

---
## 参考
[hexo史上最全搭建教程](https://blog.csdn.net/sinat_37781304/article/details/82729029)

[Luck's Cabin - Hexo博客部署](https://fly-luck.github.io/2016/03/05/Hexo%20Blog/)
---
title: 多端部署博客——用GitHub Actions部署Hexo
tags: 博客
---

# 多端部署博客——用GitHub Actions部署Hexo

## 1.安装Git

- Windows：下载并安装 [git](https://git-scm.com/download/win)。
- Mac：使用 [Homebrew](http://mxcl.github.com/homebrew/), [MacPorts](http://www.macports.org/) 或者下载 [安装程序](http://sourceforge.net/projects/git-osx-installer/)。
- Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

> Mac 用户
>
> 如果在编译时可能会遇到问题。 请先到 App Store 安装 Xcode。 Xcode 完成后，启动并进入 **Preferences -> Download -> Command Line Tools -> Install** 安装命令行工具。



## 2.安装NodeJs

Node.js 为大多数平台提供了官方的 [安装程序](https://nodejs.org/zh-cn/download/)。

其它的安装方法：

- Windows：通过 [nvs](https://github.com/jasongin/nvs/)（推荐）或者 [nvm](https://github.com/nvm-sh/nvm) 安装。
- Mac：使用 [Homebrew](https://brew.sh/) 或 [MacPorts](http://www.macports.org/) 安装。
- Linux（DEB/RPM-based）：从 [NodeSource](https://github.com/nodesource/distributions) 安装。
- 其它：使用相应的软件包管理器进行安装。 可以参考由 Node.js 提供的 [指导](https://nodejs.org/en/download/package-manager/)。

对于 Mac 和 Linux 同样建议使用 nvs 或者 nvm，以避免可能会出现的权限问题。

> Windows
>
> 使用 Node.js 官方安装程序时，请确保勾选 **Add to PATH** 选项（默认已勾选）
> Mac / Linux
>
> 如果在尝试安装 Hexo 的过程中出现 `EACCES` 权限错误，请遵循 [由 npmjs 发布的指导](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally) 修复该问题。
> Linux
>
> 如果您使用 Snap 来安装 Node.js，在 [初始化](https://hexo.io/zh-cn/docs/commands#init) 博客时您可能需要手动在目标文件夹中执行 `npm install`。

## 3.安装Hexo

Hexo可以使用npm进行安装,但在使用前,最好配置一下镜像.

``` sh
npm config set registry https://registry.npmmirror.com
```

配置完镜像后,即可使用npm进行安装.

```sh
npm install -g hexo-cli
```

## 4.使用Hexo建站

安装完Hexo之后,就可以在本地选择一个文件夹,开始建站了。比如我要在本地创建一个名为Blog的文件夹。可以执行如下命令。

```sh
hexo init Blog
cd Blog
npm install
```

等到npm命令执行完毕，Blog目录中会出现如下结构。

```
.
├── _config.landscape.yml
├── _config.yml
├── node_modules
├── package-lock.json
├── package.json
├── pnpm-lock.yaml
├── scaffolds
├── source
└── themes
```

_config.yml是博客的配置文件，本篇只介绍部署，暂时不介绍配置。

这时输入指令`hexo s`或`hexo server`，即可在本地测试博客效果。默认端口为`http://localhost:4000`。

![image-20241213194144483](/Users/wang/Library/Application Support/typora-user-images/image-20241213194144483.png)

## 5.创建一个仓库

如果你希望你的博客部署后是`username.github.io`直接能访问，就将仓库名字设置为`username.github.io`。

如果希望部署后还要一级，如`username.github.io/blog`,就将仓库名字设置为`blog`。

## 6.关联GitHub Pages和Hexo

### 1.生成部署密钥

```sh
ssh-keygen -f github-deploy-key
```

输入后一直按回车即可。执行结束后，会在当前目录生成两个文件。

- github-deploy-key
- github-deploy-key.pub

### 2.配置部署密钥

GitHub上打开上面创建的仓库，找到`Settings->Secrets and Variables->Actions->New repository secret`。

- `Name`:`HEXO_DEPLOY_PRI`
- `Value`:`github-deploy-key`中的内容

还是在这个仓库，找到`Settings->Deploy keys->Add deploy key`。

- `Title`:`HEXO_DEPLOY_PUB`
- `Key`:`github-deploy-key.pub`中的内容
- 勾选`Allow write access`选项

## 7.创建配置文件

在电脑本地查看node版本。

```sh
node -v
```

前往上面创建的仓库，找到`Settings->Pages->Build and deployment->source`，将其更改为`Github Actions`。

在`.github`目录下创建`workflow`目录，在这个目录下添加`page.yml`文件,内容如下：

```yaml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

将其中的node版本替换为自己需要的版本即可。

## 8.大功告成

将上面的项目推送到刚刚创建的仓库，访问对应的地址：

- 如果仓库名是`username.github.io`，访问`username.github.io`。
- 如果仓库名是`blog`,访问`username.github.io/blog`。

或者可以在GitHub的Page页查看。

![image-20241214091039592](/Users/wang/Library/Application Support/typora-user-images/image-20241214091039592.png)



> 参考资料：
>
> [Hexo官方文档](https://hexo.io/zh-cn/)
>
> [利用 Github Actions 自动部署 Hexo 博客](https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/)
>
> [从零开始搭建 Hexo 博客简明教程（2024版）](https://www.philoli.com/building-a-blog-from-scratch/)

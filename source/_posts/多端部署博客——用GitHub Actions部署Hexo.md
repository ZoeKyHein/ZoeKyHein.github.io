---
title: 多端部署博客——用GitHub Actions部署Hexo
tags: 博客
catogory: 胡乱捣鼓
---

# Hexo + GitHub Actions + Page部署自动化博客

最近想部署一个博客，网上看了很多相关的教程，但大部分都要么版本不一样了，要么缺少关键步骤。不如自己总结一下，来一个自己认为比较全面的，希望能给后面有相关需求的朋友一点帮助。

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

为了方便和上手，这里使用Vscode+git操作。

安装完Hexo之后,就可以在本地选择一个文件夹,开始建站了。比如我要在本地创建一个名为Hexo的文件夹。

使用Vscode打开这个文件夹，新建一个终端，运行如下命令

``` shell
hexo init
npm install
```

如果`npm install`失败，可以尝试删除文件夹中的`node_modules`文件夹重试。

等到npm命令执行完毕，Hexo目录中会出现如下结构。

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

初始化完成后，将目录中的内容推送至仓库的main分支，这里我选择使用Vscode+git更方便的进行创建并推送。

![image-20241216111417720](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216111417720.png)

## 5.创建Token。

点击GitHub头像，选择`Settings->Developer settings->Personal access tokens->Tokens(classic)->Generate new token->classic`。

![image-20241216111650090](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216111650090.png)

勾选`repo`和`workflow`，Note填入`GH_TOKEN`(其实都可以，与后面对应即可，如果不熟悉就跟着来吧)。有效期根据自己的需要选择。设置完成后点击生成，然后复制生成的token，注意保存，token只能查看一次，如果忘记了后面需要重新生成。

![image-20241216111819418](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216111819418.png)

## 6.将创建的token放入仓库内。

进入刚才推送出来的仓库，选择`Settings->Secrets and Variables->Actions->New repository secret`。

![image-20241216112206006](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216112206006.png)

将刚才我们复制的token填入，名称填写`GH_TOKEN`。

## 7.配置仓库地址。

回到Vscode，找到我们新建文件夹中的`_config.yml`文件，拉到最下方，将配置进行如下修改。

```yaml
deploy:
  type: git
  repo: https://github.com/ZoeKyHein/Hexo.git
  branch: gh-pages
```

仓库名称和地址根据自己的，灵活修改。

## 8.配置GitHub Actions工作流

在`.github`目录下新建一个名为`workflows`的文件夹(注意是有s的)。在其中新建一个名为`deploy.yml`的文件，复制以下内容。

``` yaml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: false
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "22"

     
      - name: Install Dependencies
        run: npm install

      - name: Install Hexo Git Deployer
        run: |
          npm install hexo-deployer-git --save
          npm install hexo-cli -g

      - name: Clean and Generate Static Files
        run: |
          hexo clean
          hexo generate

      - name: Configure Git
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'

      - name: Deploy to GitHub Pages
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: |
          cd public/
          git init
          git add -A
          git commit -m "Create by workflows"
          git remote add origin https://${{ secrets.GH_TOKEN }}@github.com/ZoeKyHein/Hexo.git
          git push origin HEAD:gh-pages -f
```

其中有两个位置需要修改：

- 一是node版本需要与本地对应。在本地运行`node -v`得到版本，替换上面的`22`。
- 二是最下方仓库地址修改为自己的仓库路径。

其实Actions工作流的本质就是，当你提交git时，自动帮你把markdown转静态网页，发布等工作给做了。相当于我们在后续维护博客的过程中，只需要使用git提交即可，后续操作都会自动帮我们完成。

修改完成后，将刚才的所有修改全部推送上去，查看GitHub仓库的`Action`是否有工作流工作。

![image-20241216113145810](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216113145810.png)

正常情况下，该工作流前面会有一个✅标志，表示没有错误发生，正常进行。

回到`Code`，查看是否出现了`gh-pages`分支，并检查`gh-pages`分支下的`index.html`文件是否为空，如果为空，需要根据工作流日志查阅一下问题。

## 9.配置GitHub Pages

进入到仓库，选择`Settings->Pages`,确保分支时`gh-pages`。

![image-20241216113459989](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216113459989.png)

关于网站的访问地址，可以在图中上方红框框起来的区域查看。规则如下：

- 如果想要通过`https://username.github.io`直接访问，需要把仓库名称修改为`username.github.io`，且这样不会出现外部文件引用出错的问题，**推荐**。
- 如果想要通过`https://username.github.io/xxx`访问，仓库名字设置为`xxx`，但这样需要多配置一步，来解决外部文件的引用错误。

访问网站，如果你用的是第一种，那么应该可以看到Hexo的默认页面。(这里图片是使用了主题，所以看起来可能与你的不太一样，问题不大，意思到了就好)

![image-20241216114436636](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216114436636.png)

如果是第二种，需要回到配置文件`_config.yml`，解决一下资源请求的问题。

![image-20241216114239466](https://cdn.jsdelivr.net/gh/ZoeKyHein/ImageRepo/image-20241216114239466.png)

将`url`字段修改为仓库的地址，`root`字段修改为仓库名称。可以顺手将`post_asset_folder`属性修改为`true`来解决图片不显示的问题。

## 10.修改主题

至此，默认的Hexo的配置已经完成了。下面来说一下如何更改主题，我这里用的是[fluid](https://github.com/fluid-dev/hexo-theme-fluid)，以这个为例。每个主题一般都会提供安装文档。按照安装文档一步步进行即可。

---

至此，就已经完成了全部配置，可以开始自己的博客写作之路了。

> 参考资料：
>
> [Hexo官方文档](https://hexo.io/zh-cn/docs/)
>
> [9分钟零成本搭建自动化部署个人博客(Hexo + Github Action + Page)](https://www.bilibili.com/video/BV1xTgTemEDU/?spm_id_from=333.337.search-card.all.click&vd_source=54e860cc95e5fe130d79a442d282774f)
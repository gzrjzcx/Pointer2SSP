# Introduction

This is a book where records the interesting programming problems encountered in training, tests and interviews etc.

---

在这里，我想介绍一下目前所用的写文档到静态网站的解决方案：

**`Gitbook` + `Github` + `Travis` + `Typora`**

因为在国内使用Gitbook的新版online editor体验很糟糕（需要科学上网），且desktop editor对于新注册用户已现在不能验证到GitHub上，所以只能自己在本地写好文章在发布到静态网站上。又因为不想每次都手动build，然后push到GitHub上，所以在此我们利用Travis来自动build和push到仓库上。最终实现的效果为每次在本地写好文章后，将新写的文章push到自己的仓库，然后Travis则会自己build新的静态网站所需文件并且自动push到gh-pages所在的分支，然后自动部署。所以我们只需要做的就是写文章然后推送到自己仓库即可。

- `Gitbook`主要用来生成静态网站文件，也即是你现在所看到的这本书。
- `Github`主要还是做version control，同时提供了gh-pages来部署生成的文章。
- `Travis`则主要是负责自动生成build的静态文件并push到仓库上。
- `Typora`则是主要用于本地的写作工作。


## 1. 配置Github

首先，新建一个repo，同时开启[gh-pages](https://pages.github.com/)服务。Github pages主要用于部署我们生成的静态文件，具体设置在此省略。这里需要说明的是分支的问题。我们需要在`master`分支上存放我们自己写的源文件（i.e. md文件）等，然后在`gh-pages`分支存放build出来的文件等用于部署静态网站的文件（i.e. index文件等）。并且需要注意的是index文件等需要从build生成的_book文件夹中移动到`gh-pages`分支的最外层`gh-pages`才能正确部署。也就是说，一次手动的（非Travis自动更新）的写作流程应该是（此处不更新写好的md文件）：

写文章 --> gitbook build --> 复制生成的\_book文件夹下的所以文件到分支`gh-pages` --> push到`gh-pages`

## 2. 配置Gitbook

首先，需要在本地安装好Gitbook需要的依赖[Node](https://nodejs.org/zh-cn/)。
然后安装并使用Gitbook：

```shell
sudo npm install -g hexo-cli
gitbook init
gitbook build # 会生成_book文件夹，包含了部署静态网站所需文件
```

如何使用Gitbook的细节可以查看[here](https://www.bookstack.cn/read/gitbook-zh/README.md)。

## 3.配置Travis

`Travis`是用来做CI（Continuous Integration）的，也就是能够在仓库代码变动时自动编译测试等以保证第一时间发现代码问题。在此，我们主要利用`Travis`来自动build文章和push。

1. 给新建的仓库开启`Travis`服务，这篇博客很详细[here](https://www.jianshu.com/p/3b8d86f25ee2)。
2. 因为我们需要push新生成的文件到仓库，所以需要生成GitHub的security token来获取更改仓库的权限[here](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)。
3. 设置好token后则可以给仓库[配置`yml`文件](https://docs.travis-ci.com/user/deployment/pages/)。注意一旦开启了Travis服务，只需要将yml文件上传到仓库则会自动激活Travis的服务。我自己用的yml文件可以在下方代码。



```yaml
language: node_js
node_js:
  - "10"
cache: npm

notifications:
  email: false

install:
  - npm install -g gitbook-cli
  - gitbook install

script:
  - gitbook build

deploy:
  provider: pages
  skip_cleanup: true
  github_token: $TOKEN  # Set in the settings page of your repository, as a secure variable
  local-dir: ./_book/
  verbose: true
  on:
    branch: src # I use src branch instead of master. For new repo, you can use master

branches:
  only:
    - src
```

## 4. 完成并测试

在配置完`yml`文件后且上传到存放源文件的分支（master）后，以后每次便只需要在本地写好文章，然后push写好的md文件到master分支，`Travis`分支则会自己build出部署静态网站所需文件且push到`gh-pages`分支上，从而可以直接打开相应的静态网站查看书籍。关于Travis，还可以增加build成功后的通知信息等。默认为邮件通知，但是在yml文件中已经被我取消，有时间可以试试[CCMenu](https://docs.travis-ci.com/user/cc-menu/)来增加Mac的通知栏上的通知。	
---
title: hexo主题安装
permalink: hexo主题安装
categories: [Hexo]
tags: [Hexo]
date: 2019-07-20 16:38:30
---

## 简介

目前博客有很多方式可以书写，如简书，CSDN等等，由于经常使用Git，故打算采取hexo进行自主博客的搭建。github是支持个人主页的博客搭建和项目的博客搭建的，很是方便了，一下便是采取hexo进行的个人博客的搭建过程。

## 安装Hexo

首页，我们进行hexo的安装。电脑上需要安装好基本的工具npm，node等等。安装hexo脚手架，命令：

```
npm install -g hexo-cli
```

安装好脚手架之后，新建一个空的目录folder，用来放置个人hexo生成博客源码的文件。进入文件夹，初始化hexo：

```
hexo init <folder>
cd <folder>
npm install
```

经过以上几个步骤，hexo就搭建完成了，可以进行hexo的操作了。

具体的可以参考官方：[hexo官方文档](https://hexo.io/zh-cn/)

## Hexo命令介绍

经过npm之后，hexo结构项目是有了。此时，运行`hexo server`就会进行项目的启动了，启动成功之后，访问`localhost:4000`即可看到默认的基础博客了。后续我们写的文章都会在此生成的展现出来的。



hexo常见命令如下：

```
hexo clean --清除缓存
hexo generate --生成个人博客所需的静态页面
hexo server --本地预览
hexo deploy --部署我们的个人博客
hexo new "postName" --新建文章
hexo new page "pageName" --新建页面
```

1. hexo server 命令运行后，可以直接启动博客系统，方便我们立刻查看，写的文章效果呈现。
2. hexo generate 是生成博客文档的。运行此命令之后，我们的所有博客都会经过组织生成html文件，并放置到public文件夹下，运行此命令之后，就可以看到public文件夹，里面就是博客网站的源码。
3. hexo clean 会将生成的public文件进行清空删除操作。
4. hexo deploy 是可以将我们系统生成的public文件夹所有内容，上传到指定地方去。例如：git。配置之后，当执行此命令，就会将public所有文件上传到git所在仓库中去。
5. hexo new 最常用的命令，写篇文章时，就需要运用此命令了。会在hexo的源文件source下生产markdown文件，然后就可以进行书写了。
6. hexo new page 生成页面，hexo生成的博客，默认上面只有首页，归档等等。类别不多，当我们需要其他的例如：分类，标签的时候，就需要运行此命令处理了。

## Next主题

hexo生成的博客样式是默认样式，放在theme目录下，我们可以选择其他博客样式来展示的。很简单，仅仅需要将需要展示的主题放入/theme/目录下，然后修改配置文件`_config.xml`文件中的theme即可。目前最流行的hexo主题是next，下面就展示如何安装。

```
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

经过以上命令，就将next主题下载到了`themes/next`目录下了，然后修改配置文件`_config.xml`将theme修改下`theme: next`就已经应用好了主题了。

具体的参考官网：[next官方文档](http://theme-next.iissnan.com/)

## Github上传

我们的博客生成好了之后，是放到public目录下面的，只能自己查看其他人也无法查看。此时，可以将public下的源码放到服务器上就可以了。服务器还需要单独购买，此时，我们可以选择放到Github上，Github的github pages是支持直接展示html的。

搜索github pages了解此项功能后，然后在Github上新建以`username.github.io`的仓库，这个仓库只能建立一个。建立好了之后，我们就可以把hexo生成的public文件放到项目中了。放置好了之后，直接访问`username.github.io`就可以看到博客了。

hexo可以结合github进行自动上传操作，进入`_config.xml`配置文件，设置`deploy`命令即可。

```
deploy:
  type: git
  repository: git@github.com:username/username.github.io.git
  branch: master
```

注：type类型最新的hexo是git了，不是github。

## Github源码保存

使用上面方式是可以部署github，但是此时git上面的master只有生成的博客文件，并没有我们的各种配置和源码。故，我们需要将源码也保存到github上面去，当本地环境变了之后，仍然能够进行构建。



复用之前的git仓库，建立hexo分支。（master分支只能放置博客文件，源码只能放置其他分支上面去）此时，hexo分支保存我们hexo init操作后的源码，即`_config.xml`根目录。源文件修改后，立刻进行hexo分支的推送保存操作。然后执行`hexo generate && hexo deploy`命令，进行public文件的生成，并自动上传到github的mater分支上面去。

可以参考博客进行处理：[静态博客Hexo搭建 —— 本地、服务器两开花](https://juejin.im/post/5cef68415188253fed710994)

## Hexo主题的源码上传

上面可以将hexo源码上传到github上去，但是我们的主题处理上面就有些费事了，我们的主题会有很多个性化的设计的。以下有2种方式可以选择处理。

1. Hexo的主题当做文件来处理

   下载主题并复制到`themes/`目录下面去，删除主题的.git配置文件，将主题的git项目全部当做普通该文件处理，此时主题也是hexo源码的一部分了。优点：方便，便捷。缺点：主题不好升级。

2. Hexo的主题也是git项目

   下载主题到`themes/`目录下，此时的主题也是一个git项目，相当于hexo源码是个git项目，里面的主题又是一个git项目，此时，采取git的子模块形式，运行`git submodule add xxx`将主题的git仓库当做hexo源码的子模块即可。  

   主题由于是子模块，就可以进行升级操作了。而主题则从选择的官方主题git仓库那里fork一份到自己仓库中，如此，主题的配置也会保存下来了。

这里参考博客： [hexo 使用子模块实现自定义主题与hexo文件分离，在两个git维护](http://zzy.design/6/)

## 与Travis CI构建自动上传

上面虽然解决了源码保存和生成问题，但是每次写文章还需要进行博客的生成和提交操作，还是比较费事的，此时，可以采取`Travis CI`进行自动编译保存上传操作。  

到`travis.com`下进行账号注册，关联到github上面去。然后编写脚本进行自动上传和保存操作。注意：travis.org仅可构建公开项目，已经逐渐废弃了，后续全部迁移到travis.com上去，即可构建公开项目，又可构建私有项目。这里容易让人纠结，可参考：[What's the difference between travis-ci.org and travis-ci.com?](https://devops.stackexchange.com/questions/1201/whats-the-difference-between-travis-ci-org-and-travis-ci-com)

至于travis的编写，还有自动上传之后，提交记录等等很多问题。目前网上有很多方案，个人看了很多，都觉得不满意，而且很混杂，处理方式什么都有。不是很好的方案，最终搜到这篇文章： [使用Travis CI自动部署Hexo博客](https://www.itfanr.cc/2017/08/09/using-travis-ci-automatic-deploy-hexo-blogs/)

这篇文章详细介绍了与travis结合遇到的各种问题，很是全面。本人最后也是采取此方式处理的，很好，很强大。如果要配置的话，建议进入此博主的github项目，找到他构建的github pages，拷贝出来自动上传配置文件即可。

注：强烈建议查看此博客进行travis自动上传，本人博客也是拷贝此博客配置文件来的，目前运行极好。

## 后记

当初就知道hexo，一直没有手动完全了解过。这次由于一些原因，下决心整理下hexo的完全搭建步骤，中途看了很多文章，才算是摸清了解一些内容，此篇文件也是将大体流程梳理下了，其中涉及到的部分小细节也没详细说明了，靠个人理解下了。了解到了写个博客，打字整理也是很费时间的了。

## 参考

以下博客可以进行参考，进行theme的优化等。

- [Hexo-NexT配置超炫网页效果](https://www.jianshu.com/p/9f0e90cc32c2)

- [我的个人博客之旅：从jekyll到hexo](https://blog.csdn.net/u011475210/article/details/79023429)

- [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)

- [2018最新版hexo+Github搭建个人博客教程（2018-09-10 更新）](http://www.bianxiaofeng.com/view/5)

- [Hexo+NexT 打造一个炫酷博客](https://juejin.im/post/5bcd2d395188255c3b7dc1db#heading-55)

- [使用Travis CI自动构建hexo博客](http://magicse7en.github.io/2016/03/27/travis-ci-auto-deploy-hexo-github/)

- [如何优雅地发布Hexo博客--使用ngxin,hexo-admin](https://blog.lzoro.com/2017/08/17/hexo_grace_deploy/)

- [使用Hexo搭建博客之进阶篇](https://www.xiaocoder.com/2018/07/14/hexo-blog-advance-guide/)

- [静态博客Hexo搭建 —— 本地、服务器两开花](https://juejin.im/post/5cef68415188253fed710994)

- [使用 Travis CI 自动部署 Hexo 博客](https://blessing.studio/deploy-hexo-blog-automatically-with-travis-ci/)

- [使用hexo搭建的网站，并含有hexo优化](https://www.zhyong.cn/)

---

title: 基于hexo框架以及GitHub搭载个人博客网站 （小白向）

date: 2022-07-13 04:07:56

tags: hexo

---



> 该博文是博主的一个朋友想要了解个人博客网站的搭建而撰写的一篇简易的搭建教程。我贾哥说过学习巩固某一知识点的最好方法便是传授予他人。那么话不多削我们这就开始，因为这是个没有计算机基础的同学所以我会从最基础的地方开始撰写，篇幅可能会有点长请耐心食用。



#### 0. 首先最开始的一步就是 科学上网

> 科学上网俗称 ‘翻墙上外网’，首先你要有能够翻墙上外网的能力才可以方便我们上GitHub去注册账户。当然也不局限与这一点，有了科学上网的能力你便可以使用谷歌搜索引擎以及浏览类似YouTube的国外网站或者某些奇奇怪怪的东西。（手动狗头.jpg）

那么怎么进行科学上网（翻墙）呢 ？

咱也不展开来讲有兴趣的可以自行谷歌一下翻墙原理。

这里或许有人会和我吐槽我都翻不了墙怎么上谷歌？ 最简单的一个方法就是找一个一站式的翻墙软件。博主这里用的是 [蜂巢vpn](https://666yun.men/#/register?code=E9pw8QiO) 这上面都有相应的一个使用教程的。



#### 1. 注册GitHub账号以及下载对应的代码编辑器以及编程环境



既然能科学上网了就先去[GitHub](https://github.com/)注册个账号吧 

然后下载所需的一个代码编辑器 这里推荐 [VSCode](https://www.runoob.com/w3cnote/vscode-tutorial.html)

下载完之后呢我们可以在桌面上新建一个文本文档，然后将其改名为.html后缀的文件，再通过右键或者拖拽至VSCode进行打开。

> tips: 如果不会的话可以先把下述代码贴在文本文档内再去更改为.html后缀 双击打开 





```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
  	<title>Document</title>
    <style type="text/css">
  div{
	width: 300px;
    height: 300px;
    background: pink;
    border-radius: 0  0  300px 0;
  }
  </style>
</head>
<body>
    <div></div>
  	<h1>你好呀！ 隔壁黄阿姨 (*´▽｀)ノノ❤ </h1>
</body>
</html>
```

![helloAntHuang](../../../../img/helloAntHuang.png)



这就是我们简单设计的静态页面啦，但是只能跑在自己的电脑上或者是把生成的.html发给别人，别人在自己的电脑上打开。

这时候距离我们建立一个个人博客网站的目标好像差了不是一星半点，若是我要教你自己通过这种方式一点点自己敲出来的话，你应该可以过来和我共事了__(:з」∠)_，怕你们抢我饭碗所以我们这里要借助第三方的框架来进行页面基础架构的搭建！（ 还要啥自行车╭(╯^╰)╮）。

但是我们要使用第三方插件的话 我们就得有一个nodejs的环境来执行下载的脚本

这里有个博主早期写的一个[nodeJs安装教程](https://blog.csdn.net/Jayhodon/article/details/108308405?spm=1001.2014.3001.5501)可以参考一下。

#### 2. 使用hexo框架进行个人博客搭建

  我们可以在[hexo官网](../../../../img/hexo_install.png)

  之前我们安装的nodejs就是为了能够执行第一条命令全局安装hexo的插件。

![安装Hexo插件](../../../../img/vscode_hexo.png)

按照上述步骤来的话，不出意外就是这样的。

![初始化Hexo框架](../../../../img/hexo_init.png)

根据提示按住Ctrl点击那个:4000的链接就可以跳转到对应页面![Hexo初始化页面](../../../../img/hexo_init_page.png)

这上面有一个简单介绍，具体的使用说明可以看[官方文档](https://hexo.io/zh-cn/docs/)

到这里你就成功的迈出了一大步了，接下来我们就可以去官网的主题Tab栏下面寻找你喜欢的博客主题。可以先在官方文档了解一下hexo的主题。这里博主采用的是 blinkfox 前辈的主题 [教程开源主题介绍](https://blinkfox.github.io/2018/09/28/qian-duan/hexo-bo-ke-zhu-ti-zhi-hexo-theme-matery-de-jie-shao/)。（前辈的使用教程写的很清楚简洁了我这里就不做多余的赘述了 【其实是现在已经快3点了 扛不住了 明晚再肝吧 晚安社会黄 *_(:з」∠)_* 】 ）

#### 3. 挂载hexo主题框架

接上文 我们通过 blinkfox 前辈的开源主题介绍 将下载并解压的hexo_theme主题文件放到blog文件夹下对应的theme文件夹下后，对_config.yml文件进行配置。

> 这里博主一开始采用的这种方法，但是有些配置项比较麻烦，所以建议使用第二种方法，但是_congfig,yml的配置是一样的流程

![hexo_theme](../../../../img/hexo_theme.png)

> 但是用这种直接解压的方式可能会跑步起来 要进行一些其他的配置项 我不嫌麻烦 你们也会嫌麻烦把 （ 好吧我也嫌麻烦 x_x ）

第二种方法就是在blog下的themes文件夹下打开终端，通过拉远程代码库的形式来部署主题文件。![拉取hexo主题代码](../../../../img/hexo_theme_clone.png)

![hexo-theme-matery](../../../../img/hexo-theme-matery.png)

成功启动后的页面就是这个样子哒~~~ ，是不是有一种刚刚还是在拧螺丝现在突然发现火箭突然起飞的感觉。

#### 4. 发布个人博文

既然博客的框架已经搭建好了，那么我们得用起来我们得发布文章呀。

> hexo新建文章的方式便是通过 hexo new '新建博文.md标题'

![new_post](../../../../img/new_post.png)

![新建的文章](../../../../img/hexo_new_post.png)

但是这样子通过VSCode自带的.md编辑器写博文是一件非常蛋疼的事情，对的没错我们又要借助第三方的工具了，博主这里使用的是 [Typora](https://typoraio.cn/) 。我们可以使用类似的第三方Markdown编辑器来撰写我们的博文，然后将编辑排好版的.md文件直接贴到 source\posts 文件夹下再来发布博文 这样的体验会好一点。

![typora](../../../../img/typora.png)



>  \---
>
> title: 新建的博客标题
>
>  date: 2022-07-17 01:29:48
>
> tags: 博客的类型标题
>
>  \--- 
>
> 记得将上面这一段加在博文.md文件的最前面哦 这是hexo识别文章的参数

#### 4. 博客个人配置化

首先是整个框架的的配置，这些在 blinkfox 前辈的 [教程开源主题介绍 ](https://blinkfox.github.io/2018/09/28/qian-duan/hexo-bo-ke-zhu-ti-zhi-hexo-theme-matery-de-jie-shao/)中有一些个性化的配置没有打开 有兴趣的的可以查阅一下 写的已经很详细了 我就不一一赘述了。这里我着重讲一下如何去更改的一个 思路 / 方法 。

> 我这里采用的是最笨的办法，也是最简单的办法，有更加简单的办法可以交流传阅一下 Thanks♪(･ω･)ﾉ 

1. 首先是_config.yml内的一个配置，这里可以更改hexo的一个默认配置。

![hexo_author](../../../../img/hexo_author.png)

2. 但是页面上的一些文案之类的要怎么处理呢，这里最简单的一个 方法/ 思路 就是将文案进行复制然后进行全局搜索进行替换。

![hexo_style](../../../../img/hexo_style.png)

这里我们以 【我的梦想】为例子

![hexo_theme_config](../../../../img/hexo_theme_config.png)

3. 如果博文内含有图片的话要记得将图片文件放到编译生成的public里的img文件夹内部 并在博文.md内修改引用的图片地址。（ 不小心又肝到了4点，这一块后面在写！）

![修改图片路径](../../../../img/change_img.png)

这里按照上图 将图片文件放在img文件下 并 更改文件路劲即可。 文件路径不清楚的可自行百度一下（一般按照上面这个路径也不会出错）

#### 6. 部署在GitHub上

在hexo的[官方文档](https://hexo.io/zh-cn/docs/github-pages)上有如何部署到GitHub的讲解，有兴趣的可以前往查阅。

1. 安装Git

  为了能把本地的项目上传到GitHub上去，我们就会需要使用到 Git --- 【分布式版本控制工具】 [安装链接](https://git-scm.com/download/win)

  安装的时候选择好对应的位数无脑然后全部无脑默认下一步即可。

>    最后一步添加路径的时候选择 Use Git from the Windows Command 以便于我们可以直接在命令行里直接打开git

  安装完后可以在命令行中输入`git --version` 来验证是否安装成功。

2. 登录GitHub并新建一个代码仓库

![new_repository](../../../../img/new_repository.png)

3. 设置github pages

![初始化github pages](../../../../img/init_github_pages.png)

选择好后拉到下面 点击 commit change 然后在回去刚刚那个页面就会看到有个提示

>   Your site is ready to be published at  【你的个人博客网站地址】

打开这个地址就可以访问到你的个人博客网站啦，接下来就是把我们用hexo框架搭载的博客挂载上去了

![github_pages](../../../../img/github_pages.png)

4. 连接Github与本地

![git_bush](../../../../img/git_bush.png)

首先右键打开git bash，然后输入下面命令：

```bash
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

然后生成密钥SSH key：

```bash
ssh-keygen -t rsa -C "邮箱"
```

打开[github](https://link.zhihu.com/?target=http%3A//github.com/)，在头像下面点击`settings`，再点击`SSH and GPG keys`，新建一个SSH，名字随便。

然后在git bash中输入

```bash
cat ~/.ssh/id_rsa.pub
```

将输出的内容复制到框中，点击确定保存。

输入`ssh -T git@github.com`，如果如下图所示，出现你的用户名，那就成功了。

>  Hi Jayhodon! You've successfully authenticated, but GitHub does not provide shell access.

5. 将远程代码库的代码下拉至本地

![获取仓库里的远程代码链接](../../../../img/git_clone.png)



![拉取仓库里的远程代码](../../../../img/git_bash_clone.png)

> 输入 git clone [你复制的SSH链接] 
>
> 然后桌面上会出现一个拉下来的文件夹 这时候我们就把blog直接拷贝进去 并用 VSCode 打开这个文件夹

![文件传输](../../../../img/blog_to_io.png)

6. 配置_config.yml 下的deploy参数

> deploy:
>
>  type: 'git'
>
>  repository: 'github 上的 ssh链接'
>
>  branch: main

7. 将代码部署到github上

在终端输入以下指令 安装 hexo-deployer-git 插件

> npm install hexo-deployer-git --save

**此处附加上 hexo 框架常用的指令：**

| 指令                   | 说明                                      |
| :--------------------- | :---------------------------------------- |
| `hexo clean && hexo g` | 清除本地项目并重新生成 （重新部署时使用） |
| `hexo g`               | 重新生成                                  |
| `Hexo s`               | 开启本地预览                              |
| `Hexo d`               | 推送到github                              |

安装成功后就可以输入命令： hexo g -d  来上传到对应的github地址。到了这里基本上就大功告成了。

> hexo clean 最好不要用 x_x 
>
> 先用 hexo s 来本地查看 然后使用 hexo g -d 推代码 就够了

安装成功后就可以输入命令： hexo g -d  来上传到对应的github地址。到了这里基本上就大功告成了。



#### 小记

> ​	这篇文章的是写给不懂相关计算机知识的小白的个人简易博客网站搭建的教程，通篇博主都以读者的角度出发。个人认为如果这一片文能够让一个小白能够搭建其属于自己的博客网站，那么这篇文就是有意义的。
>
> ​	现在是简单的教授了如何搭建以及发布博文，后续如果博主有时间的话会去更新一下个性化配置（诸如音乐播放器，视频播放器之类的博客小组件）以及使用上的优化（发布博文过于繁琐需要不断提交代码）之类的东西。
>
> ​	这里如果有小伙伴们是哪里卡住了 或者是不懂得都可以给我留言 工作之余我会改进一下 争取让这篇文成为有手就行的个人博客网站搭建教程搭建

【这里着重感谢一下 blinkfox 前辈提供的hexo开源主题框架】

​	

#### 参考出处

[Hexo博客主题之hexo-theme-matery的介绍](https://blinkfox.github.io/2018/09/28/qian-duan/hexo-bo-ke-zhu-ti-zhi-hexo-theme-matery-de-jie-shao/)
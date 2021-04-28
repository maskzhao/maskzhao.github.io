---
title: Hexo创建博客托管GitHub Pages完全指南
date: 2021-03-25 10:29:45
---

本文介绍基于Hexo创建博客，托管到GitHub Pages。

主要功能有：

- 用Hexo创建博客
- 使用[GitHub Pages](https://pages.github.com/)托管博客
- 使用[GitHub Actions](https://github.com/features/actions)自动部署博客
<!-- - 自定义域名访问GitHub Pages托管的域名 -->
- 为Hexo博客配置自适应主题[fluid](https://github.com/fluid-dev/hexo-theme-fluid)
- 为Hexo博客配置无后端评论插件[valine](https://valine.js.org/)
- 为Hexo博客添加[百度统计](https://tongji.baidu.com/web/welcome/login)
- 为Hexo博客添加[Google Analytics](https://analytics.google.com/analytics/web/)


# 用Hexo创建并使用GitHub Pages托管博客


## 安装Hexo

Hexo基于Nodejs的博客框架，支持Markdown作为源文件，生成静态网页。

- 安装git：前往[git官网](https://git-scm.com/)下载对应操作系统版本完成安装。
- 安装Nodejs： 前往[Nodejs官网](https://nodejs.org/en/)下载对应操作系统版本完成安装。
- 安装Hexo：命令行执行`npm install -g hexo-cli`如提示权限问题可以在命令前添加`sudo`。

## 创建博客项目文件

Hexo安装完成后，命令行进入到要保存项目的目录，执行以下命令`hexo init maskzhao.github.io`（实际操作中用`<username>.github.io`替换`maskzhao.github.io`）：

## 运行博客网站

命令行进入博客目录，执行以下命令在本地启动服务器查看博客效果：

```
npm run server
```

博客启动后浏览器通过`http://localhost:4000`即可访问。

## 新建文章

执行`hexo new <title>`Hexo会使用默认layout在`source/_posts`下创建对应`.md`文档，刷新浏览器可以在文章列表看到新创建的文章，编辑后刷新文章详情可实时预览。

## 生成静态网站

修改`_config.yml`中的`public_dir: public`为`public_dir: docs`，后面配置GitHub Pages托管时只支持根目录或者docs目录，所以这里先修改。

执行`hexo generate`会根据系统配置编译生产静态html文件保存在`docs`文件夹下，将`docs`目录托管到服务器即可正常访问。

此时一个最简单的博客网站就生成了。后续逐步添加功能进行丰富。


# 使用GitHub Pages托管博客

Github Pages可以托管用户或者项目网站，用户网站需要创建格式为`<username>.github.io`的仓库。然后指定网页托管的分支和目录。

1. 创建用户相关仓库
  在github创建名为[maskzhao.github.io](https://github.com/maskzhao/maskzhao.github.io)的仓库
2. 本地编辑博客并提交
  执行`hexo generate`后将项目文件推送到GitHub`master`分支。
3. 设置GitHub Pages托管分支及目录
  浏览器访问[maskzhao.github.io](https://github.com/maskzhao/maskzhao.github.io)，点击`Settings`. 滚动到GitHub Pages区域，托管文件branch选择`master`, 托管目录选择`docs`并保存
  ![](github-pages-setting.png)
4. 访问博客
  浏览器访问[https://maskzhao.github.io/](https://maskzhao.github.io/)


# 使用GitHub Actions自动部署博客

每次写完文章都执行`hexo generate`生成静态文件再推送到GitHub是很繁琐的。GitHub Actions可以实现CI/CD工作流制动完成编译、打包、部署等工作。下面我们通过GitHub Actions实现自动部署博客。


1. 在项目根目录创建`.github/workflows/pages.yml`填写内容如下：
```
name: Pages

on:
  push:
    branches:
      - master  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 12.x
        uses: actions/setup-node@v1
        with:
          node-version: '12.x'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
          publish_branch: gh-pages  # deploying branch
```
GitHub Actions会在master有更新时执行`npm run build`并将成成的`docs`目录作为根目录推送到`gh-pages`分支
![](github-actions-gh-pages.png)

2.修改GitHub Pages配置，设置source为`gh-pages`分支的根目录。
3.修改博客文件，推送到GitHub仓库`master`分支，刷新页面查看是否更新。


<!--
# 自定义域名访问GitHub Pages托管的域名

1.在GitHub Pages设置页"Custom domain（自定义域）"下，输入自定义域，然后单击 Save（保存）。 这将创建一个在发布源根目录中添加 CNAME 文件的提交。
2.在DNS解析设置中创建CNAME记录，使子域指向您站点的默认域。 例如，如果要对您的用户站点使用子域 www.example.com，您可以创建 CNAME 记录，使 www.example.com 指向 <user>.github.io。 -->


# 为Hexo博客配置自适应主题fluid

[fluid](https://github.com/fluid-dev/hexo-theme-fluid)是一款Material Design风格的主题，支持响应式。

1.获取主题最新版本
  Hexo 5.0.0以上版本，通过npm安装，在博客根目录执行以下命令
  ```
  npm install --save hexo-theme-fluid
  ```
  然后在博客目录下创建` _config.fluid.yml`，将主题的[_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml) 内容复制进去。

2.修改hexo配置文件`_config.yml`主题配置
  ```
  theme: fluid  # 指定主题
  language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
  ```

3.重启本地服务，查看效果


# 为Hexo博客配置无后端评论插件valine

[valine](https://valine.js.org/)是一款快速、简洁且高效的无后端评论系统。fluid主题集成了对valine的支持，下面我们基于fluid来添加valine。

1.注册Leancloud账号并认证
  前往[LeanCloud](https://console.leancloud.cn/register)官网注册账号，登录后完成身份认证、邮箱验证。

2.创建应用
  ![](leancloud-create-app.png)

3.在`_config.fluid.yml`中开启评论并配置参数
  ```
    comments:
      enable: true
      type: valine
    valine:
      appid: xxxx
      appkey: xxxx
  ```
  其中的appid和appkey在**应用-》设置-》应用keys**页面查找

  配置完成后重启，文章底部出现评论区。

4.管理评论
  在LeanCloud后台，应用中点击**存储 -> 结构化数据**，选择创建Class，名称Comment可以查看所有评论

<!--todo： 配置阅读数 -->
<!-- todo: 配置waline -->
<!-- todo: hexo支持图片  -->


# 为Hexo博客添加百度统计

fluid主题支持百度统计配置，只需要在百度统计中创建网站，在配置文件中填写对应代码即可。

1.新增网站
  登录[百度统计](https://tongji.baidu.com/web/welcome/login)，进入**管理-》账户管理-》网站列表-》自有网站**，点击**新增网站**填写网站信息。

2.配置秘钥
  在网站列表操作项中点击**代码获取**，复制`hm.src = "https://hm.baidu.com/hm.js?xxxx";`中`.js?`后面的字符串，填写到`_config.fluid.yml`中，同时确保`web_analytics`的`enable`为`true`
  ```
  web_analytics:
    enable: true
    baidu: 209ab21******0a5fc7fb
  ```

3.检查安装
  推送博客源码到GitHub，部署完成后在百度统计后台**管理-》代码管理-》代码安装检查**确认已正确安装。

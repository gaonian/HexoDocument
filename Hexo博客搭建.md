---
title: Hexo博客搭建
date: 2000/7/13 20:46:25
categories: Hexo博客
---



## Hexo

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

首先需要先安装git和node



### 安装Node.js

建议使用Node.js 10.0及以上版本

- Mac

  - mac可以通过homebrew快速安装

  ```
  brew install node
  ```

  - CentOS 下安装

    1. 从官网下载压缩包

       ```
       wget https://nodejs.org/dist/v10.16.0/node-v10.16.0-linux-x64.tar.xz
       ```

    2. 解压

       ```
       tar xf node-v10.16.0-linux-x64.tar.xz
       ```

    3. 解压之后最好修改文件夹名字为node，不要后面的版本号信息。修改完之后进入node文件夹

       ```
       cd node
       ```

       这是操作 `node -v` `npm -v` 查看是否已经好使

    4. 最后一步就是配置系统环境变量，配置之后就可以在任意地方使用 `node` 命令

       ```
       export PATH=$PATH:/xxxx/node/bin
       
       source /etc/profile
       ```

       

### 安装hexo

上述git和node安装完毕之后就可以安装hexo

```
npm install -g hexo-cli
```

至此，hexo就安装完毕了

下一步就是建站，可在任意位置下建立hexo根文件夹



### 建站

```
hexo init 文件夹
cd 文件夹
npm install
```

完成之后，文件夹目录如下:

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

#### _config.yml

网站的配置信息，可以修改此文件完成一些自定义设置。[更多配置详情](https://hexo.io/zh-cn/docs/configuration)

```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: xxx
subtitle: ''
description: ''
keywords:
author: xxx
language: zh-CN
timezone: ''

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: Butterfly

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: 项目地址
  branch: master
```

#### scaffolds

[模版](https://hexo.io/zh-cn/docs/writing) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

Hexo的模板是指在新建的文章文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。

#### source

资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。

#### themes

[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。



### 主题更换

安装完hexo之后，我们一般都开始选用一个自己喜欢的主题配置。 这里以[Butterfly](https://docs.jerryc.me/)为例

1. 首先进入hexo的根目录下

2. 下载Butterfly

   ```
   git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
   ```

3. 修改站点配置文件 _config.yml，把主题改为 Butterfly

   ```
   theme: Butterfly
   ```

> 如果没有 pug 以及 stylus 的渲染器，请下载安装： `npm install hexo-renderer-pug hexo-renderer-stylus --save` or `yarn add hexo-renderer-pug hexo-renderer-stylus`

推荐把主题默认的配置文件`_config.yml`复制到 Hexo 工作目录下的`source/_data/butterfly.yml`，如果`source/_data`的目录不存在那就创建一个。

这里有两个配置文件说明一下

_config.yml 是hexo的配置文件，在hexo跟目录下

butterfly.yml 是主题的配置文件，在这个配置文件内可以修改主题的一些设置

[Butterfly具体配置]([https://docs.jerryc.me/config.html#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AA%AC%E6%98%8E](https://docs.jerryc.me/config.html#配置文件説明))



### 常用命令

#### generate

```
$ hexo generate
```

生成静态文件。

| 选项                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `-d`, `--deploy`      | 文件生成后立即部署网站                                       |
| `-w`, `--watch`       | 监视文件变动                                                 |
| `-b`, `--bail`        | 生成过程中如果发生任何未处理的异常则抛出异常                 |
| `-f`, `--force`       | 强制重新生成文件 Hexo 引入了差分机制，如果 `public` 目录存在，那么 `hexo g` 只会重新生成改动的文件。 使用该参数的效果接近 `hexo clean && hexo generate` |
| `-c`, `--concurrency` | 最大同时生成文件的数量，默认无限制                           |

该命令可以简写为

```
$ hexo g
```

#### server

```
hexo server
hexo server --debug
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

| 选项             | 描述                           |
| :--------------- | :----------------------------- |
| `-p`, `--port`   | 重设端口                       |
| `-s`, `--static` | 只使用静态文件                 |
| `-l`, `--log`    | 启动日记记录，使用覆盖记录格式 |

#### deploy

```
hexo deploy
```

部署网站。

| 参数               | 描述                     |
| :----------------- | :----------------------- |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

该命令可以简写为：

```
hexo d
```

#### clean

```
hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。



### Front-matter

Front-matter 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```
---
title: Hello World
date: 2013/7/13 20:46:25
categories: iOS
---
```

以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数         | 描述                                                 | 默认值       |
| :----------- | :--------------------------------------------------- | :----------- |
| `layout`     | 布局                                                 |              |
| `title`      | 标题                                                 | 文章的文件名 |
| `date`       | 建立日期                                             | 文件建立日期 |
| `updated`    | 更新日期                                             | 文件更新日期 |
| `comments`   | 开启文章的评论功能                                   | true         |
| `tags`       | 标签（不适用于分页）                                 |              |
| `categories` | 分类（不适用于分页）                                 |              |
| `permalink`  | 覆盖文章网址                                         |              |
| `keywords`   | 仅用于 meta 标签和 Open Graph 的关键词（不推荐使用） |              |



### 部署配置

Hexo 提供了快速方便的一键部署功能，让您只需一条命令就能将网站部署到服务器上。

```
hexo deploy
```

在开始之前，您必须先在 `_config.yml` 中修改参数，一个正确的部署配置中至少要有 `type` 参数，例如：

```
deploy:
  type: git
```

您可同时使用多个 deployer，Hexo 会依照顺序执行每个 deployer。

```
deploy:
- type: git
  repo:
- type: heroku
  repo:
```

#### git

1. 安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)。

```
$ npm install hexo-deployer-git --save
```

1. 修改配置。

```
deploy:
  type: git
  repo: <repository url> #https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
```

| 参数      | 描述                                                         | 默认                                                         |
| :-------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `repo`    | 库（Repository）地址                                         |                                                              |
| `branch`  | 分支名称                                                     | `gh-pages` (GitHub) `coding-pages` (Coding.net) `master` (others) |
| `message` | 自定义提交信息                                               | `Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}`)            |
| `token`   | Optional token value to authenticate with the repo. Prefix with `$` to read token from environment variable |                                                              |

1. 生成站点文件并推送至远程库。执行`hexo clean && hexo deploy`
   - You will be prompted with username and password of the target repository, unless you authenticate with a token or ssh key.
   - hexo-deployer-git does not store your username and password. Use [git-credential-cache](https://git-scm.com/docs/git-credential-cache) to store them temporarily.
2. 登入 Github/BitBucket/Gitlab，请在库设置（Repository Settings）中将默认分支设置为`_config.yml`配置中的分支名称。稍等片刻，您的站点就会显示在您的Github Pages中。

##### 这一切是如何发生的？

当执行 `hexo deploy` 时，Hexo 会将 `public` 目录中的文件和目录推送至 `_config.yml` 中指定的远端仓库和分支中，并且**完全覆盖**该分支下的已有内容。

> For 使用 Git 管理站点目录的用户
>
> 由于 Hexo 的部署默认使用分支 `master`，所以如果你同时正在使用 Git 管理你的站点目录，你应当注意你的部署分支应当不同于写作分支。
> 一个好的实践是将站点目录和 Pages 分别存放在两个不同的 Git 仓库中，可以有效避免相互覆盖。
> Hexo 在部署你的站点生成的文件时并不会更新你的站点目录。因此你应该手动提交并推送你的写作分支。

**此外，如果您的 Github Pages 需要使用 CNAME 文件自定义域名，请将 CNAME 文件置于 `source` 目录下，只有这样 `hexo deploy` 才能将 CNAME 文件一并推送至部署分支。**



### 生成并部署

您可执行下列的其中一个命令，让 Hexo 在生成完毕后自动部署网站，两个命令的作用是相同的。

```
$ hexo generate --deploy
$ hexo deploy --generate
```

> 简写
>
> 上面两个命令可以简写为
> hexo g -d
> hexo d -g





## Github Pages

### 创建github pages

1. github创建仓库 `xxx.github.io` ，仓库的命名要严格按照这个规则

2. clone仓库到本地

   ```
   git clone https://github.com/xxx/xxx.github.io
   ```

3. 新建index.md，提交

   ```
   cd xxx.gitbub.io
   echo "hello world" > index.md
   
   git commit -am 'init'
   git push
   ```

4. 完成之后去浏览器测试，输入 `https://xxx.github.io`，如果看到页面上显示hello world则代表成功。如果没有稍等片刻再试

   

### Hexo + Github Pages

 在hexo那一块已经提到要设置deploy的git信息

```
deploy:
  type: git
  repo: <repository url>  // 刚创建仓库的url https://github.com/xxx/xxx.github.io
  branch: [branch]
  message: [message]
```

确认配置无误的话，执行 `hexo g -d` 就可以看到会把hexo的信息推送到指定的仓库中，注意是完全覆盖信息，所以需要保留原有信息的要设置好branch

如果成功的话，再在浏览器刷新 `https://xxx.github.io` ，就可以看到已经部署的hexo。



## 云服务器

除了把hexo部署到github pages上之外，我们还可以部署到自己的服务器上。

### 1. 安装Nginx

- #### 通过yum安装

  ```
  sudo yum -y install nginx
  ```

  通过yum安装 nginx操作指令

  ```
  $ sudo systemctl enable nginx # 设置开机启动 
  $ sudo service nginx start # 启动nginx服务
  $ sudo service nginx stop # 停止nginx服务
  $ sudo service nginx restart # 重启nginx服务
  $ sudo service nginx reload # 重新加载配置，一般是在修改过nginx配置文件时使用。
  ```

  

- #### 源码安装

  1. 先安装依赖库

     ```
     $ sudo yum -y install gcc gcc-c++ # nginx编译时依赖gcc环境
     $ sudo yum -y install pcre pcre-devel # 让nginx支持重写功能
     $ sudo yum -y install zlib zlib-devel 
     $ sudo yum -y install openssl openssl-devel
     ```

  2. 下载nginx的压缩包，上传到服务器

  3. ```
     tar -zxvf nginx-1.11.5.tar.gz
     ```

  4. ```
     cd nnginx-1.11.5
     ./configure --prefix=/usr/local/nginx
     
     --prefix=/usr/local/nginx  是nginx编译安装的目录（推荐），安装完后会在此目录下生成相关文件
     ```

  5. ```
     make
     make install
     ```

  通过nginx源码安装操作命令

  ```
  // 启动服务
  nginx 
  
  // 重新加载服务
  nginx -s reload
  
  // 停止服务
  nginx -s stop
  ```

  

### 2. 安装Hexo，安装步骤同上

### 3. 配置conf

找到nginx的配置文件 `nginx.conf` ，通过上述源码安装的，配置文件在 `/usr/local/nginx/conf/` 目录下。

```
listen       80;
server_name  localhost;

#charset koi8-r;

#access_log  logs/host.access.log  main;

location / {
    root   hexoDoc/public;
    index  index.html index.htm;
}

```

修改nginx的代理目录，这里配置为hexo目录下的public文件夹

### 4. 重启nginx

重启nginx，在浏览器输入服务器ip，即可看到配置的hexo



## 域名绑定

上述介绍了两种方式去部署hexo，两种方式的访问如下：

- github pages的访问url 是 `https://xxx.github.io`

- 服务器的访问就是直接使用服务器的ip地址

试想，如果我们使用的是自己的域名来访问，岂不是很爽。 域名的购买途径有很多，我们可以买一个自己喜欢的便宜的来使用

### Github Pages

如果我们用的是github pages，这时候我们就需要在hexo根目录下的`source`文件夹内创建一个`CNAME` 文件内，文件内容就是我们的域名。然后执行 `hexo g -d` ，hexo会把CNAME文件放到public文件夹内上传到github，github会自动识别到CNAME文件内的域名。在设置内Custom domain我们可以看到已经识别了域名

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/image/hexo1.png)



接下来需要在域名的后台去配置域名解析。注意类型是CNAME，后边直接写上我们的xxx.github.io 就可以了

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/image/hexo2.png)



稍等片刻，我们就可以通过域名去访问了



### 服务器

如果是服务器，则更简单，直接在域名后台解析配置的时候，类型选择 `A记录`，值直接使用服务器的ip地址即可



## 参考资料

[hexo](https://hexo.io/zh-cn/docs/setup)

[Butterfly主题](https://docs.jerryc.me/)

[Gitbub Pages](https://help.github.com/en/github/working-with-github-pages)

[CentOS Nginx安装](https://blog.csdn.net/sqlquan/article/details/101099850)




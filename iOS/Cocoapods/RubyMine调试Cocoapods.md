## Ruby

首先查看本地Ruby环境，系统自带的版本可能有些老，建议安装ruby新版本，这里使用的是2.7.2

对于只是简单使用ruby，不做版本兼容的用户来说。建议使用brew安装ruby

```
➜  ~ ruby -v
ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-darwin19]
```

- 通过homebrew安装ruby

  ```
  brew install ruby@2.7
  ```

  安装成功之后，通过 `ruby -v` 查看版本信息，若还是老版本，则需要添加系统环境变量

  根据自己的情况，选择 `.zshrc` 或者 `.bash_profile`，加入下面一行

  ```
  export PATH="/usr/local/Cellar/ruby@2.7/2.7.2/bin:$PATH"
  ```

  生效

  ```
  source ~/.zhsrc
  或
  source ~/.bash_profile
  ```

- 通过rvm安装ruby

  - 安装rv'm

    ```
    curl -sSL https://get.rvm.io | bash -s stable
    ```

  - 安装ruby

    ```
    rvm install ruby-2.7.2
    ```

  - 切换版本

    ```
    rvm 2.7.2 -- current && default
    ```



## Cocoapods源码

- 新建文件夹，这里以 `rubyDebug` 为例

- 下载cocoapods源码

  ```
  git clone https://github.com/CocoaPods/CocoaPods.git
  ```

  由于我这里需要调试的是1.9.3版本的。所以在下载完成之后切换到对应tag

  ```
  git checkout 1.9.3
  ```

- 进入cocoapods文件夹，执行 `bundle install`，安装gem组件

  ```
  bundle install
  ```

  这一步可能会耗时较长，也可能会出现一些组件下载失败的情况，有两种解决方案:

  1. 重试`bundle install`

  2. 针对失败的组件，选择对应的版本自行从github下载，并修改Gemfile的依赖关系，改为本地库。

     举例：

     比如在下载 `cocoapods-core ` 失败了，此时通过cocoapods文件夹下的Gemfile.lock看到依赖的版本为1.9.3

     ```
     cocoapods-core (= 1.9.3)
     ```

     所以我去github下载对应的库到本地，放在同一目录下。此时目录结构如下，Core-1.9.3为刚下载的core库

     ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05edbc29035c4873af4cb36335abfc53~tplv-k3u1fbpfcp-watermark.image)

     然后修改Cocoapods文件下的Gemfile文件

     ```ruby
     group :development do
       cp_gem 'claide',                'CLAide'
       cp_gem 'cocoapods-core',        'Core-1.9.3', path: true
       cp_gem 'cocoapods-deintegrate', 'cocoapods-deintegrate'
       cp_gem 'cocoapods-downloader',  'cocoapods-downloader', path: true
       cp_gem 'cocoapods-plugins',     'cocoapods-plugins'
       cp_gem 'cocoapods-search',      'cocoapods-search'
       cp_gem 'cocoapods-stats',       'cocoapods-stats'
       cp_gem 'cocoapods-trunk',       'cocoapods-trunk'
       cp_gem 'cocoapods-try',         'cocoapods-try'
       cp_gem 'molinillo',             'Molinillo'
       cp_gem 'nanaimo',               'Nanaimo'
       cp_gem 'xcodeproj',             'Xcodeproj-1.16.0', path: true
     ```

     把第二个参数repo_name修改为自己下载的文件夹名称，比如我的为`Core-1.9.3`，后面新增参数 `path: true`，意思为从本地查找文件。

     完成之后继续执行 `bundle install`，看到例如以下信息就成功了

     ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1a77eac3fe44493ac54a2f5811041a7~tplv-k3u1fbpfcp-watermark.image)

  在这个步骤可能会遇到的问题比较多，哪个库有问题实在不行就尝试自己下载到本地重试。需有耐心！



## RubyMine配置

1. 从RubyMine打开rubyDebug文件夹

   ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39f4077d026147c080d9db51837cf679~tplv-k3u1fbpfcp-watermark.image)

2. 配置debug信息，点击右上角 Add Configuration -> 添加 -> Ruby

   ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2892579aea2a487db5b55d86fd8892cb~tplv-k3u1fbpfcp-watermark.image)

   ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a86c05ddade4404af7ba215bd943a5a~tplv-k3u1fbpfcp-watermark.image)

   - `Run script`

     脚本路径，由于是本地调试，所以选择自己的Cocoapods/bin下的pod命令

   - `Script arguments`

     脚本的参数。这里填写 install，编译器就会找到上一步填写的pod，然后执行 `pod install`。若需要调试其他执行，修改这里即可

   - `Working directory`

     命令在何处执行。这里选择Cocoapods默认给的example，选择afn。运行时会在此目录下执行 `pod install`

   - `Ruby SDK`

     这里选择我们自己安装的 ruby-2.7.2。

     若这里没有我们下载的ruby。打开 Preferences，搜索 `Ruby SDK and Gems`，新增自己下载的ruby路径

     ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6243e65f46624a9eaf2c2ed310234ce4~tplv-k3u1fbpfcp-watermark.image)

3. 最后应用配置Apply，点击ok，完成配置



## 调试

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40b0f28c921e4084a38d4af7dda0a87b~tplv-k3u1fbpfcp-watermark.image)

1. 完成上述配置之后，按上图的位置，在install.rb中打上断点

2. 点击右上角debug按钮，会自动在上述配置的工作目录下执行命令 `xxxx/Cocoapods/bin/pod install`。断点执行，成功

![ ](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88a79a5b734c44bcb50595c75ad04ae6~tplv-k3u1fbpfcp-watermark.image) 




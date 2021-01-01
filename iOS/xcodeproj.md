# Xcodeproj

[Xcodeproj](https://github.com/CocoaPods/Xcodeproj) 通过Ruby创建和修改Xcode项目结构。同样支持对 Xcode workspaces（.xcworkspace）、配置文件（.xcconfig）和 Xcode Scheme文件（.xcscheme）的修改

[API Reference](https://www.rubydoc.info/gems/xcodeproj)



## 1. 安装

```
sudo gem install xcodeproj
```



## 2. 使用

### 1. open

``` ruby
require 'xcodeproj'
project = Xcodeproj::Project.open("KBTEST/KBTEST.xcodeproj")
...
...
...
project.save
```

首先通过xcodeproj路径打开文件，内部会生成对应的Project类，所有信息都在此类中。若进行了修改，最后调用save方法保存，重新生成.pbxproj文件



### 2. 获取所有target

``` ruby
# target
project.targets.each do |target|
  puts target.name
end
```



### 3. 获取target的source files

``` ruby
# target source files
target = project.targets.first
files = target.source_build_phase.files.to_a.map do |pbx_build_file|
  pbx_build_file.file_ref.real_path.to_s

end.select do |path|
  path.end_with?(".m", ".mm", ".swift")

end.select do |path|
  File.exists?(path)
end

puts files
```



### 4. 获取target的build configuration

``` ruby
# target build setting
project.targets.each do |target|
  target.build_configurations.each do |config|
    if config.name == "Debug"
      puts config.build_settings
    end
  end
end
```

获取debug模式下的所有build_settings，可以自行进行修改配置或者新增配置

``` ruby
# 修改描述文件名称
config.build_settings["PROVISIONING_PROFILE_SPECIFIER"] = "xxProfileName"

# 新增配置
config.build_settings['MY_CUSTOM_FLAG'] ||= 'TRUE'
```



### 5. 引入类文件到工程中

``` ruby
# 添加文件
new_path = File.join("KBTEST","New")
group = project.main_group.find_subpath(new_path, true )
group.set_source_tree("<group>")
group.set_path("New")

file_ref = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.h"))
file_ref1 = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.m"))

target = project.targets.first
target.add_file_references([file_ref1])

project.save
```

新建一个名称为New的group，放在maingroup下

添加a.h、a.m文件到New group下

由于.m文件要进行编译，所以需要执行`add_file_references` ，添加到buildFile和sourcefiles。.h文件只用调用`new_reference`添加文件引用即可



# 源码分析

## 1. open

``` ruby
require 'xcodeproj'
project = Xcodeproj::Project.open("KBTEST/KBTEST.xcodeproj")
...
...
...
project.save
```

首先需要传入工程路径，调用Project类的open方法

open方法定义在`lib/project.rb` 文件中

``` ruby
def self.open(path)
  path = Pathname.pwd + path
  unless Pathname.new(path).exist?
    raise "[Xcodeproj] Unable to open `#{path}` because it doesn't exist."
  end
  project = new(path, true)
  project.send(:initialize_from_file)
  project
end
```

1. 首先拼接绝对路径，根据路径判断文件是否存在

2. 创建project对象，第一个参数为path，第二个参数skip_initialization = true，说明不需要从头进行初始化，因为接下来会根据path去创建初始化root_object对象

   ``` ruby
   def initialize(path, skip_initialization = false, object_version = Constants::DEFAULT_OBJECT_VERSION)
     @path = Pathname.new(path).expand_path
     @project_dir = @path.dirname
     @objects_by_uuid = {}
     @generated_uuids = []
     @available_uuids = []
     @dirty           = true
     unless skip_initialization.is_a?(TrueClass) || skip_initialization.is_a?(FalseClass)
       raise ArgumentError, '[Xcodeproj] Initialization parameter expected to ' \
         "be a boolean #{skip_initialization}"
     end
     unless skip_initialization
       initialize_from_scratch
       @object_version = object_version.to_s
       unless Constants::COMPATIBILITY_VERSION_BY_OBJECT_VERSION.key?(object_version)
         raise ArgumentError, "[Xcodeproj] Unable to find compatibility version string for object version `#{object_version}`."
       end
       root_object.compatibility_version = Constants::COMPATIBILITY_VERSION_BY_OBJECT_VERSION[object_version]
     end
   end
   ```

3. 初始化root_object对象，对应的是PBXProject对象

   ``` ruby
   # Initializes the instance with the project stored in the `path` attribute.
       #
   def initialize_from_file
     pbxproj_path = path + 'project.pbxproj'
     plist = Plist.read_from_path(pbxproj_path.to_s)
     root_object.remove_referrer(self) if root_object
     @root_object     = new_from_plist(plist['rootObject'], plist['objects'], self)
     @archive_version = plist['archiveVersion']
     @object_version  = plist['objectVersion']
     @classes         = plist['classes'] || {}
     @dirty           = false
   
     unless root_object
       raise "[Xcodeproj] Unable to find a root object in #{pbxproj_path}."
     end
   
     if archive_version.to_i > Constants::LAST_KNOWN_ARCHIVE_VERSION
       raise '[Xcodeproj] Unknown archive version.'
     end
   
     if object_version.to_i > Constants::LAST_KNOWN_OBJECT_VERSION
       raise '[Xcodeproj] Unknown object version.'
     end
   
     # Projects can have product_ref_groups that are not listed in the main_groups["Products"]
     root_object.product_ref_group ||= root_object.main_group['Products'] || root_object.main_group.new_group('Products')
   end
   ```

   `plist = Plist.read_from_path(pbxproj_path.to_s)` 内部会通过 `Nanaimo::Reader.new(contents).parse!.as_ruby` 把pbx文件转换为ruby的hash字典类型

      ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3154ad4d768f4ca6a4bd4f013a5f5f40~tplv-k3u1fbpfcp-watermark.image)

   然后通过 `new_from_plist` 方法生成root_object对象。这一步就把pbx文件所有内容都一一映射为root_object对象，从此对象中可以获取到任意想要的内容

      ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38d975d045b14dca8d349ec13cfa4f6f~tplv-k3u1fbpfcp-watermark.image)

   - build_configuration_list 对应的是 XCConfigurationList，内部包含的是所有的XCBuildConfiguration
   - main_group对应的是 PBXGroup，工程的主group
   - product_ref_group 对应的是包文件对应的group
   - simple_attributes_hash 对应的是一些其他的简单配置项
   - targets 对应的就是项目内所有的target，PBXNativeTarget
   - ...

至此，open的逻辑基本分析完毕。总结一下就是通过xcodeproj路径把pbx文件转换为ruby对应的对象。最后来看一下project类初始化完毕后的全部属性

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f504c17e7e9c4fdf98aaf09ccb322b98~tplv-k3u1fbpfcp-watermark.image)



## 2. 获取target

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/388f78b3e7784f9db1a25fbdb5d44b41~tplv-k3u1fbpfcp-watermark.image)

``` ruby
# target
project.targets.each do |target|
  puts target.name
end
```

1. 调用project对象的targets方法，返回一个target抽象类数组

   ``` ruby
   # @return [ObjectList<AbstractTarget>] A list of all the targets in the
   #         project.
   #
   def targets
     root_object.targets
   end
   ```

2. 遍历targets数组，这里只是打印出每一个target的name。可以根据自己的需求进行不同的操作

      ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e82439c07d93415ca6409327b6ff9873~tplv-k3u1fbpfcp-watermark.image)



## 3. 获取target的souce files

``` ruby
# target source files
target = project.targets.first
files = target.source_build_phase.files.to_a.map do |pbx_build_file|
  pbx_build_file.file_ref.real_path.to_s

end.select do |path|
  path.end_with?(".m", ".mm", ".swift")

end.select do |path|
  File.exists?(path)
end

puts files
```

这里大体分为三步

1. 首先通过 `target.source_build_phase.files` 获取此target下的build_file ，然后获取到build_file 的 file_ref。最后获取到file_ref的真实路径
2. 只筛选出`.m .mm .swift` 结尾的文件，说明这是我们自己编写的代码文件
3. 最后再根据路径判断是否是真实存在的文件

我们主要来分析第一步，如何获取到target下的build_file。了解pbx格式的就知道获取source files就是要获取PBXBuildPhase下的 PBXSourcesBuildPhase 段的内容，通过PBXSourcesBuildPhase下的files数组可以找到对应的PBXBuildFile，然后就可以找到PBXFileReference。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a67b4fe88e754e989dedf0bb04a9b467~tplv-k3u1fbpfcp-watermark.image)

下图是ruby下的build_phase结构

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbeddb3e8f3e4863bf766a75f8cba107~tplv-k3u1fbpfcp-watermark.image)

`target.source_build_phase` 就是获取上图中build_phase对象

`target.source_build_phase.files` 是获取到 build_phase 对象中的files数组，对files数组进行遍历，其中每一个对象对PBXBuildFile

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f02208351ad4a29b2b76f148112cb61~tplv-k3u1fbpfcp-watermark.image)

`pbx_build_file.file_ref` 就是获取上图中的 PBXFileReference 对象

最后调用 `real_path.to_s` 获取到真实路径转换为字符串形式



## 4. 获取target的build configuration

``` ruby
# target build setting
project.targets.each do |target|
  target.build_configurations.each do |config|
    if config.name == "Debug"
      puts config.build_settings
    end
  end
end
```

获取每一个target Debug环境下的buildSettings

调用 `target.build_configurations` ，获取到 XCConfigurationList，接着获取到 XCBuildConfiguration

```ruby
# @return [ObjectList<XCBuildConfiguration>] the build
#         configurations of the target.
#
def build_configurations
  build_configuration_list.build_configurations
end
```

遍历中config此时就是 XCBuildConfiguration 类型，通过XCBuildConfiguration中的属性build_settings获取到所有配置

```ruby
# @return [Hash] the build settings to use for building the target.
#
attribute :build_settings, Hash, {}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da9a35c3f1274a648b765940d39f925a~tplv-k3u1fbpfcp-watermark.image)



## 5. 引入类文件到工程中

在KBTEST下新建New，添加a.h，a.m到New group下，代码如下：

```ruby
# 添加文件
new_path = File.join("KBTEST","New")
group = project.main_group.find_subpath(new_path, true )
group.set_source_tree("<group>")
group.set_path("New")

file_ref = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.h"))
file_ref1 = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.m"))

target = project.targets.first
target.add_file_references([file_ref1])


project.save
```



首先，我们在xcode下主动新建一个group，命名为New1。 在New1下新建new1.h 、new1.m文件。然后观察pbxproj文件的变化

1. 在PBXGroup下新增了New1对应的模块，并在 KBTEST（ mainGroup）children下新增了对 New1的引用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/181cd3ed50b248b58d8615c00c155077~tplv-k3u1fbpfcp-watermark.image)

2. 在 PBXFileReference 段下，新增了两项，代表了 new1.h、new1.m。所有添加的文件都会进入到此模块中

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18fe606ec3fd4220baeac4c02250cb29~tplv-k3u1fbpfcp-watermark.image)

3. 由于.m文件要参与编译，所以在 PBXBuildFile 下会新增一项new1.m，具体引用的是 PBXFileReference 下new1.m文件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fb85ae4c77146049e6c3fb7bbffa33c~tplv-k3u1fbpfcp-watermark.image)

4. 在 PBXSourcesBuildPhase 下会新增一项new1.m，这里new1.m引用的是上面 PBXBuildFile中new1.m文件。（PBXSourcesBuildPhase对应的是xcode中 Build Phase下的 Compile Sources）

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79a7991b6bf345d2aae1b84cf57c6eb3~tplv-k3u1fbpfcp-watermark.image)

通过上述步骤我们来分析一下代码的每一步都做了什么？



1. **在KBTEST（mainGroup）下新建New（Group），设置source_tree和path，设置方式可以参考KBTEST，source_tree为`<group>`**

   ``` ruby
   new_path = File.join("KBTEST","New")
   group = project.main_group.find_subpath(new_path, true )
   group.set_source_tree("<group>")
   group.set_path("New")
   ```

   ```
   5E785D438559401E397AB673 /* New */ = {
     isa = PBXGroup;
     children = (
       41B83A50825CA76C4A7D57D7 /* a.h */,
       BBCB1AF3369FC78AEC34D2F2 /* a.m */,
     );
     name = New;
     path = New;
     sourceTree = "<group>";
   };
   ```

   

2. **添加a.h、a.m的文件引用。path路径为绝对路径**

   ``` ruby
   file_ref = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.h"))
   file_ref1 = group.new_reference(File.join(project.project_dir, "/KBTEST/New/a.m"))
   ```

   ```
   /* Begin PBXFileReference section */
   
     41B83A50825CA76C4A7D57D7 /* a.h */ = {isa = PBXFileReference; includeInIndex = 1; lastKnownFileType = sourcecode.c.h; path = a.h; sourceTree = "<group>"; };
     BBCB1AF3369FC78AEC34D2F2 /* a.m */ = {isa = PBXFileReference; includeInIndex = 1; lastKnownFileType = sourcecode.c.objc; path = a.m; sourceTree = "<group>"; };
   
   /* End PBXFileReference section */
   ```

   

   `new_file_reference`定义在`file_references_factory.rb`文件中

   ``` ruby
   def new_file_reference(group, path, source_tree)
     path = Pathname.new(path)
     ref = group.project.new(PBXFileReference)
     group.children << ref
     GroupableHelper.set_path_with_source_tree(ref, path, source_tree)
     ref.set_last_known_file_type
     ref
   end
   ```

   从上述代码中可以看出首先会创建 PBXFileReference文件，然后加入到group的children下，最后返回PBXFileReference引用对象

   

3. **添加.m文件到目标target的PBXBuildFile和PBXSourcesBuildPhase下，参与编译。**

   ``` ruby
   target = project.targets.first
   target.add_file_references([file_ref1])
   ```

   这里选择的是第一个target。调用`add_file_references()`方法

   ``` ruby
   # Adds source files to the target.
   #
   # @param  [Array<PBXFileReference>] file_references
   #         the files references of the source files that should be added
   #         to the target.
   #
   # @param  [String] compiler_flags
   #         the compiler flags for the source files.
   #
   # @yield_param [PBXBuildFile] each created build file.
   #
   # @return [Array<PBXBuildFile>] the created build files.
   #
   def add_file_references(file_references, compiler_flags = {})
     file_references.map do |file|
       extension = File.extname(file.path).downcase
       header_extensions = Constants::HEADER_FILES_EXTENSIONS
       is_header_phase = header_extensions.include?(extension)
       phase = is_header_phase ? headers_build_phase : source_build_phase
   
       unless build_file = phase.build_file(file)
         build_file = project.new(PBXBuildFile)
         build_file.file_ref = file
         phase.files << build_file
       end
   
       if compiler_flags && !compiler_flags.empty? && !is_header_phase
         (build_file.settings ||= {}).merge!('COMPILER_FLAGS' => compiler_flags) do |_, old, new|
           [old, new].compact.join(' ')
         end
       end
   
       yield build_file if block_given?
   
       build_file
     end
   end
   ```

   - 第一步首先根据文件名称判断类型是 PBXHeadersBuildPhase 或者 PBXSourcesBuildPhase

     ``` ruby
     extension = File.extname(file.path).downcase
     header_extensions = Constants::HEADER_FILES_EXTENSIONS
     is_header_phase = header_extensions.include?(extension)
     phase = is_header_phase ? headers_build_phase : source_build_phase
     ```

     ``` ruby
     # Finds or creates the source build phase of the target.
     #
     # @note   A target should have only one source build phase.
     #
     # @return [PBXSourcesBuildPhase] the source build phase.
     #
     def source_build_phase
       find_or_create_build_phase_by_class(PBXSourcesBuildPhase)
     end
     
     
     # @!group Internal Helpers
     #--------------------------------------#
     
     # Find or create a build phase by a given class
     #
     # @param [Class] phase_class the class of the build phase to find or create.
     #
     # @return [AbstractBuildPhase] the build phase whose class match the given phase_class.
     #
     def find_or_create_build_phase_by_class(phase_class)
       @phases ||= {}
       unless phase_class < AbstractBuildPhase
         raise ArgumentError, "#{phase_class} must be a subclass of #{AbstractBuildPhase.class}"
       end
       @phases[phase_class] ||= build_phases.find { |bp| bp.class == phase_class } ||
         project.new(phase_class).tap { |bp| build_phases << bp }
     end
     ```

     这里的目的是找到或者新创建 PBXSourcesBuildPhase，然后返回

     ​     ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8350c9bdaa1f47208aaa0de8a45f24e8~tplv-k3u1fbpfcp-watermark.image)

   - 第二步根据phase查找是否内部有对应的file。如果未找到，则新建PBXBuildFile，并添加到phase的files内部

     ``` ruby
     unless build_file = phase.build_file(file)
       build_file = project.new(PBXBuildFile)
       build_file.file_ref = file
       phase.files << build_file
     end
     
     
     # @return [PBXBuildFile] the first build file associated with the given
     #         file reference if one exists.
     #
     def build_file(file_ref)
       (file_ref.referrers & files).first
     end
     ```

   - 最后一步处理外部的块调用yield传参也为build_file。返回PBXBuildFile

     ``` ruby
     yield build_file if block_given?
     
     build_file
     ```

     

4. **保存上述对工程进行的修改**

   使用初始化期间提供的path或参数传入的path（xcodeproj文件），以xcodeproj格式序列化项目。
   如果提供了path，则依赖于项目根的文件引用不会自动更新，因此客户端负责在保存之前执行任何所需的修改。

   ``` ruby
   # Serializes the project in the xcodeproj format using the path provided
   # during initialization or the given path (`xcodeproj` file). If a path is
   # provided file references depending on the root of the project are not
   # updated automatically, thus clients are responsible to perform any needed
   # modification before saving.
   #
   # @param  [String, Pathname] path
   #         The optional path where the project should be saved.
   #
   # @example Saving a project
   #   project.save
   #   project.save
   #
   # @return [void]
   #
   def save(save_path = nil)
     save_path ||= path
     @dirty = false if save_path == path
     FileUtils.mkdir_p(save_path)
     file = File.join(save_path, 'project.pbxproj')
     Atomos.atomic_write(file) do |f|
       Nanaimo::Writer::PBXProjWriter.new(to_ascii_plist, :pretty => true, :output => f, :strict => false).write
     end
   end
   ```

   主要是调用`Nanaimo::Writer::PBXProjWriter.new(to_ascii_plist, :pretty => true, :output => f, :strict => false).write` 重新写pbxproj文件

   [Nanaimo ](https://github.com/CocoaPods/Nanaimo) 是一个实现ASCII Plist序列化和反序列化的简单库，完全使用原生Ruby代码（并且没有依赖关系）。它还提供了对序列化Xcode projects（带注释）和XML plist的现成支持。在open和save中都使用了此库来操作pbxproj文件



# 参考资料

- [Xcodeproj](https://github.com/CocoaPods/Xcodeproj)
- [API Reference](https://www.rubydoc.info/gems/xcodeproj)
- [Xcode Project File Format](http://www.monobjc.net/xcode-project-file-format.html)
- [ruby库xcodeproj使用心得](https://www.jianshu.com/p/cca701e1d87c)

- [XcodeProject的内部结构分析](https://www.jianshu.com/p/50cc564b58ce)
- [使用代码为 Xcode 工程添加文件](https://draveness.me/bei-xcodeproj-keng-de-zhe-ji-tian/)


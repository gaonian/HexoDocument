# Xcodeproj

[Xcodeproj](https://github.com/CocoaPods/Xcodeproj) 通过Ruby创建和修改Xcode项目结构。同样支持对 Xcode workspaces（.xcworkspace）、配置文件（.xcconfig）和 Xcode Scheme文件（.xcscheme）的修改

[API Reference](https://www.rubydoc.info/gems/xcodeproj)



## 安装

```
sudo gem install xcodeproj
```



## 使用





# 参考资料

- [Xcodeproj](https://github.com/CocoaPods/Xcodeproj)
- [API Reference](https://www.rubydoc.info/gems/xcodeproj)
- [Xcode Project File Format](http://www.monobjc.net/xcode-project-file-format.html)
- [ruby库xcodeproj使用心得](https://www.jianshu.com/p/cca701e1d87c)

- [XcodeProject的内部结构分析](https://www.jianshu.com/p/50cc564b58ce)
- [使用代码为 Xcode 工程添加文件](https://draveness.me/bei-xcodeproj-keng-de-zhe-ji-tian/)





- project.rb

  

## Xcodeproj

### Project

#### Object

##### AbstractBuildPhase

```
lib/xcodeproj/project/object/build_phase.rb
```

已知子类：

[PBXCopyFilesBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXCopyFilesBuildPhase), [PBXFrameworksBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXFrameworksBuildPhase), [PBXHeadersBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXHeadersBuildPhase), [PBXResourcesBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXResourcesBuildPhase), [PBXRezBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXRezBuildPhase), [PBXShellScriptBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXShellScriptBuildPhase), [PBXSourcesBuildPhase](https://www.rubydoc.info/gems/xcodeproj/Xcodeproj/Project/Object/PBXSourcesBuildPhase)



##### XCConfigurationList



##### PBXGroup

```
lib/xcodeproj/project/object/group.rb
```

```ruby
# Creates a new reference with the given path and adds it to the
# group. The reference is configured according to the extension
# of the path.
#
# @param  [#to_s] path
#         The, preferably absolute, path of the reference.
# 				最好是绝对路径
#
# @param  [Symbol] source_tree
#         The source tree key to use to configure the path (@see
#         GroupableHelper::SOURCE_TREES_BY_KEY).
#
# @return [PBXFileReference, XCVersionGroup] The new reference.
#
def new_reference(path, source_tree = :group)
  FileReferencesFactory.new_reference(self, path, source_tree)
end
alias_method :new_file, :new_reference
```



```ruby
def new_reference(group, path, source_tree)
  ref = case File.extname(path).downcase
        when '.xcdatamodeld'
          new_xcdatamodeld(group, path, source_tree)
        when '.xcodeproj'
          new_subproject(group, path, source_tree)
        else
          new_file_reference(group, path, source_tree)
        end

  configure_defaults_for_file_reference(ref)
  ref
end
```



``` ruby
# Creates a new file reference with the given path and adds it to the
# given group.
# 使用给定的path创建新的文件引用，并且加入到指定的group
#
# @param  [PBXGroup] group
#         The group to which to add the reference.
#
# @param  [#to_s] path
#         The, preferably absolute, path of the reference.
#
# @param  [Symbol] source_tree
#         The source tree key to use to configure the path (@see
#         GroupableHelper::SOURCE_TREES_BY_KEY).
#
# @return [PBXFileReference] The new file reference.
#
def new_file_reference(group, path, source_tree)
  path = Pathname.new(path)
  ref = group.project.new(PBXFileReference)
  group.children << ref
  GroupableHelper.set_path_with_source_tree(ref, path, source_tree)
  ref.set_last_known_file_type
  ref
endv
```



``` ruby
# Sets the path of the given object according to the provided source
# tree key. The path is converted to relative according to the real
# path of the source tree for group and project source trees, if both
# paths are relative or absolute. Otherwise the path is set as
# provided.
#
# @param  [PBXGroup, PBXFileReference] object
#         The object whose path needs to be set.
#
# @param  [#to_s] path
#         The path.
#
# @param  [Symbol, String] source_tree
#         The source tree, either a string or a key for
#         {SOURCE_TREES_BY_KEY}.
#
# @return [void]
#

def set_path_with_source_tree(object, path, source_tree)
  path = Pathname.new(path)
  source_tree = normalize_source_tree(source_tree)
  object.source_tree = source_tree

  if source_tree == SOURCE_TREES_BY_KEY[:absolute]
    unless path.absolute?
      raise '[Xcodeproj] Attempt to set a relative path with an ' \
        "absolute source tree: `#{path}`"
    end
    object.path = path.to_s
  elsif source_tree == SOURCE_TREES_BY_KEY[:group] || source_tree == SOURCE_TREES_BY_KEY[:project]
    source_tree_real_path = GroupableHelper.source_tree_real_path(object)
    if source_tree_real_path && source_tree_real_path.absolute? == path.absolute?
      relative_path = path.relative_path_from(source_tree_real_path)
      object.path = relative_path.to_s
    else
      object.path = path.to_s
    end
  else
    object.path = path.to_s
  end
end
```



``` ruby
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





#### AbstractObject

```
lib/xcodeproj/project/object.rb
```

这是Xcode项目中可以存在的所有对象类型的基类。因此，它提供了常见的行为，但是您只能使用AbstractObject子类的实例，因为这个类不存在于实际的Xcode项目中。



#### UUIDGenerator









- CLAide

  命令解析器

- Cocoapods-Core

  DSL解析器

- Cocoapods-Downloader

  下载模块

- Molinillo

  依赖仲裁算法

- nanaimo

  nanaimo是一个实现ASCII Plist序列化和反序列化的简单库，完全使用原生Ruby代码（并且没有依赖关系）。它还提供了对序列化Xcode projects（带注释）和XML plist的现成支持。

- Xcodeproj

  xcode文件生成

- Cocoapods-Plugins

  插件管理


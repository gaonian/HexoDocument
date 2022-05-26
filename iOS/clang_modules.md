# Introduction

大多数软件是使用许多库构建的，包括平台提供的库、作为软件本身提供结构的一部分构建的内部库，以及第三方库。对于每个库，都需要访问它的接口(API)和实现。在C家族语言中，通过包含适当的头文件来访问库的接口:

``` c
#include <SomeLib.h>
```

通过链接适当的库来单独处理实现。例如，通过将 `-lSomeLib` 传递给链接器

Modules 提供了另一种更简单的方法来使用软件库，它提供了更好的 compile-time scalability (编译时可伸缩性)，并消除了使用C预处理器访问库的API所固有的许多问题

## Problems with the current model

C 预处理器提供的 `#include` 机制是访问库API的一种非常糟糕的方式，原因有很多:

- **Compile-time scalability** (编译时可伸缩性)：每次包含一个头文件时，编译器必须预处理并解析该头文件及其包含的每个头文件中的文本。应用中的每个转换单元都必须重复这个过程，这涉及到大量的冗余工作。在一个包含 N 个转换单元和每个转换单元中包含 M 个头文件的项目中，即使大多数 M 头文件在多个转换单元中共享，编译器仍在执行`M * N` 的工作。c++尤其糟糕，因为模板的编译模型迫使大量代码进入头文件

- **Fragility** （脆弱性）：`#include` 指令被预处理程序视为文本包含，因此在包含时受到任何活动宏定义的约束。如果任何活动宏定义与库中的名称发生冲突，它可能会破坏库API或导致库头文件本身的编译失败。对于一个极端的例子，`#define std "The c++ Standard"`，然后包含一个标准库头文件:结果是c++标准库实现中出现了一系列可怕的失败。更微妙的现实问题发生在两个不同库的头文件由于宏冲突而相互作用时，用户被迫重新排序`#include` 指令或引入 `#undef`指令以中断(无意的)依赖。

- **Conventional workarounds**（传统的解决方案）：C程序员已经采用了许多约定来解决C预处理器模型的脆弱性。例如，绝大多数头文件都需要包含保护，以确保多个包含不会破坏编译。宏名使用`LONG_PREFIXED_UPPERCASE_IDENTIFIERS` 来避免冲突，一些库/框架开发人员甚至在头文件中使用`__underscored`的名称来避免与(按照惯例)不应该是宏的“正常”名称发生冲突。这些约定是来自非c语言的开发人员进入的障碍，是更有经验的开发人员的样板，并且使我们的headers比它们应该的更难看。
- **Tool confusion**（工具混乱）：在基于c的语言中，很难构建与软件库很好地工作的工具，因为库的边界不清楚。哪些头文件属于某个特定的库，这些头文件应该以什么顺序包含以保证正确编译?  headers 是C, C++， Objective-C++，还是这些语言的变体之一? 那些头文件中的哪些声明实际上是API的一部分，哪些声明只是因为必须作为头文件的一部分而出现?

## Semantic import 

Modules 通过用更健壮、更高效的语义模型替换文本预处理器包含模型，改善了对软件库API的访问。从用户的角度来看，代码看起来只有轻微的不同，因为我们使用了`import`声明而不是`#include`预处理器指令:

```c
import std.io; // pseudo-code; see below for syntax discussion
```

然而，这个 module import 的行为与相应的`#include <stdio.h>`非常不同: 当编译器看到上面的模块导入时，它会加载`std.io`模块的二进制表示，并让应用程序可以直接使用它的API。导入声明之前的预处理器定义对`std.io`提供的API没有影响，因为模块本身是作为一个独立的模块编译的。此外，当模块被导入时，使用`std.io`模块所需的任何链接器标志都会自动提供

- **Compile-time scalability** (编译时可伸缩性)：`std.io`模块只编译一次，并且将模块导入到转换单元是一个常量时间操作(独立于模块系统)。这样，每个软件库的API只被解析一次，将 `M * N` 的编译问题简化为 `M + N` 问题
- **Fragility** （脆弱性）：每个模块都是作为一个独立的实体进行解析的，因此它有一个一致的预处理器环境。这完全消除了对 `__underscored` 的名称和类似的防御技巧的需要。此外，当遇到导入声明时，当前的预处理器定义将被忽略，因此一个软件库不会影响另一个软件库的编译方式，从而消除了包含顺序依赖关系。

- **Tool confusion**（工具混乱）：模块描述了软件库的API，工具可以推理并将模块作为API的表示来呈现。因为模块只能独立构建，所以工具可以依赖模块定义来确保它们获得完整的库API。此外，模块可以指定它们使用的语言，例如，不能意外地尝试将c++模块加载到C程序中。

## Problems modules do not solve

许多编程语言都有一个模块或包系统，由于这些语言提供了各种各样的特性，所以定义模块不能做什么很重要。特别地，以下所有的模块都被认为是超出范围的:

- **Rewrite the world’s code**：要求应用程序或软件库进行剧烈或非向后兼容的更改是不现实的，完全消除头文件也是不可行的。模块必须与现有的软件库互操作，并允许逐步过渡。
- **Versioning**：模块没有版本信息的概念。程序员仍然必须依赖底层语言的现有版本控制机制(如果存在的话)来对软件库进行版本控制
- **Namespaces**：与某些语言不同，模块不包含任何命名空间的概念。因此，在一个模块中声明的结构仍然会与在不同模块中声明的同名结构发生冲突，就像在两个不同的头文件中声明的结构一样。这方面对于向后兼容性很重要，因为(例如)在引入模块时，不能更改软件库中实体的混乱名称
- **Binary distribution of modules**：头文件(尤其是c++头文件)暴露了该语言的全部复杂性。跨架构、编译器版本和编译器供应商维护稳定的二进制模块格式在技术上是不可行的。

# Using Modules

要开启modules功能，请传递命令行标志 `-fmodules`。这将使任何支持模块的软件库都可以作为模块使用，并引入任何特定于模块的语法。其他命令行参数将在后面的单独部分中进行描述。

## Objective-C Import declaration

Objective-C 提供通过@import声明导入模块的语法，该声明将导入已命名的模块:

```objective-c
@import std;
```

上面的 `@import` 声明导入了 `std` 模块的全部内容(它将包含，例如，整个C或c++标准库)，并使其API在当前的转换单元中可用。为了只导入模块的一部分，可以对特定的子模块使用点语法，例如:

```objective-c
@import std.io;
```

冗余的导入声明被忽略，并且只要导入声明在全局范围内，就可以在转换单元内的任何地点导入模块。

目前，导入声明没有C或c++语法。Clang将在c++委员会中跟踪模块提案。请参阅 `include as imports`一节，了解当前模块是如何导入的。

## Includes as imports

模块的主要用户级特性是导入操作，它提供了对软件库API的访问。然而，今天的程序广泛使用 `#include`，假设所有这些代码一夜之间就会改变是不现实的。相反，模块会自动将 `#include`指令转换为相应的模块导入。例如，include指令

```c
#include <stdio.h>
```

将自动映射到模块 `std.io` 的导入。即使在语言中有特定的导入语法，这个特殊的特性对于采用和向后兼容性都很重要: `#include` to` import`的自动转换允许应用程序获得模块的好处(对于所有支持模块的库)，而不需要对应用程序本身进行任何更改。因此，用户可以很容易地在使用一个编译器时使用模块，而在使用其他编译器时则退回到包含预处理程序的机制

在构建模块时，也支持 `#include_next`，但有一点需要注意。`#include_next` 的通常行为是在包含路径列表中搜索指定的文件名，从找到当前文件的路径之后的路径开始。因为在模块映射中列出的文件不能通过include路径找到，所以在这样的文件中，对 `#include_next`指令使用了不同的策略: 在包含路径列表中搜索指定的头文件名，以找到指向当前文件的第一个包含路径。`#include_next` 将被解释为当前文件已经在该路径中找到。如果这个搜索找到一个由模块映射命名的文件，`#include_next`指令会被翻译成导入，就像`#include`指令一样。

## Module maps

modules 和 headers 之间的关键链接由 module map 描述，它描述了现有 headers maps  如何映射到 module 的(逻辑)结构上。例如，可以想象一个模块 `std` 覆盖了C标准库。每个C标准库头文件(<stdio.h>， <stdlib.h>， <math.h>，等等)都将对std模块做出贡献，将它们各自的api放入相应的子模块(std.io, std.lib, std.math，等等)。拥有一个 `std` 模块的 headers 列表，允许编译器将 `std` 模块构建为一个独立的实体，并且拥有从头名称到(子)模块的映射，允许将 `#include` 指令自动转换为模块导入。

module maps 被指定为单独的文件(每个名为 `module.modulemap` )以及它们所描述的头文件，这允许它们被添加到现有的软件库中，而无需更改库头文件本身(在大多数情况下是[2])。后面的部分将描述实际的  [Module map language](https://clang.llvm.org/docs/Modules.html#module-map-language)

可以使用 module maps 而不使用 mudules 来检查头文件使用的完整性。为此，可以使用 `-fimplicit-module-maps` 选项来代替 `-fmodules` 选项，或者使用 `-fmodule-map-file=` 选项来显式指定要加载的模块映射文件

## Compilation model

模块的二进制表示是由编译器根据需要自动生成的。当一个模块被导入时(例如，通过一个模块头文件的`#include`)，编译器将生成自己的第二个实例[3]，并带有一个新的预处理上下文[4]，以解析该模块中的头文件。生成的抽象语法树(AST)随后被持久化到模块的二进制表示中，然后将该模块加载到遇到模块导入的转换单元中。

模块的二进制表示被持久化在模块缓存中。导入模块将首先查询模块缓存，如果所需模块的二进制表示已经可用，则将直接加载该表示。因此，每个语言配置只解析模块头一次，而不是每个使用模块的转换单元解析一次。

模块维护对模块构建中每个头文件的引用。如果这些头文件中的任何一个发生了变化，或者某个模块所依赖的任何模块发生了变化，那么该模块将(自动)被重新编译。这个过程不应该需要任何用户干预

## Command-line parameters

- `-fmodules`

  Enable the modules feature

- `-fbuiltin-module-map`

  加载Clang builtins模块映射文件。(等价于 `-fmodule-map-file=<resource dir>/include/module.modulemap` )

- `-fimplicit-module-maps`

  对名为 `module.modulemap`和类似的模块映射文件启用隐式搜索。这个选项是由 `-fmodules` 隐含的。如果使用 `-fno-implicit-module-maps` 禁用此功能，则模块映射文件只有在通过 `-fmodule-map-file`显式指定或由另一个模块映射文件传递使用时才会加载

- `-fmodules-cache-path=<directory>`

  指定模块缓存的路径。如果没有提供，Clang将选择一个适合系统的默认值。

- `-fno-autolink`

  禁用与导入模块关联的库的自动链接

- `-fmodules-ignore-macro=macroname`

  指示模块在选择适当的模块变体时忽略指定的宏。对于在命令行上定义的不影响模块构建方式的宏，可以使用此方法来改进编译模块文件的共享

- `-fmodules-prune-interval=seconds`

  指定尝试修剪模块缓存之间的最小延迟(以秒为单位)。模块缓存修剪试图清除旧的、未使用的模块文件，以便模块缓存本身不会无限制地增长。默认延迟很大(604,800秒，或7天)，因为这是一个昂贵的操作。将此值设置为0以关闭修剪。

- `-fmodules-prune-after=seconds`

  指定模块缓存中的文件在模块修剪将其删除之前必须未使用的最小时间(以秒为单位)。默认延迟很大(2,678,400秒，或31天)，以避免过度的模块重建。

- `-module-file-info <module file name>`

  输出有关给定模块文件(扩展名为.pcm)的信息的调试辅助工具，包括构建特定模块变体时使用的语言和预处理器选项。

- `-fmodules-decluse`

  启用模块 `use` 声明的检查

- `-fmodule-name=module-id`

  将源文件视为给定模块的一部分

- `-fmodule-map-file=<file>`

  如果加载了给定模块映射文件的目录或其子目录中的一个头文件，则加载该模块映射文件

- `-fmodules-search-all`

  如果没有找到符号，搜索当前模块映射中引用的模块，但没有为符号导入，因此错误消息可以通过名称引用模块。注意，如果之前没有构建全局模块索引，这可能会花费一些时间，因为它需要构建所有模块。注意，这个选项在模块构建中不适用，以避免递归。

- `-fno-implicit-modules`

  构建中使用的所有模块都必须通过 `-fmodule-file` 指定

- `-fmodule-file=[<name>=]<file>`

  指定模块名称到预编译模块文件的映射。如果省略名称，则加载模块文件，无论实际是否需要。如果指定了名称，那么映射将被视为另一种预构建模块搜索机制(除了`-fprebuilt-module-path`)，并且只在需要时加载模块。注意，在这种情况下，指定的文件还覆盖了可能嵌入到其他预编译模块文件中的模块路径。

- `-fprebuilt-module-path=<directory>`

  指定预构建模块的路径。如果指定了，我们将在这个目录中查找给定的顶级模块名的模块。在这个目录中，我们不需要模块映射来加载预构建的模块，编译器也不会尝试重新构建这些模块。可以多次指定。

- `-fprebuilt-implicit-modules`

  启用预构建的隐式模块。如果在预构建模块路径(通过 `-fprebuilt-module-path`指定)中没有找到预构建模块，我们将在预构建模块路径中寻找匹配的隐式模块。

### -cc1 Options

- `-fmodules-strict-context-hash`

  支持对隐式构建中可能影响模块语义的所有编译器选项进行散列。这包括标题搜索路径和诊断。如果命令行参数在整个构建中不相同，使用此选项可能会导致构建的模块数量过多。

## Using Prebuilt Modules

下面是一些通过不同选项演示预构建模块用法的示例

首先，让我们为示例设置文件

``` c
/* A.h */
#ifdef ENABLE_A
void a() {}
#endif
```

```c
/* B.h */
#include "A.h"
```

```c
/* use.c */
#include "B.h"
void use() {
#ifdef ENABLE_A
  a();
#endif
}
```

```c
/* module.modulemap */
module A {
  header "A.h"
}
module B {
  header "B.h"
  export *
}
```

在下面的示例中，可以在不使用 `-cc1` 的情况下编译 `use.c`，但是用于预构建模块的命令需要更新，以考虑传递给 `clang -cc1` 的默认选项。还要注意，由于我们使用了 `-cc1`，所以我们显式地指定了`-fmodule-map-file=`或 `-fimplit -module-maps` 选项。当使用clang驱动程序时，`-fimplit -module-maps` 是由 `-fmodules` 隐含的。

首先，让我们使用从模块到文件的显式映射

```shell
rm -rf prebuilt ; mkdir prebuilt
clang -cc1 -emit-module -o prebuilt/A.pcm -fmodules module.modulemap -fmodule-name=A
clang -cc1 -emit-module -o prebuilt/B.pcm -fmodules module.modulemap -fmodule-name=B -fmodule-file=A=prebuilt/A.pcm
clang -cc1 -emit-obj use.c -fmodules -fmodule-map-file=module.modulemap -fmodule-file=A=prebuilt/A.pcm -fmodule-file=B=prebuilt/B.pcm
```

不需要手动指定映射，使用 `-fprebuilt-module-path` 选项会很方便。我们还可以使用 `-fimplici -module-maps`，而不是手动指向我们的模块映射。

```shell
rm -rf prebuilt; mkdir prebuilt
clang -cc1 -emit-module -o prebuilt/A.pcm -fmodules module.modulemap -fmodule-name=A
clang -cc1 -emit-module -o prebuilt/B.pcm -fmodules module.modulemap -fmodule-name=B -fprebuilt-module-path=prebuilt
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt
```

在一个命令中预构建源文件所需的所有模块的一个技巧是在使用 `-fdisable-module-hash` 选项时生成隐式模块

```shell
rm -rf prebuilt ; mkdir prebuilt
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fmodules-cache-path=prebuilt -fdisable-module-hash
ls prebuilt/*.pcm
# prebuilt/A.pcm  prebuilt/B.pcm
```

注意，对于显式或预构建的模块，我们要负责，并且应该特别注意模块的兼容性。使用不匹配的编译选项和模块可能会导致问题。

```shell
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -DENABLE_A
# use.c:4:10: warning: implicit declaration of function 'a' is invalid in C99 [-Wimplicit-function-declaration]
#   return a(x);
#          ^
# 1 warning generated.
```

因此，我们需要维护预构建模块的多个版本。我们可以使用手动模块映射，或者指向不同的预构建模块缓存路径。例如:

```shell
rm -rf prebuilt ; mkdir prebuilt ; rm -rf prebuilt_a ; mkdir prebuilt_a
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fmodules-cache-path=prebuilt -fdisable-module-hash
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fmodules-cache-path=prebuilt_a -fdisable-module-hash -DENABLE_A
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt
clang -cc1 -emit-obj use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt_a -DENABLE_A
```

与手动管理不同的模块版本不同，我们可以在给定的缓存路径中构建隐式模块(使用 `-fmodules-cache-path` )，并通过传递 `-fprebuilt-module-path` 和 `-fprebuilt-implicit-modules` 将它们重用为预构建的隐式模块。

```shell
rm -rf prebuilt; mkdir prebuilt
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fmodules-cache-path=prebuilt
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fmodules-cache-path=prebuilt -DENABLE_A
find prebuilt -name "*.pcm"
# prebuilt/1AYBIGPM8R2GA/A-3L1K4LUA6O31.pcm
# prebuilt/1AYBIGPM8R2GA/B-3L1K4LUA6O31.pcm
# prebuilt/VH0YZMF1OIRK/A-3L1K4LUA6O31.pcm
# prebuilt/VH0YZMF1OIRK/B-3L1K4LUA6O31.pcm
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -fprebuilt-implicit-modules
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -fprebuilt-implicit-modules -DENABLE_A
```

最后，我们希望允许隐式模块用于未预构建的配置。当使用clang驱动程序时，会隐式选择模块缓存路径。使用 `-cc1` ，我们只需添加 `-fmodules-cache-path` 选项。

```shell
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -fprebuilt-implicit-modules -fmodules-cache-path=cache
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -fprebuilt-implicit-modules -fmodules-cache-path=cache -DENABLE_A
clang -cc1 -emit-obj -o use.o use.c -fmodules -fimplicit-module-maps -fprebuilt-module-path=prebuilt -fprebuilt-implicit-modules -fmodules-cache-path=cache -DENABLE_A -DOTHER_OPTIONS
```

这样，一个包含多个模块变体的目录就可以准备和重用了。配置模块缓存的选项独立于其他选项



# Module Semantics

modules 被建模为每个子模块都是一个独立的转换单元，模块导入使其他转换单元的名称可见。每个子模块都以一个新的预处理器状态和一个空的转换单元开始

> 这种行为目前仅在构建带有子模块的模块时近似。已经构建的子模块中的实体在该模块中构建以后的子模块时是可见的。这可能导致脆弱的模块，这些模块依赖于模块的子模块所使用的构建顺序，不应该被依赖。这种行为可能会改变。

例如，在C语言中，这意味着如果两个结构体定义在具有相同名称的不同子模块中，这两个类型是不同的类型(但如果它们的定义匹配，则可能是兼容类型)。在c++中，在不同的子模块中定义的具有相同名称的两个结构体是相同的类型，并且在c++的一个定义规则下必须是等效的。

> Clang目前只对违反“一个定义规则”的行为进行最低限度的检查。

如果模块的任何子模块被导入到程序的任何部分，则整个顶级模块被认为是程序的一部分。因此，Clang可以诊断未导入子模块中声明的实体与当前转换单元中声明的实体之间的冲突，并且Clang可以基于未导入子模块的知识进行内联或去虚拟化。

## Macros

C 和 C++ 预处理器假设输入文本是一个单一的线性缓冲区，但对于模块，情况并非如此。可以导入两个对宏有冲突定义的模块 (或者一个`#define` 一个宏，另一个`#undef`)。在模块存在时处理宏定义的规则如下:

- 宏的每个定义和未定义都被认为是一个不同的实体
- 如果这些实体来自当前子模块或转换单元，或者它们是从已导入的子模块中导出的，则它们是可见的
- `#define x` 或 `#undef x` 指令会覆盖指令所在地点可见的所有 `x` 定义
- 如果一个 `#define` 或 `#undef` 指令是可见的，并且没有可见指令覆盖它，那么它就是活动的
- 如果一组宏指令仅由 `#undef` 指令组成，或者如果集合中的所有 `#define` 指令将宏名定义为相同的标记序列(遵循宏重定义的通常规则)，则该宏指令集是一致的。
- 如果使用了宏名，并且活动指令集不一致，则程序是病态的。否则，将使用宏名称的(唯一)含义。

例如, 假设:

- `<stdio.h>` 定义一个宏 `getc` (并导出它的 `#define` )
- `<cstdio>` 导入 `<stdio.h>` 模块并取消定义宏(并导出它的 `#undef` )

`#undef` 将覆盖 `#define` ，并且以任意顺序导入两个模块的源文件将不会看到 `getc` 被定义为宏



# Module Map Language

> 模块映射语言目前不能保证在Clang的主要版本之间保持稳定

模块映射语言描述了从头文件到模块逻辑结构的映射。为了支持将库作为模块使用，必须为该库编写一个 `module.modulemap` 文件。`module.modulemap` 文件被放置在头文件旁边，并且是用下面描述的模块映射语言编写的。

> 为了与以前版本的兼容性，如果一个模块映射文件名为`module.modulemap`没有找到, Clang也会搜索名为 `module.map` 的文件。这种行为已被弃用，我们计划最终删除它。

例如，C标准库的模块映射文件可能看起来像这样:

```
module std [system] [extern_c] {
  module assert {
    textual header "assert.h"
    header "bits/assert-decls.h"
    export *
  }

  module complex {
    header "complex.h"
    export *
  }

  module ctype {
    header "ctype.h"
    export *
  }

  module errno {
    header "errno.h"
    header "sys/errno.h"
    export *
  }

  module fenv {
    header "fenv.h"
    export *
  }

  // ...more headers follow...
}
```

在这里，顶级模块 `std` 包含了整个C标准库。它有许多子模块，包含标准库的不同部分: `complex` 用于复数，`ctype` 用于字符类型等等。每个子模块列出了为该子模块提供内容的多个头文件中的一个。最后，`export *` 命令指定该子模块包含的所有内容将被自动重新导出

## Lexical structure（词汇结构）

模块映射文件使用简化形式的 C99 词法分析器，对 identifiers、token、string字面值、`/* */` 和 `//` 注释使用相同的规则。模块映射语言有以下保留字; 所有其他C标识符都是有效的 identifiers

```
config_macros export_as  private
conflict      framework  requires
exclude       header     textual
explicit      link       umbrella
extern        module     use
export
```

## Module map file

模块映射文件由一系列模块声明组成:

```
module-map-file:
  module-declaration*
```

在模块映射文件中，模块通过模块id来引用，模块id使用句点分隔模块名的每个部分:

```
module-id:
  identifier ('.' identifier)*
```

## Module declaration

模块声明描述一个模块，包括贡献给该模块的头文件、它的子模块以及该模块的其他方面。

```
module-declaration:
  explicitopt frameworkopt module module-id attributesopt '{' module-member* '}'
  extern module module-id string-literal
```

`module-id` 应该只由一个标识符组成，该标识符提供被定义模块的名称。每个模块应该有一个单独的定义。

`explicit` 限定符只能应用于子模块，即嵌套在另一个模块中的模块。只有当子模块本身在导入声明中显式命名或从导入的模块中重新导出时，显式子模块的内容才可用。

`framework` 限定符指定该模块对应于一个 Darwin-style 的框架。一个 Darwin-style 的框架(主要用于macOS和iOS)完全包含在目录 `Name.framework`中，其中Name是框架的名称(因此，也是模块的名称)。该目录的布局如下:

```
Name.framework/
  Modules/module.modulemap  Module map for the framework
  Headers/                  Subdirectory containing framework headers
  PrivateHeaders/           Subdirectory containing framework private headers
  Frameworks/               Subdirectory containing embedded frameworks
  Resources/                Subdirectory containing additional resources
  Name                      Symbolic link to the shared library for the framework
```

`system` 属性指定该模块是系统模块。当重新构建系统模块时，该模块的所有头都将被视为系统头，这将抑制警告。这相当于在每个模块的头文件中放置 `#pragma GCC system_header`。属性的形式在下面的属性一节中描述。

`extern_c` 属性指定模块包含可以在c++内部使用的C代码。当构建这样一个模块用于c++代码时，该模块的所有头文件都将被视为包含在隐式 `extern "C"` 块中。具有此属性的模块的导入可以出现在`extern "C"` 块中。但是，没有取消其他限制:模块目前不能在命名空间中的 `extern "C"` 块中导入

`no_undeclared_includes` 属性指定模块只能访问非模块头和来自已使用模块的头。因为一些headers可以出现在多个搜索路径和映射到不同的模块在每个路径,这种机制帮助叮当声找到合适的header, 例如,更喜欢一个对当前模块或子模块,而不是第一个匹配的搜索路径

模块可以有许多不同类型的成员，每个成员的描述如下:

```
module-member:
  requires-declaration
  header-declaration
  umbrella-dir-declaration
  submodule-declaration
  export-declaration
  export-as-declaration
  use-declaration
  link-declaration
  config-macros-declaration
  conflict-declaration
```

extern模块引用由 `string-literal` 给出的文件中的 `module-id` 定义的模块。该文件可以通过绝对路径或相对于当前映射文件的路径来引用

### Requires declaration

*requires-declaration* 指定导入转换单元使用模块时必须满足的需求

```
requires-declaration:
  requires feature-list

feature-list:
  feature (',' feature)*

feature:
  !opt identifier
```

requirements子句允许特定的模块或子模块指定它们只能通过特定的语言、平台、环境和特定的目标特性访问。特性列表是一组标识符，定义如下。如果在给定的转换单元中没有任何功能，则该转换单元不得导入该模块。当构建模块供编译使用时，需要不可用特性的子模块将被忽略。可选!表示某个特性与该模块不兼容

定义了以下特性:

- altivec

  ​	target 支持 AltiVec

- blocks

  ​	"blocks" 语言特性是可用的

- coroutines

  ​	支持协同程序 TS 是可用的

- cplusplus

- cplusplus11

- cplusplus14

- cplusplus17

  ​	支持 C++、C++11、C++14、C++17 

- c99

- c11

- c17

  ​	支持 c99、c11，c17

- freestanding

  ​	有独立的环境

- gnuinlineasm

  ​	支持 gnu 内联 asm（汇编）

- objc

- objc_arc

  ​	支持 Objective-C 及 arc

- opencl

  ​	支持 OpenCL

- tls

  ​	支持线程本地存储

- *target feature*

  ​	一个特定的目标功能(例如，sse4, avx, neon)是可用的

- *platform/os*

  ​	一个操作系统/平台变体(例如freebsd, win32, windows, linux, ios, macos, iossimulator)是可用的

- *environment*

  ​	一个环境变量(例如gnu, gnueabi, android, msvc)是可用的

示例:  `std` 模块可以通过 `requires-declaration` 扩展为包含 c++ 和 c++ 11 头文件:

```
module std {
   // C standard library...

   module vector {
     requires cplusplus
     header "vector"
   }

   module type_traits {
     requires cplusplus11
     header "type_traits"
   }
 }
```

### Header declaration

header declaration 指定特定的 header 与封装模块相关联

```
header-declaration:
  privateopt textualopt header string-literal header-attrsopt
  umbrella header string-literal header-attrsopt
  exclude header string-literal header-attrsopt

header-attrs:
  '{' header-attr* '}'

header-attr:
  size integer-literal
  mtime integer-literal
```

不包含 `exclude` 或 `textical` 的 `header declaration`  指定了有助于封闭模块的header。具体来说，当模块被构建时，命名的头文件将被解析，它的声明将(逻辑上)被放置到外围的子模块中。

带有 `umbrella` 说明符的header称为 `umbrella header`。`umbrella header`包含其目录(和任何子目录)中的所有header，通常用于(在`#include`世界中)轻松访问特定库提供的完整API。使用模块时，`umbrella header` 是一种方便的快捷方式，无需为每个库头文件写出 header 声明。给定的目录只能包含单个`umbrella header`

> `umbrella`头中未包含的任何header都应具有显式`header declarations`。使用 `-Wincomplete-umbrella` 警告选项，要求Clang投诉`umbrella header`或 `module map` 未涵盖的头文件

带有`private`说明符的头文件不能从模块本身外部包含

当模块构建时，带有`textual`说明符的头文件将不会被编译，如果它由 `#include` 指令命名，它将被文本包含。但是，为了检查使用声明，它被认为是模块的一部分，并且仍然必须是词法上有效的头文件。将来，我们打算对这些头进行标记预处理，并在预构建的模块表示中包含标记序列

带有 `exclude` 说明符的头文件将被从模块中排除。当模块被构建时，它将不会被包含，也不会被认为是模块的一部分，即使 `umbrella header` 或目录会使它成为模块的一部分

示例: C头文件 `assert.h` 是 `textual header`的绝佳候选，因为它意味着要被包含多次(可能使用不同的 `NDEBUG` 设置)。但是，其中的声明通常应拆分为单独的 modular header

```
module std [system] {
  textual header "assert.h"
}
```

给定的 header 不得被多个 `header-declaration` 引用

两个 `header-declarations` 或者 一个 `header-declaration` 和 `#include`, 被认为指的是同一个文件如果路径解决相同的文件, 指定 `header-attrs` (如果有的话) 匹配的属性文件,即使文件被命名为不同(例如,通过一个相对路径或通过符号链接)。

> header-attrs的使用避免了Clang投机统计模块映射引用的每个头信息的需要。建议header-attrs只在机器生成的模块映射中使用，以避免属性值和相应文件之间的不匹配。

### Umbrella directory declaration

umbrella directory 声明指定目录中的所有 headers 都应该包含在模块中

```
umbrella-dir-declaration:
  umbrella string-literal
```

`string-literal` 指向一个目录。当构建模块时，该目录(及其子目录)中的所有头文件都包含在模块中

`umbrella-dir-declaration`  不能引用与 `umbrella header-declaration` 相同的目录。换句话说，对于给定的目录，只能指定一种类型的umbrella

> Umbrella directories  对于具有大量 header 但没有  umbrella header 的库非常有用

### Submodule declaration

`Submodule declarations ` 描述嵌套在其外围模块中的模块

```
submodule-declaration:
  module-declaration
  inferred-submodule-declaration
```

作为 `module-declaration` 的 `submodule-declaration` 声明是一个嵌套的模块。如果 `module-declaration` 有一个 `framework` 说明符，那么外围的模块也应该有一个 `framework` 说明符，子模块的内容应包含在子目录 `Frameworks/SubName.framework` 中，其中 `SubName` 为子模块的名称。

`submodule-declaration` (推论子模块声明)描述了一组子模块，这些子模块对应于属于模块的任何报头，但没有由报头声明显式描述。

作为 `inferred-submodule-declaration` 的 `submodule-declaration` 描述了一组 submodules，这些submodules对应于作为模块一部分但未由`header-declaration`显式描述的任何header。

```
inferred-submodule-declaration:
  explicitopt frameworkopt module '*' attributesopt '{' inferred-submodule-member* '}'

inferred-submodule-member:
  export '*'
```

包含` inferred-submodule-declaration` 的模块应具有 umbrella header 或 umbrella directory。`inferred-submodule-declaration` 所适用的头正是 umbrella header（可传递地）包含的头，或者包含在模块中的头，因为它们位于umbrella directory（或其子目录）中

对于`umbrella header`包含的或 `umbrella directory` 中没有通过 `header-declaration` 命名的每个头，将从` inferred-submodule-declaration `隐式生成模块声明。该模块将:

- 与头文件具有相同的名称(没有文件扩展名)
- 是否有 `explicit` 的说明符，如果 `inferred-submodule-declaration` 有 `explicit` 的说明符
- 是否有 `framework` 说明符，如果 `inferred-submodule-declaration` 有 `framework` 说明符
- `inferred-submodule-declaration`指定了属性吗
- 包含单个 `header-declaration` 的头文件声明
- 如果`inferred-submodule-declaration`包含`inferred-submodule-member`  `export *`，则包含单个导出声明 `export *`

示例: 如果子目录" MyLib "包含头文件 `A.h` 和 `B.h`，则如下模块映射:

```
module MyLib {
  umbrella "MyLib"
  explicit module * {
    export *
  }
}
```

等价于 (更详细的) 模块映射:

```
module MyLib {
  explicit module A {
    header "A.h"
    export *
  }

  explicit module B {
    header "B.h"
    export *
  }
}
```

### Export declaration

`export-declaration` 指定哪些导入的模块将作为给定模块API的一部分自动重新导出

```
export-declaration:
  export wildcard-module-id

wildcard-module-id:
  identifier
  '*'
  identifier '.' wildcard-module-id
```

`export-declaration` 命名了一个或一组模块，这些模块将被重新导出到导入外围模块的任何转换单元。每个与 `wildcard-module-id` 匹配但不包括第一个 `*` 的导入模块将被重新导出。

在以下示例中，导入 `MyLib.Derived` 还提供了`MyLib.Base` 的API：

```
module MyLib {
  module Base {
    header "Base.h"
  }

  module Derived {
    header "Derived.h"
    export Base
  }
}
```

请注意，如果 `Derived.h` 包括 `Base.h`，则只需使用通配符导出即可重新导出`Derived.h`的所有内容

```
module MyLib {
  module Base {
    header "Base.h"
  }

  module Derived {
    header "Derived.h"
    export *
  }
}
```

> 通配符导出语法 `export *` 重新导出实际头文件中导入的所有模块。因为`#include`指令会自动映射到模块导入，所以 `export *` 提供了与C预处理器相同的传递-包含行为，例如，导入一个给定的模块会隐式导入它所依赖的所有模块。因此，`export *`的自由使用为依赖传递包含的程序(即所有程序)提供了出色的向后兼容性。

### Re-export Declaration

`export-as-declaration`指定当前模块将由指定模块重新导出其接口

```
export-as-declaration:
  export_as identifier
```

`export-as-declaration` 命名当前模块将通过其重新导出。只有顶级模块可以被重新导出，任何给定的模块只能通过单个模块被重新导出。

在下面的示例中，`MyFrameworkCore` 模块将通过 `MyFramework` 模块重新导出:

```
module MyFrameworkCore {
  export_as MyFramework
}
```

### Use declaration

`use-declaration` 指定当前顶级模块打算使用的另一个模块。当指定了选项 `-fmodules-decluse` 时，模块只能使用以这种方式显式指定的其他模块

```
use-declaration:
  use module-id
```

在下面的示例中，未声明从C中使用A，因此将触发警告。

```
module A {
  header "a.h"
}

module B {
  header "b.h"
}

module C {
  header "c.h"
  use B
}
```

当编译实现模块的源文件时，使用选项 `-fmodule-name=module-id`表示源文件在逻辑上是该模块的一部分

编译器目前只对直接构建的模块施加限制

### Link declaration

`link-declaration` 指定了一个 library 或 framework，如果在该程序的任何转换单元中导入了外围模块，则该程序应根据该库或框架进行链接

```
link-declaration:
  link frameworkopt string-literal
```

`string-literal` 指定了程序应该链接到的库或框架的名称。例如，指定“clangBasic”将指示链接器使用 `-lclangBasic` 链接unix风格的链接器

框架中的链接声明指定了链接器应该链接到指定的框架，例如，使用 `-framework MyFramework`

> 使用 `link` 指令的自动链接还没有被广泛实现，因为它需要object文件格式和链接器的支持。这个概念类似于Microsoft Visual Studio的#pragma comment(lib…)。

### Conflict declarations

`conflict-declaration` 描述了在同一转换单元中存在两个不同模块可能导致问题的情况。例如，两个模块可能提供相似但不兼容的功能

```
conflict-declaration:
  conflict module-id ',' string-literal
```

`conflict-declaration`  的 `module-id` 指定与封装模块冲突的模块。在导入外围模块时，指定的模块不应已导入转换单元

当两个模块发生冲突时，`string-literal`提供一条消息，作为编译器诊断的一部分

> 当发现模块冲突时，Clang会发出警告(在 `-Wmodule-conflict` 的控制下)。

```
module Conflicts {
  explicit module A {
    header "conflict_a.h"
    conflict B, "we just don't like B"
  }

  module B {
    header "conflict_b.h"
  }
}
```



## Attributes

语法中的许多地方都使用属性来描述其他声明的特定行为。属性的格式相当简单。

```
attributes:
  attribute attributesopt

attribute:
  '[' identifier ']'
```

任何标识符都可以用作属性，每个声明都指定可以应用到它的属性



## Private Module Map Files

模块映射文件通常被命名为`module.modulemap`并放置在它们所描述的头文件旁边，或者放在它们所描述的头文件的父目录中。这些模块映射通常描述库的所有API

然而，在某些情况下，特定头文件的存在或不存在被用来区分特定库的“公共”和“私有”api。例如，库可能包含头文件 `Foo.h` 和 `Foo_Private.h`，分别提供公共和私有api。另外，`Foo_Private.h` 可能只在某些版本的库中可用，而在其他版本中则不存在。我们不能简单地用库中的单个模块映射文件来表达这一点:

```
module Foo {
  header "Foo.h"
  ...
}

module Foo_Private {
  header "Foo_Private.h"
  ...
}
```

因为头文件 `Foo_Private.h` 并不总是可用。模块映射文件可以根据 `Foo_Private.h` 是否可用来定制，但这样做需要自定义构建机制

私有模块映射文件，命名为`module.private.modulemap`(或者，为了向后兼容，`module_private.map` )，允许用一个额外的模块来扩充主模块映射文件。例如，我们将上面的模块映射文件分割为两个模块映射文件

```
/* module.modulemap */
module Foo {
  header "Foo.h"
}

/* module.private.modulemap */
module Foo_Private {
  header "Foo_Private.h"
}
```

当在`module.modulemap`文件旁边发现 `module.private.modulemap` 文件时，它在`module.modulemap`文件后加载。在我们的示例库中，当 `Foo_Private.h` 可用时，`module.private.modulemap`文件将可用，这使得沿着头边界分割库的公共api和私有api更加容易

当编写私有模块作为框架的一部分时，建议:

- 此模块的头文件存在于 `PrivateHeaders` 框架子目录中
- 私有模块被定义为带有公共框架名称前缀的顶级模块，如上面的 `Foo_Private`。Clang有额外的逻辑来处理这个命名，使用 `FooPrivate` 或 `Foo.Private`(子模块)触发警告，可能无法按预期工作。



# Modularizing a Platform

要从模块中获得任何好处，需要从堆栈的底部开始为软件库引入模块映射。这通常意味着引入一个模块映射，覆盖操作系统的头文件和C标准库头文件(对于Unix系统，在/usr/include中)。

模块映射将使用 [module map language](https://clang.llvm.org/docs/Modules.html#module-map-language) 编写，该语言提供了描述头文件和模块之间映射的必要工具。因为header集成在不同的系统之间是不同的，所以模块映射可能需要为特定的发行版和操作系统版本进行一些定制。此外，如果系统头本身显示出破坏模块的任何反模式，则可能需要进行一些修改。下面将描述这些常见模式。

- **Macro-guarded copy-and-pasted definitions**

  ​	系统头文件为用户提供核心类型，如 `size_t`。在许多系统头文件中经常需要这些类型，编写这些类型几乎是微不足道的。因此，在头文件中经常看到如下的复制粘贴定义:

  ```
  #ifndef _SIZE_T
  #define _SIZE_T
  typedef __SIZE_TYPE__ size_t;
  #endif
  ```

  不幸的是，当模块将所有的C库头文件编译成单个模块时，只有 `size_t` 的第一个实际类型定义是可见的，而且只在与幸运的第一个头文件对应的子模块中可见。具有该模式的复制粘贴版本的任何其他头文件都不会有 `size_t` 的定义。因此，导入与这些头文件对应的子模块将不会产生 `size_t`作为API的一部分，因为解析头文件时它并不存在。解决这个问题的方法是要么将复制的声明放到一个公共头文件中，在 `size_t` 是API的一部分的地方都包含该头文件，要么消除 `#ifndef` 并重新定义 `size_t` 类型。后者适用于c++头文件和C11，但会导致非模块C90/C99的错误，其中不允许重新定义 `typedef`

- **Conflicting definitions**

  ​	不同的系统头文件可能会为不同的宏、函数或类型提供冲突的定义。在预模块世界中，这些相互冲突的定义往往不会造成问题，除非有人碰巧在一个翻译单元中包含了两个头文件。由于修复通常只是简单地“不要那样做”，这样的问题仍然存在。模块要求消除相互冲突的定义，或者将它们放在单独的模块中(前者通常是更好的答案)。

- **Missing includes**

  ​	头文件通常缺少它们实际依赖的头文件的 `#include` 指令。与定义冲突的问题一样，这只会影响那些碰巧没有按正确顺序包含头文件的倒霉用户。对于模块，特定模块的头将被单独解析，因此如果缺少包含，模块可能无法构建

- **Headers that vend multiple APIs at different times**

  ​	有些系统的头文件包含许多不同类型的API定义，其中只有一部分是在给定的include中可用的。例如，只有在包含该头文件之前定义了宏`__need_size_t`时，头文件才可能 `vend size_t`，而且只有在定义了宏__need_wchar_t时，头文件才可能vend wchar_t。这样的头通常在单个翻译单元中包含很多次，并且没有包含保护。没有合理的方法将这个头文件映射到子模块。一个可以消除头(例如，通过分割成单独的头，每个实际的API)或简单地排除它在模块映射。

为了检测并帮助解决其中的一些问题，`clang-tools-extra` 存储库包含了一个模块化的工具，该工具解析一组给定的头文件并试图检测这些问题并生成报告。有关如何检查系统或库标头的信息，请参阅该工具的源代码文档。



# Future Directions

模块支持正在积极开发中，还有许多机会可以改进它。以下是一些建议:

- **Detect unused module imports**

  ​	与 `#include` 指令不同，跟踪一个直接导入的模块是否被使用过应该是相当简单的。通过这样做，Clang可以发出 `unused import` 或 `unused #include` 诊断，包括Fix-Its删除无用的imports/includes 

- **Fix-Its for missing imports**

  ​	在编写代码时使用某些API，却只会得到 “unknown type” 或者 “no function named” 的编译器错误，因为相应的头文件没有包含在内，这是相当常见的。Clang可以检测这种情况并自动导入所需的模块，但应该提供一个Fix-It来添加导入

- **Improve modularize**

  ​	模块化工具既非常重要(对于部署)，又非常粗糙。它需要更好的UI，更好的问题检测(特别是对于c++)，也许还需要一个辅助模式来帮助您编写模块映射



# Where To Learn More About Modules

Clang源代码提供了关于模块的额外信息:

- `clang/lib/Headers/module.modulemap`

  ​	Clang的编译器特定头文件的模块映射

- `clang/test/Modules/`

  ​	专门与模块功能相关的测试

- `clang/include/clang/Basic/Module.h`

  ​	这个头文件中的Module类描述了一个模块，并在整个编译器中使用它来实现模块

- `clang/include/clang/Lex/ModuleMap.h`

  ​	这个头文件中的 `ModuleMap` 类描述了完整的模块映射，包括所有已解析的模块映射文件，并提供了查找模块映射以及模块与头文件之间映射(双向)的工具



[clang module](https://clang.llvm.org/docs/Modules.html)


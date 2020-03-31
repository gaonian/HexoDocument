## 1. 摘要

以下两个对象和八个方法组成了SQLite接口的基本元素：

- `sqlite3` → database connection 对象。由 `sqlite3_open()` 创建，由`sqlite3_close()` 销毁。
- `sqlite3_stmt` → prepared statement 对象。由 `sqlite3_prepare()` 创建，由 `sqlite3_finalize()` 销毁。
- `sqlite3_open()` → 打开到新的或现有SQLite数据库的连接。`sqlite3` 的构造函数。
- `sqlite3_prepare()` → 将SQL文本编译成字节码，用于查询或更新数据库。`sqlite3` 的构造函数。
- `sqlite3_bind()` → 将应用程序数据存储到原始SQL的参数中。
- `sqlite3_step()` → 将 `sqlite3 stmt` 前进到下一个结果行或完成。
- `sqlite3_column()` → `sqlite3_stmt` 当前结果行中的列值。
- `sqlite3_finalize()` → `sqlite3_stmt` 的析构函数。
- `sqlite3_close()` → `sqlite3` 的析构函数。
- `sqlite3_exec()` → 为一个或多个SQL语句字符串执行`sqlite3_prepare()` 、`sqlite3_step()` 、`sqlite3_column()` 和`sqlite3_finalize()` 的包装函数。



## 2. 介绍

SQLite有超过225个api。但是，大多数api都是可选的，而且非常专业，初学者可以忽略它们。核心API小，简单，易学。本文总结了核心API。

一个单独的文档，SQLite C/C++接口，为SQLite提供了所有C/C++  API的详细规范。一旦读者理解了SQLite的基本操作原则，就应该使用该文档作为参考指南。本文仅作为介绍，既不是SQLite API的完整参考，也不是权威参考。



## 3. 核心对象和接口

SQL数据库引擎的主要任务是计算SQL语句。为此，开发人员需要两个对象：

**database connection 数据库连接对象：sqlite3**

**prepared statement 准备好的语句对象：sqlite3_stmt**

严格地说，prepared statement对象不是必需的，因为可以使用方便包装器接口`sqlite3_exec` 或 `sqlite3_get_table`，这些方便包装器封装并隐藏准备语句对象。然而，要充分利用SQLite，需要理解prepared statement。

**database connection** 和 **prepared statement对象** 由下面列出的一组C/C++接口控制。

- sqlite3_open()

- sqlite3_prepare()

- sqlite3_setup()

- sqlite3_column()

- sqlite3_finalize()

- sqlite3_close()

注意，上面的例程列表是概念性的，而不是实际的。许多这些例程都有多种版本。例如，上面的列表显示了一个名为 sqlite3_open() 的例程，而实际上有三个独立的例程以稍微不同的方式完成相同的任务：sqlite3_open()、sqlite3_open16() 和sqlite3_open v2() 。列表中提到 sqlite3_column() ，但实际上不存在这样的例程。列表中显示的  sqlite3_column() 是整个例程家族的占位符，这些例程在各种数据类型中都有额外的列数据。



以下是核心接口的功能摘要：

- **sqlite3_open()**

  此例程打开到SQLite数据库文件的连接并返回数据库连接对象。这通常是应用程序发出的第一个SQLite API调用，也是大多数其他SQLite API的先决条件。许多SQLite接口需要一个指向数据库连接对象的指针作为它们的第一个参数，并且可以将其视为数据库连接对象上的方法。此例程是数据库连接对象的构造函数。

- **sqlite3_prepare()**

  此例程将SQL文本转换为prepared statement对象，并返回指向该对象的指针。此接口需要通过先前调用 sqlite3_open() 创建的数据库连接指针和包含要准备的SQL语句的文本字符串。这个API实际上并不计算SQL语句。它只准备用于计算的SQL语句。

  把每个SQL语句看作一个小的计算机程序。sqlite3_prepare() 的目的是将该程序编译成目标代码。prepared statement 是目标代码。然后，sqlite3_step() 接口运行对象代码以获得结果。

  新应用程序应始终调用 sqlite3_prepare_v2() ，而不是sqlite3_prepare() 。为了向后兼容，保留了旧的 sqlite3_prepare() 。但是sqlite3_prepare_v2() 提供了更好的接口。

- **sqlite3_setup()**

  用于计算先前由 `sqlite3_prepare()` 接口创建的prepared statement。语句的计算一直到第一行结果可用为止。若要前进到结果的第二行，请再次调用sqlite3_step() 。继续调用 sqlite3_step() ，直到语句完成。不返回结果的语句（例如：INSERT、UPDATE或DELETE语句）在调用 sqlite3_step() 时运行到完成。

- **sqlite3_column()**

  此例程从由 sqlite3_step() 计算的prepared statement 的结果集的当前行返回一列。每次 sqlite3_step() 使用新的结果集行停止时，都可以多次调用此例程来查找该行中所有列的值。

  如上所述，SQLite API中实际上没有 sqlite3_column() 函数。相反，我们在这里称之为 sqlite3_column() 的是整个函数族的占位符，这些函数从各种数据类型的结果集中返回一个值。这个系列中还有一些例程返回结果的大小（如果是字符串或BLOB）和结果集中的列数。

  - sqlite3_column_blob()
  - sqlite3_column_bytes()
  - sqlite3_column_bytes16()
  - sqlite3_column_count()
  - sqlite3_column_double()
  - sqlite3_column_int()
  - sqlite3_column_int64()
  - sqlite3_column_text()
  - sqlite3_column_text16()
  - sqlite3_column_type()
  - sqlite3_column_value()

- **sqlite3_finalize()**

  此例程销毁由先前调用 `sqlite3_prepare()` 创建的prepared statement。为了避免内存泄漏，必须使用对这个例程的调用来销毁每个prepared statement。

- **sqlite3_close()**

  此例程关闭先前通过调用 `sqlite3_open()` 打开的数据库连接。与连接相关的所有prepared statement 应在关闭连接之前完成。

  

## 4. 核心例程和对象的典型用法

应用程序通常在初始化期间使用 `sqlite3_open()` 创建单个数据库连接。请注意，sqlite3_open() 可用于打开或创建现有数据库文件 和 打开新的数据库文件。虽然许多应用程序只使用一个数据库连接，但没有理由不能为了打开多个数据库连接（同一个数据库或不同的数据库）而多次调用 `sqlite3_open()`。有时多线程应用程序会为每个线程创建单独的数据库连接。请注意，单个数据库连接可以使用ATTACH SQL命令访问两个或多个数据库，因此不必为每个数据库文件单独建立数据库连接。



许多应用程序在关闭时使用对 `sqlite3_close()` 的调用来破坏它们的数据库连接。或者，例如，使用SQLite作为其应用程序文件格式的应用程序可能会打开数据库连接以响应文件/打开菜单操作，然后销毁相应的数据库连接以响应文件/关闭菜单。



要运行SQL语句，应用程序遵循以下步骤：

- 使用 `sqlite3_prepare()` 创建 prepared statement。

- 通过调用 `sqlite3_step()` 一次或多次来计算 prepared statement。

- 对于查询，通过在两次调用 `sqlite3_step()` 之间调用 `sqlite3_column()` 来提取结果。

- 使用 `sqlite3_finalize()` 销毁 prepared statement。

为了有效地使用SQLite，我们需要知道的就是前面的内容。剩下的就是优化和细节。



## 5. 核心例程的方便包装器

`sqlite3_exec()` 接口是一个方便的包装器，它通过一个函数调用来执行上述四个步骤。传递给 `sqlite3_exec()` 的回调函数用于处理结果集的每一行。

`sqlite3_get_table()` 是执行上述四个步骤的另一个方便包装器。`sqlite3_get_table()` 接口不同于 `sqlite3_exec()` ，它将查询结果存储在堆内存中，而不是调用回调。

重要的是要认识到，无论是 `sqlite3_exec()` 还是 `sqlite3_get_table()` 都不会做任何使用核心例程无法完成的事情。事实上，这些包装器完全是根据核心例程实现的。



## 6. 绑定参数和重用 Prepared Statements

在前面的讨论中，假设每个SQL语句都准备一次、求值一次，然后销毁。但是，SQLite允许对同一个 prepared statement 进行多次求值。这是使用以下例程完成的：

- **sqlite3_reset()**

- **sqlite3_bind()**

在一次或多次对 `sqlite3_step()` 的调用对 prepared statement 求值之后，可以通过调用 `sqlite3_reset()` 重置它，以便可以再次求值。

把 `sqlite3_reset()` 看作是将 prepared statement 程序倒回开头。对现有的prepared statement 使用 `sqlite3_reset()` 而不是创建新的 prepared statement 可以避免对 `sqlite3_prepare()` 的不必要调用。对于许多SQL语句，运行 `sqlite3_prepare()` 所需的时间等于或超过 `sqlite3_step()` 所需的时间。**因此，避免调用 `sqlite3_prepare()` 可以显著提高性能**。



多次计算完全相同的SQL语句通常并不有用。更常见的是，人们想要评估类似的陈述。例如，您可能需要使用不同的值多次计算INSERT语句。或者您可能希望在WHERE子句中使用不同的键多次计算同一查询。为了适应这种情况，SQLite允许SQL语句包含在计算之前“绑定”到值的参数。以后可以更改这些值，并使用新值再次计算同一 prepared statement。



SQLite允许在任何允许字符串文本、数字常量或NULL的地方使用参数。（参数不能用于列名或表名。）参数采用以下形式之一：

- **?**
- **?NNN**
- **:AAA**
- **$AAA**
- **@AAA**

在上面的例子中，NNN是一个整数值，AAA是一个标识符。参数最初的值为空。在第一次调用 `sqlite3_step()` 之前或在 `sqlite3_reset()` 之后，应用程序可以调用 `sqlite3_bind()` 接口将值附加到参数。对 `sqlite3_bind()` 的每次调用都会重写同一参数上以前的绑定。



允许应用程序预先准备多个SQL语句，并根据需要对它们进行求值。未完成的准备语句的数量没有任意限制。有些应用程序在启动时多次调用`sqlite3_prepare()` ，以创建它们需要的所有 prepared statement。其他应用程序保存最近使用的 prepared statement 的缓存，然后在可用时重用缓存中的 prepared statement。另一种方法是只在 prepared statement 位于循环内时重用它们。



## 7. 配置SQLite

SQLite的默认配置适用于大多数应用程序。但有时开发人员希望调整设置，以尽量挤出更多的性能，或利用一些模糊的功能。

`sqlite3_config()` 接口用于对SQLite进行全局、进程范围的配置更改。必须在创建任何数据库连接之前调用 `sqlite3_config()` 接口。`sqlite3_config()` 接口允许程序员执行以下操作：

- 调整SQLite进行内存分配的方式，包括设置适用于安全关键的实时嵌入式系统和应用程序定义的内存分配程序的备用内存分配程序。

- 设置进程范围的错误日志。

- 指定应用程序定义的页缓存。

- 调整互斥量的使用，使其适合于各种线程模型，或者替换应用程序定义的互斥量系统。

完成整个进程的配置并创建数据库连接后，可以使用对 `sqlite3_limit()` 和`sqlite3_db_config()` 的调用来配置单个数据库连接。



## 8. 扩展SQLite

SQLite包含可用于扩展其功能的接口。这些程序包括：

- **sqlite3_create_collation()**

  `sqlite3_create_collation()` 接口用于为文本排序创建新的排序序

- **sqlite3_create_function()**

  `sqlite3_create_module()` 接口用于注册新的虚拟表实现

- **sqlite3_create_module()**

  `sqlite3_create_function()` 接口创建新的SQL函数-标量函数或聚合函数。新功能实现通常使用以下附加接口：

  - [sqlite3_aggregate_context()](https://sqlite.org/c3ref/aggregate_context.html)
  - [sqlite3_result()](https://sqlite.org/c3ref/result_blob.html)
  - [sqlite3_user_data()](https://sqlite.org/c3ref/user_data.html)
  - [sqlite3_value()](https://sqlite.org/c3ref/value_blob.html)

- **sqlite3_vfs_register()**

  `sqlite3 vfs_register()` 接口创建新的vfs。

SQLite的所有内置SQL函数都是使用完全相同的接口创建的。请参阅SQLite源代码，特别是date.c和func.c源文件以获取示例。

共享库或DLLs可以用作SQLite的可加载扩展。



## 9. 其他接口

本文只提到最重要和最常用的SQLite接口。SQLite库包括许多其他api，这些api实现了这里没有描述的有用特性。在C/C++接口规范中找到了形成SQLite应用程序编程接口的[完整功能列表](https://sqlite.org/c3ref/funclist.html)。有关所有SQLite接口的完整和权威信息，[请参阅该文档](https://sqlite.org/c3ref/intro.html)。

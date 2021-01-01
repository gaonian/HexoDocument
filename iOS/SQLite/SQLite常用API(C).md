---
title: SQLite常用API（C）
categories: SQL
---



## 简介

SQLite接口元素可以分为三类：

- **对象列表**。这是SQLite库使用的所有抽象对象和数据类型的列表。总共有几十个对象，但最重要的两个对象是：**database connection object (sqlite3)**  和 **prepared statement object (sqlite3_stmt)**。

- **常数列表**。这是一个由SQLite使用的数字常量列表，由sqlite3.h头文件中的定义来表示。这些常量是各种接口（例如：SQLITE_OK）的数值结果代码或传递到函数中以控制行为的标志（例如：SQLITE_OPEN_READONLY）。

- **功能列表**。这是对对象进行操作并使用和/或返回常量的所有函数和方法的列表。有许多函数，但大多数应用程序只使用少数函数。



## Database Connection Object

``` c
typedef struct sqlite3 sqlite3;
```

每个打开的SQLite数据库都由一个指向名为 `sqlite3` 的不透明结构实例的指针表示。 将 `sqlite3` 指针视为对象很有用。

`sqlite3_open()` 、`sqlite3_open16()` 和 `sqlite3_open_v2()` 接口是其构造函数

`sqlite3_close()` 和 `sqlite3_close_v2()` 是其析构函数。

还有许多其他接口（如 `sqlite3_prepare_v2()` 、`sqlite3_create_function()` 和 `sqlite3_busy_timeout()` ）是sqlite3对象上的方法。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0de2eb8bd244a2cb09576e7897301a0~tplv-k3u1fbpfcp-watermark.image)



## Prepared Statement Object

``` c
typedef struct sqlite3_stmt sqlite3_stmt;
```

一个实例对象表示已编译为二进制形式并准备好进行计算的单个SQL语句。

把每个SQL语句看作一个单独的计算机程序。原始SQL文本是源代码。prepared语句对象是编译后的对象代码。所有SQL都必须转换为prepared语句才能运行。

prepared语句对象的生命周期通常如下：

1. 使用sqlite3_prepare_v2() 创建prepared语句对象。
2. 使用sqlite3_bind_*() 将值绑定到参数
3. 通过调用sqlite3_step() 一次或多次运行SQL
4. 使用sqlite3_reset() 重置prepared的语句，然后返回步骤2。这样做零次或多次
5. 使用sqlite3_finalize() 销毁对象



构造函数：

- sqlite3_prepare
- sqlite3_prepare_v2
- sqlite3_prepare_v3
- sqlite3_prepare16
- sqlite3_prepare16_v2
- sqlite3_prepare16_v3

析构函数:

- sqlite3_finalize()

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2677d9a95f9144b386cfdefa5d788030~tplv-k3u1fbpfcp-watermark.image)



## 常用api

### sqlite3_open

```c
int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
int sqlite3_open16(
  const void *filename,   /* Database filename (UTF-16) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
```

这些例程打开由 filename 参数指定的SQLite数据库文件。对于 `sqlite3_open()` 和`sqlite3_open_v2()`，filename为UTF-8；对于 `sqlite3_open16()`，filename为UTF-16。数据库连接句柄通常在 `*ppDb` 中返回，即使发生错误。唯一的例外是，如果SQLite无法分配内存来保存sqlite3对象，则会在 `*ppDb` 中写入一个NULL，而不是指向sqlite3对象的指针。如果数据库成功打开（and/or创建），则返回 `SQLITE_OK`。否则返回错误代码。`sqlite3_errmsg()` 或 `sqlite3_errmsg16()` 例程可用于获取任何`sqlite3_open()` 例程失败后错误的英文描述。

对于使用 `sqlite3_open()` 或 `sqlite3_open_v2()` 创建的数据库，默认编码为UTF-8。使用 `sqlite3_open16()` 创建的数据库的默认编码将按本机字节顺序为UTF-16。

**无论打开数据库连接句柄时是否发生错误，当不再需要时，都应通过将其传递给`sqlite3_close()` 来释放与数据库连接句柄相关联的资源**。

`sqlite3_open_v2()` 接口的工作方式与 `sqlite3_open()` 类似，只是它接受两个额外的参数来控制新的数据库连接。`sqlite3_open_v2()` 的flags参数必须至少包含以下三种标志组合之一：

- SQLITE_OPEN_READONLY

  数据库以只读模式打开。如果数据库不存在，则返回错误。

- SQLITE_OPEN_READWRITE

  如果可能，将打开数据库进行读写，或者仅当文件受操作系统写保护时才进行读操作。无论哪种情况，数据库必须已经存在，否则将返回错误。

- SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE

  打开数据库进行读写操作，如果数据库不存在，则创建数据库。这是始终用于sqlite3_open() 和 sqlite3_open16() 的行为。

除了必需的标志外，还支持以下可选标志：

- SQLITE_OPEN_URI

  如果设置了此标志，则文件名可以解释为URI。

- SQLITE_OPEN_MEMORY

  数据库将作为内存数据库打开。如果启用了共享缓存模式，则为实现缓存共享，数据库由“filename”参数命名，否则将忽略“filename”。

- SQLITE_OPEN_NOMUTEX

  新的数据库连接将使用“多线程”线程模式。这意味着允许单独的线程同时使用SQLite，只要每个线程使用不同的数据库连接。

- SQLITE_OPEN_FULLMUTEX

  新的数据库连接将使用“序列化”线程模式。这意味着多个线程可以安全地尝试同时使用同一个数据库连接。（互斥锁将阻止任何实际的并发，但在这种模式下，尝试没有任何危害。）

- SQLITE_OPEN_SHAREDCACHE

  数据库已打开并启用共享缓存，覆盖sqlite3_enable_shared_cache（）提供的默认共享缓存设置。

- SQLITE_OPEN_PRIVATECACHE

  数据库打开时共享缓存被禁用，覆盖sqlite3_enable_shared_cache（）提供的默认共享缓存设置。

- SQLITE_OPEN_NOFOLLOW

  数据库文件名不允许是符号链接



如果`sqlite3_open_v2()` 的第三个参数不是上面所示的必需组合之一（可选地与其他SQLITE_OPEN_* 位组合），则此行为是未定义的。

`sqlite3_open_v2()` 的第四个参数是定义新数据库连接应使用的操作系统接口的sqlite3_vfs对象的名称。如果第四个参数是空指针，则使用默认的sqlite3 vfs对象。

如果文件名为 **:memory:**，则会为连接创建一个专用的临时内存数据库。当数据库连接关闭时，这个内存中的数据库将消失。SQLite的未来版本可能会使用以 ":" 字符开头的其他特殊文件名。建议在数据库文件名实际以 ":" 字符开头时，应在文件名前面加上 "./" 等路径名，以避免歧义。

如果文件名为空字符串，则将创建一个专用的临时磁盘数据库。一旦数据库连接关闭，此专用数据库将被自动删除。



### sqlite3_close

``` c
int sqlite3_close(sqlite3*);
int sqlite3_close_v2(sqlite3*);
```

`sqlite3_close()` 和 `sqlite3_close_v2()` 是 sqlite3对象的析构函数。调用`sqlite3_close()` 和 `sqlite3_close_v2()` 将在成功销毁sqlite3对象并释放所有关联资源时返回 `SQLITE_OK`。

如果数据库连接与未完成的prepared语句或未完成的 sqlite3_backup 对象关联，则`sqlite3_close()` 将使数据库连接保持打开状态，并返回 `SQLITE_BUSY`。如果使用未完成的prepared语句或未完成的 sqlite3_backups 调用 `sqlite3_close v2()`，则数据库连接将成为一个不可用的“僵尸”，在最后一个prepared语句完成或最后一个 `sqlite3_backup` 完成时将自动释放。`sqlite3_close_v2()` 接口用于垃圾收集的宿主语言，其中调用析构函数的顺序是任意的。



应用程序应完成所有prepared语句，关闭所有BLOB句柄，并在尝试关闭该对象之前完成与 sqlite3对象关联的所有sqlite3_backup对象。如果对仍有未完成的prepared语句、BLOB句柄和/或sqlite3_backup对象的数据库连接调用 `sqlite3_close_v2()`，则它将返回 `SQLITE_OK`，并延迟资源的释放，直到所有已准备的语句、BLOB句柄和sqlite3_backup对象也被销毁。

如果在事务打开时销毁了sqlite3对象，则会自动回滚该事务。

`sqlite3_close（C）` 和 `sqlite3_close v2（C）` 的C参数必须是空指针，或者是从`sqlite3_open（）` 、`sqlite3_open16（）` 或 `sqlite3_open v2（）` 获得的sqlite3对象指针，并且之前没有关闭。使用空指针参数调用 `sqlite3_close（）` 或 `sqlite3_close_v2（）` 是无害的no-op。



### sqlite3_prepare_v2

``` c
int sqlite3_prepare_v2(
  sqlite3 *db,            /* Database handle */
  const char *zSql,       /* SQL statement, UTF-8 encoded */
  int nByte,              /* Maximum length of zSql in bytes. */
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle */
  const char **pzTail     /* OUT: Pointer to unused portion of zSql */
);
```

要执行SQL语句，必须首先使用sqlite3_prepare_v2将其编译为字节码程序。或者，换句话说，这些例子是prepared语句对象的构造函数。

- 第一个参数 `db` 是从先前成功调用sqlite3_open（）、sqlite3_open v2（）或sqlite3_open16（）获得的数据库连接。数据库连接必须尚未关闭。

- 第二个参数 `zSql` 是要编译的语句，编码为UTF-8或UTF-16。sqlite3_prepare（）、sqlite3_prepare_v2（）和sqlite3_prepare_v3（）接口使用UTF-8，sqlite3_prepare16（）、sqlite3_prepare16_v2（）和sqlite3_prepare16_v3（）使用UTF-16。

- 如果`nByte`参数为负，则`zSql`被读取到第一个零终止符。如果`nByte`为正，则它是从`zSql`读取的字节数。如果`nByte`为零，则不生成任何准备好的语句。如果调用者知道提供的字符串是以nul-terminated，那么传递一个nByte参数（包括nul-terminated在内的输入字符串中的字节数）有一个小的性能优势。

- 如果 `pzTail` 不为空，那么`*pzTail`将指向`zSql`中第一个SQL语句结束后的第一个字节。这些例程只编译zSql中的第一个语句，因此`*pzTail`指向未编译的内容。

- `*ppStmt`左指可以使用 sqlite3_step() 执行的已编译准备语句。如果有错误，`*ppStmt`设置为空。如果输入文本不包含SQL（如果输入是空字符串或注释），则`*ppStmt`设置为空。调用过程负责在完成编译后使用 sqlite3_finalize() 删除已编译的SQL语句。ppStmt不能为NULL。

如果成功，`sqlite3_prepare_v2` 将返回`SQLITE_OK`；否则将返回错误代码。



### sqlite3_bind_parameter_count

```c
int sqlite3_bind_parameter_count(sqlite3_stmt*);
```

可用于查找parepared语句中的SQL参数的数量。SQL参数是 `"?"`, `"?NNN"`, `":AAA"` , `"$AAA"`, or `"@AAA"` 形式的标记,  用作以后绑定到参数的值的占位符。



### sqlite3_bind_parameter_index

``` c
int sqlite3_bind_parameter_index(sqlite3_stmt*, const char *zName);
```

返回给定名称的SQL参数的索引。返回的索引值适合用作sqlite3_bind（）的第二个参数。如果找不到匹配的参数，则返回零。即使原始语句是使用sqlite3_prepare16_v2（）或sqlite3_prepare16_v3（）从UTF-16文本准备的，也必须在UTF-8中指定参数名。



### sqlite3_bind_parameter_name

```c
const char *sqlite3_bind_parameter_name(sqlite3_stmt*, int);
```

sqlite3_bind_parameter_name（P，N）接口返回prepared语句P中的第N个SQL参数的名称。

SQL的表单参数 `"?NNN"` 或`:AAA`或 `@AAA` 或 `$AAA` 的名称分别对应的是字符串 `?NNN` 或 `:AAA` 或 `@AAA` 或 `$AAA`。 换句话说，首字母 `：`或 `$` 或 `@` 或 `?` 包含在名称中。表格参数 `？`以下整数没有名称，称为“无名”或“匿名参数”。

第一个参数的索引为1，而不是0。返回的字符串始终采用UTF-8编码。



### sqlite3_bind_xxx

```c
int sqlite3_bind_blob(sqlite3_stmt*, int, const void*, int n, void(*)(void*));
int sqlite3_bind_blob64(sqlite3_stmt*, int, const void*, sqlite3_uint64,
                        void(*)(void*));
int sqlite3_bind_double(sqlite3_stmt*, int, double);
int sqlite3_bind_int(sqlite3_stmt*, int, int);
int sqlite3_bind_int64(sqlite3_stmt*, int, sqlite3_int64);
int sqlite3_bind_null(sqlite3_stmt*, int);
int sqlite3_bind_text(sqlite3_stmt*,int,const char*,int,void(*)(void*));
int sqlite3_bind_text16(sqlite3_stmt*, int, const void*, int, void(*)(void*));
int sqlite3_bind_text64(sqlite3_stmt*, int, const char*, sqlite3_uint64,
                         void(*)(void*), unsigned char encoding);
int sqlite3_bind_value(sqlite3_stmt*, int, const sqlite3_value*);
int sqlite3_bind_pointer(sqlite3_stmt*, int, void*, const char*,void(*)(void*));
int sqlite3_bind_zeroblob(sqlite3_stmt*, int, int n);
int sqlite3_bind_zeroblob64(sqlite3_stmt*, int, sqlite3_uint64);
```

在输入到 `sqlite3_prepare_v2()` 及其变体的SQL语句文本中，文本可以替换为与以下模板之一匹配的参数：

- ?
- ?NNN
- :VVV
- @VVV
- $VVV

在上面的模板中，**NNN表示整数文本，VVV表示字母数字标识符**。这些参数的值（也称为“主机参数名”或“SQL参数”）可以使用此处定义的`sqlite3_bind_*() `进行设置。

- `sqlite3_bind_*() ` 的第一个参数始终是指向从 `sqlite3_prepare_v2()` 或其变体返回的 `sqlite3_stmt` 对象的指针。

- 第二个参数是要设置的SQL参数的索引。最左边的SQL参数的索引为1。当同一个命名的SQL参数被多次使用时，第二个和随后的出现的都与第一个出现的具有相同的索引。 如果需要，可以使用 `sqlite3_bind_parameter_index()` API查找命名参数的索引。索引 `?NNN` 参数是NNN的值。NNN值必须介于1和`sqlite3_limit()` 参数`SQLITE_LIMIT_VARIABLE_NUMBER`之间（默认值：999）。

- 第三个参数是要绑定到该参数的值。如果 `sqlite3_bind_text()` 或`sqlite3_bind_text16()` 或 `sqlite3_bind_blob()` 的第三个参数是空指针，则忽略第四个参数，最终结果与 `sqlite3_bind_NULL()` 相同。

- 在那些有第四个参数的例子中，它的值是参数中的字节数。要清楚：值是值中的字节数，而不是字符数。如果 `sqlite3_bind_text()` 或`sqlite3_bind_text16()` 的第四个参数为负，则字符串的长度是第一个零终止符之前的字节数。如果 `sqlite3_bind_blob()` 的第四个参数为负，则行为未定义。如果向 `sqlite3_bind_text()` 或 `sqlite3_bind_text16()` 或 `sqlite3_bind_text64()` 提供了非负的第四个参数，则该参数必须是假定字符串以NUL结尾时NUL结束符出现的字节偏移量。如果任何NUL字符的字节偏移量小于第四个参数的值，则生成的字符串值将包含嵌入的NUL。包含嵌入nul的字符串的表达式的结果是未定义的。

- BLOB和string绑定接口的第五个参数是一个析构函数，用于在SQLite处理完BLOB或string之后对其进行处理。调用析构函数是为了释放BLOB或字符串，即使对bind API的调用失败，但如果第三个参数为空指针或第四个参数为负，则不调用析构函数。如果第五个参数是特殊值 `SQLITE_STATIC`，那么SQLite假设信息位于静态的非托管空间中，不需要释放。如果第五个参数的值是 `SQLITE_TRANSIENT`，那么SQLite会在 `sqlite3_bind_*() ` 例子返回之前立即创建自己的数据私有副本。
- `sqlite3_bind_text64()` 的第六个参数必须是SQLITE_UTF8、SQLITE_UTF16、SQLITE_UTF16BE 或 SQLITE_UTF16LE 之一，才能指定第三个参数中文本的编码。如果sqlite3_bind_text64()的第六个参数不是上面显示的允许值之一，或者如果文本编码与第六个参数指定的编码不同，则行为未定义。

`sqlite3_bind_zeroblob()`例程绑定长度为N的BLOB，该BLOB由零填充。zeroblob在处理时使用固定数量的内存（只是一个整数来保存其大小）。ZeroBlob用作BLOB的占位符，其内容稍后将使用增量BLOB I/O例程编写。零BLOB的负值将导致零长度BLOB。

`sqlite3_bind_pointer（S，I，P，T，D）` 例程使准备好的语句S中的第I个参数的SQL值为空，但也与T类型的指针P相关联。D既可以是空指针，也可以是指向P的析构函数的指针。使用完P后，SQLite将用P的单个参数调用析构函数DT参数应该是静态字符串，最好是字符串文本。sqlite3_bind_pointer（）例程是为sqlite3.20.0添加的指针传递接口的一部分。

如果使用prepared语句的空指针或使用比sqlite3_reset（）最近为其调用了sqlite3_step（）的准备好的语句调用任何 `sqlite3_bind_*() `例程，则调用将返回 `SQLITE_MISUSE`。如果任何 `sqlite3_bind_*() `例程被传递一个已完成的prepared语句，则结果是未定义的，并且可能有危害。

`sqlite3_reset()` 不能清除绑定。未绑定参数解释为NULL。

`sqlite3_bind_*` 在成功时返回 `SQLITE_OK`，如果有任何错误，则返回错误代码。如果字符串或BLOB的大小超过了sqlite3_limit（`SQLITE_LIMIT_LENGTH`）或 `SQLITE_MAX_LENGTH` 设置的限制，则可能返回 `SQLITE_TOOBIG`。如果参数索引超出范围，则返回 `SQLITE_RANGE`。如果 `malloc()` 失败，则返回`SQLITE_NOMEM`。



### sqlite3_step

```c
int sqlite3_step(sqlite3_stmt*);
```

在调用 `sqlite3_prepare_v2()` 等接口生成prepared语句后，必须调用此函数一次或多次以计算语句。

在传统接口中，返回值将是`SQLITE_BUSY`、`SQLITE_DONE`、`SQLITE_ROW`、`SQLITE_ERROR` 或 `SQLITE_MISUSE`。使用“v2”接口，还可以返回任何其他结果代码或扩展结果代码。

- `SQLITE_BUSY` 表示数据库引擎无法获取执行其工作所需的数据库锁。如果该语句是提交语句或发生在显式事务之外，则可以重试该语句。如果该语句不是COMMIT，并且发生在显式事务中，则应在继续之前回滚该事务。

- `SQLITE_DONE` 表示语句已成功完成执行。如果不首先调用 `sqlite3_reset()` 将虚拟机重置回初始状态，则不应在此虚拟机上再次调用 `sqlite3_step()`

- 如果正在执行的SQL语句返回任何数据，则每次调用方准备好处理新的数据行时，都会返回 `SQLITE_ROW`。可以使用列访问函数访问这些值。再次调用`sqlite3_step()` 以检索下一行数据。

- `SQLITE_ERROR` 表示发生了运行时错误（例如约束冲突）。不应在虚拟机上再次调用 `sqlite3_step()`。可以通过调用 `sqlite3_errmsg()` 找到更多信息。对于传统接口，可以通过对准备好的语句调用 `sqlite3_reset()` 来获得更具体的错误代码（例如，SQLITE_INTERRUPT、SQLITE_SCHEMA、SQLITE_CORRUPT等等）。在“v2”接口中，更具体的错误代码由 `sqlite3_step()` 直接返回。

- `SQLITE_MISUSE` 意味着这个例程被不适当地调用。可能是在已完成的准备好的语句上调用的，也可能是在以前返回SQLITE_ERROR或SQLITE_DONE的语句上调用的。或者可能是同一数据库连接同时被两个或多个线程使用。



### sqlite3_reset

``` c
int sqlite3_reset(sqlite3_stmt *pStmt);
```

调用 `sqlite3_reset()` 函数将prepared语句对象重置回其初始状态，准备重新执行。任何使用 `sqlite3_bind_*()` API将值绑定到它们的SQL语句变量都将保留它们的值。使用 `sqlite3_clear_bindings()` 重置绑定。

`sqlite3_reset(S)`  接口将 prepared语句 S 重置回其程序的开头。

如果为prepared语句S最近一次调用 sqlite3_step 返回了 SQLITE_ROW 或SQLITE_DONE，或者如果 sqlite3_step 以前从未在S上调用过，则sqlite3_reset返回SQLITE_OK。

如果对prepared语句的sqlite3_step(S) 的最新调用指示错误，则sqlite3_reset将返回相应的错误代码。

sqlite3_reset（S）接口不会更改prepared语句S上任何绑定的值。



### sqlite3_column_count

```c
int sqlite3_column_count(sqlite3_stmt *pStmt);
```

返回prepared语句返回的结果集中的列数。如果此函数返回0，则表示prepared语句不返回数据（例如UPDATE）。然而，仅仅因为返回一个正数并不意味着将返回一行或多行数据。SELECT语句将始终具有正的 `sqlite3_column_count()` ，但根据WHERE子句约束和表内容，它可能不会返回任何行。



### sqlite3_data_count

``` c
int sqlite3_data_count(sqlite3_stmt *pStmt);
```

`sqlite3_data_count（P）` 接口返回prepared statement P结果集中当前行中的列数。如果prepared statement P没有准备好返回的结果（通过调用sqlite3_column（）接口系列），则sqlite3_data_count（P）返回0。如果P是空指针，sqlite3_data_count（P）例程也返回0。如果之前对sqlite3步骤（P）的调用返回SQLITE_DONE，则sqlite3_data_count（P）例程返回0。如果先前调用sqlite3_step（P）返回SQLITE_ROW，则sqlite3_data_count（P）将返回非零，除非PRAGMA incremental_vacuum始终返回零，因为该多步骤PRAGMA的每个步骤都返回0列数据。



### sqlite3_column_name

``` c
const char *sqlite3_column_name(sqlite3_stmt*, int N);
const void *sqlite3_column_name16(sqlite3_stmt*, int N);
```

这些例程返回在SELECT语句的结果集中指定给特定列的名称。`sqlite3_column_name()` 接口返回指向以零结尾的UTF-8字符串的指针，`sqlite3_column_name16()` 返回指向以零结尾的UTF-16字符串的指针。第一个参数是实现SELECT语句的prepared语句。第二个参数是列号。最左边的列是0。

返回的字符串指针有效，直到准备好的语句被 `sqlite3_finalize()` 销毁，或者直到第一次调用 `sqlite3_step()` 为特定运行自动重新准备语句，或者直到下一次调用同一列上的 `sqlite3_column_name()` 或 `sqlite3_column_name16()` 。

如果sqlite3_malloc（）在处理任一例程期间（例如在从UTF-8转换为UTF-16期间）失败，则返回空指针。

结果列的名称是该列的“AS”子句的值（如果有AS子句）。如果没有AS子句，则列的名称是未指定的，可能会改变从下一个release SQLite版本。



### sqlite3_column_xxx

```c
const void *sqlite3_column_blob(sqlite3_stmt*, int iCol);
double sqlite3_column_double(sqlite3_stmt*, int iCol);
int sqlite3_column_int(sqlite3_stmt*, int iCol);
sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol);
const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);
const void *sqlite3_column_text16(sqlite3_stmt*, int iCol);
sqlite3_value *sqlite3_column_value(sqlite3_stmt*, int iCol);
int sqlite3_column_bytes(sqlite3_stmt*, int iCol);
int sqlite3_column_bytes16(sqlite3_stmt*, int iCol);
int sqlite3_column_type(sqlite3_stmt*, int iCol);
```

```c
sqlite3_column_blob	→	BLOB result
sqlite3_column_double	→	REAL result
sqlite3_column_int	→	32-bit INTEGER result
sqlite3_column_int64	→	64-bit INTEGER result
sqlite3_column_text	→	UTF-8 TEXT result
sqlite3_column_text16	→	UTF-16 TEXT result
sqlite3_column_value	→	The result as an unprotected sqlite3_value object.
 	 	 
sqlite3_column_bytes	→	Size of a BLOB or a UTF-8 TEXT result in bytes
sqlite3_column_bytes16  	→  	Size of UTF-16 TEXT in bytes
sqlite3_column_type	→	Default datatype of the result
```

这些例程返回有关查询的当前结果行的单个列的信息。在任何情况下，第一个参数都是一个指针，指向正在计算的prepared语句（从sqlite3_prepare_v2（）或其变体返回的sqlite3_stmt*），第二个参数是应返回其信息的列的索引。结果集最左边的列具有索引0。结果中的列数可以使用 `sqlite3_column_count()` 确定。

如果SQL语句当前未指向有效行，或者列索引超出范围，则返回结果未定义。只有在最近对 `sqlite3_step()` 的调用返回了 `SQLITE_ROW` 并且随后既没有调用`sqlite3_reset()` 也没有调用 `sqlite3_finalize()` 时，才可以调用这些例程。如果在 `sqlite3_reset()` 或 `sqlite3_finalize()` 之后或 `sqlite3_step()` 返回`SQLITE_ROW` 以外的内容之后调用这些例程，则结果未定义。如果在挂起这些例程时从其他线程调用 `sqlite3_step()` 或 `sqlite3_reset()` 或 `sqlite3_finalize()` ，则结果未定义。

前六个接口（“blob”、“double”、“int”、“int64”、“text”和“text16”）都以特定的数据格式返回结果列的值。如果结果列最初不是请求的格式（例如，如果查询返回整数，但使用sqlite3_column_text（）接口提取值），则执行自动类型转换。

`sqlite3_column_type()` 例程返回结果列的初始数据类型的数据类型代码。返回的值是 `SQLITE_INTEGER`、`SQLITE_FLOAT`、`SQLITE_TEXT`、`SQLITE_BLOB` 或 `SQLITE_NULL` 之一。`sqlite3_column_type()` 的返回值可用于决定前六个接口中的哪一个应用于提取列值。`sqlite3_column_type()` 返回的值只有在没有对该值进行自动类型转换时才有意义。在类型转换之后，调用 `sqlite3_column_type()` 的结果是未定义的，尽管是无害的。SQLite的未来版本可能会在类型转换之后更改sqlite3_column_type（）的行为。



如果结果是BLOB或文本字符串，则可以使用 `sqlite3_column_bytes()` 或`sqlite3_column_bytes16()` 接口来确定该BLOB或字符串的大小。



如果结果是BLOB或UTF-8字符串，则 `sqlite3_column_bytes()` 返回该BLOB或字符串中的字节数。如果结果是UTF-16字符串，那么 `sqlite3_column_bytes()` 将字符串转换为UTF-8，然后返回字节数。如果结果是数值，则 `sqlite3_column_bytes()` 使用`sqlite3_snprintf()` 将该值转换为UTF-8字符串，并返回该字符串中的字节数。如果结果为空，则 `sqlite3_column_bytes()` 返回零。



`sqlite3_column_bytes()` 和 `sqlite3_column_bytes16()` 返回的值不包括字符串末尾的零终止符。为了清楚起见：sqlite3_column_bytes（）和sqlite3_column_bytes16（）返回的值是字符串中的字节数，而不是字符数。



`sqlite3_column_text()` 和 `sqlite3_column_text16()` 返回的字符串，即使是空字符串，也始终以零结尾。对于零长度blob，sqlite3_column_blob（）的返回值是空指针。



这些例程可能试图转换结果的数据类型。例如，如果内部表示为FLOAT，并且请求文本结果，则在内部使用 `sqlite3_snprintf()` 自动执行转换。下表详细说明了应用的转换：

> | Internal Type | Requested Type | Conversion                                                   |
> | ------------- | -------------- | ------------------------------------------------------------ |
> | NULL          | INTEGER        | Result is 0                                                  |
> | NULL          | FLOAT          | Result is 0.0                                                |
> | NULL          | TEXT           | Result is a NULL pointer                                     |
> | NULL          | BLOB           | Result is a NULL pointer                                     |
> | INTEGER       | FLOAT          | Convert from integer to float                                |
> | INTEGER       | TEXT           | ASCII rendering of the integer                               |
> | INTEGER       | BLOB           | Same as INTEGER->TEXT                                        |
> | FLOAT         | INTEGER        | [CAST](https://sqlite.org/lang_expr.html#castexpr) to INTEGER |
> | FLOAT         | TEXT           | ASCII rendering of the float                                 |
> | FLOAT         | BLOB           | [CAST](https://sqlite.org/lang_expr.html#castexpr) to BLOB   |
> | TEXT          | INTEGER        | [CAST](https://sqlite.org/lang_expr.html#castexpr) to INTEGER |
> | TEXT          | FLOAT          | [CAST](https://sqlite.org/lang_expr.html#castexpr) to REAL   |
> | TEXT          | BLOB           | No change                                                    |
> | BLOB          | INTEGER        | [CAST](https://sqlite.org/lang_expr.html#castexpr) to INTEGER |
> | BLOB          | FLOAT          | [CAST](https://sqlite.org/lang_expr.html#castexpr) to REAL   |
> | BLOB          | TEXT           | Add a zero terminator if needed                              |

注意，当发生类型转换时，以前调用 `sqlite3_column_blob()` 、`sqlite3_column_text()` 和/或 `sqlite3_column_text16()` 返回的指针可能会失效。在以下情况下，可能会发生类型转换和指针无效：

初始内容是BLOB，并调用 `sqlite3_column_text()` 或 `sqlite3_column_text16()`。可能需要将零结束符添加到字符串中。

初始内容是UTF-8文本，并调用 `sqlite3_column_bytes16()` 或`sqlite3_column_text16()` 。内容必须转换为UTF-16。

初始内容是UTF-16文本，并调用 `sqlite3_column_bytes()` 或 `sqlite3_column_text()` 。内容必须转换为UTF-8。

UTF-16be和UTF-16le之间的转换总是在适当的位置进行的，并且不会使先前的指针失效，当然，先前指针引用的缓冲区的内容将被修改。其他类型的转换是在可能的情况下进行的，但有时它们是不可能的，在这些情况下，先前的指针是无效的。



最安全的策略是以下列方式之一调用这些例程：

`sqlite3_column_text()` 后跟 `sqlite3_column_bytes()`

`sqlite3_column_blob()` 后跟 `sqlite3_column_bytes()`

`sqlite3_column_text16()` 后跟 `sqlite3_column_bytes16()`

换言之，您应该首先调用sqlite3_column_text（）、sqlite3_column_blob（）或sqlite3_column_text16（），将结果强制为所需格式，然后调用sqlite3_column_bytes（）或sqlite3_column_bytes16（）来查找结果的大小。不要将对sqlite3_column_text（）或sqlite3_column_blob（）的调用与对sqlite3_column_bytes16（）的调用混合，也不要将对sqlite3_column_text16（）的调用与对sqlite3_column_bytes（）的调用混合。



返回的指针在如上所述进行类型转换之前，或在调用sqlite3_step（）或sqlite3_reset（）或sqlite3_finalize（）之前有效。用于保存字符串和blob的内存空间将自动释放。不要将从sqlite3_column_blob（）、sqlite3_column_text（）等返回的指针传递到sqlite3_free（）中。



只要输入参数正确，只有在格式转换期间发生内存不足错误时，这些例程才会失败。只有以下接口子集会出现内存不足错误：

sqlite3_column_blob()
sqlite3_column_text()
sqlite3_column_text16()
sqlite3_column_bytes()
sqlite3_column_bytes16()

如果发生内存不足错误，则这些例程的返回值与列包含SQL空值时的返回值相同。通过在获取可疑返回值之后和在同一数据库连接上调用任何其他SQLite接口之前调用sqlite3_errcode（），可以将有效的SQL NULL返回与内存不足错误区分开来。



### sqlite3_finalize

```c
int sqlite3_finalize(sqlite3_stmt *pStmt);
```

调用 `sqlite3_finalize()` 函数以删除prepared语句。如果最近对语句的求值没有遇到错误，或者从未对语句求值，则 `sqlite3_finalize()` 返回 `SQLITE_OK`。如果对语句S的最新求值失败，则 `sqlite3_finalize` 返回相应的错误代码或扩展错误代码。

`sqlite3_finalize（S）` 可以在prepared语句S的生命周期中的任何时间调用：在对语句S求值之前、在对sqlite3_reset（）的一个或多个调用之后，或在对sqlite3_step（）的任何调用之后，无论语句是否已完成执行。

对空指针调用sqlite3_finalize（）是无害的no-op。

**应用程序必须完成每一个prepared语句，以避免资源泄漏**。在finalize一个prepared语句之后，对它的任何使用都可能导致未定义和不受欢迎的行为，如segfaults和堆损坏。



### sqlite3_exec

``` c
int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
```

`sqlite3_exec()` 接口是 `sqlite3_prepare_v2()` 、`sqlite3_step()` 和`sqlite3_finalize()` 的方便包装，它允许应用程序运行多个SQL语句，而无需使用大量C代码。

`sqlite3_exec()` 接口在作为第一个参数传入的数据库连接的上下文中，运行零个或多个传递到其第二个参数中的UTF-8编码、分号分隔的SQL语句。如果 `sqlite3_exec()` 的第三个参数的回调函数不为空，那么将为从计算的SQL语句中输出的每个结果行调用该函数。`sqlite3_exec()` 的第四个参数被转发到每个回调调用的第一个参数。如果指向 `sqlite3_exec()` 的回调指针为空，则永远不会调用任何回调并忽略结果行。

如果在计算传递给 `sqlite3_exec()` 的SQL语句时出错，则停止执行当前语句，并跳过后续语句。如果 `sqlite3_exec()` 的第5个参数不为空，则任何错误消息都将写入从`sqlite3_malloc()` 获取的内存中，并通过第5个参数传回。为了避免内存泄漏，应用程序应该在不再需要错误消息字符串之后，对通过 `sqlite3_exec()` 的第5个参数返回的错误消息字符串调用 `sqlite3_free()`。如果 `sqlite3_exec()` 的第5个参数不为空并且没有出现错误，那么 `sqlite3_exec()` 在返回之前将其第5个参数中的指针设置为空。

如果 `sqlite3_exec()` 回调返回非零，则 `sqlite3_exec()` 例程返回 `SQLITE_ABORT`，而不再次调用回调，也不运行任何后续的SQL语句。

`sqlite3_exec()` 回调函数的第二个参数是结果中的列数。`sqlite3_exec()` 回调函数的第三个参数是指向字段值的指针数组，这些字符串是从 `sqlite3_column_text()` 获得的，每列一个。如果结果行的元素为空，则 `sqlite3_exec()` 回调的对应字符串指针为空指针。`sqlite3_exec()` 回调的第四个参数是指向字段名称的指针数组，其中每个项表示从 `sqlite3_column_name()` 获得的相应结果列的名称。

如果 `sqlite3_exec()` 的第二个参数是空指针、指向空字符串的指针或只包含空白和/或SQL注释的指针，则不计算SQL语句，也不更改数据库。



限制：

应用程序必须确保 `sqlite3_exec()` 的第一个参数是有效且开放的数据库连接。

在运行 `sqlite3_exec()` 时，应用程序不能关闭由第一个参数指定的到`sqlite3_exec()` 的数据库连接。

在运行 `sqlite3_exec()` 时，应用程序不能修改传递到 `sqlite3_exec()` 的第二个参数中的SQL语句文本。



### sqlite3_busy_handler

``` c
int sqlite3_busy_handler(sqlite3*,int(*)(void*,int),void*);
```

`sqlite3_busy_handler（D，X，P）` 例程设置一个回调函数X，当另一个线程或进程锁定了一个表时，当试图访问与数据库连接D相关联的数据库表时，可以使用参数P调用该函数。sqlite3_busy_handler（）接口用于实现sqlite3_busy_timeout（）和PRAGMA busy_timeout。

如果busy回调为空，则遇到锁时立即返回SQLITE_BUSY。如果busy回调不为空，则可以使用两个参数调用回调。

busy处理程序的第一个参数是void*指针的副本，它是sqlite3_busy_handler（）的第三个参数。busy handler回调的第二个参数是以前为同一锁定事件调用busy handler的次数。如果busy回调返回0，则不会再尝试访问数据库，并将SQLITE_BUSY返回给应用程序。如果回调返回非零，则再次尝试访问数据库并重复周期。

繁忙处理程序的存在并不保证在发生锁争用时会调用它。如果SQLite确定调用busy处理程序可能导致死锁，它将继续并将SQLITE_BUSY返回给应用程序，而不是调用busy处理程序。考虑一个场景，其中一个进程持有一个读取锁，它正试图升级为保留锁，而另一个进程持有一个保留锁，它正试图升级为独占锁。第一个进程无法继续，因为它被第二个进程阻止；第二个进程无法继续，因为它被第一个进程阻止。如果两个进程都调用繁忙的处理程序，两个进程都不会取得任何进展。因此，SQLite返回第一个进程的SQLITE_BUSY，希望这将导致第一个进程释放其读锁并允许第二个进程继续。

默认回调为空。

只能为每个数据库连接定义一个忙处理程序。设置新的忙处理程序将清除以前设置的任何处理程序。请注意，调用sqlite3_busy_timeout（）或计算PRAGMA busy_timeout=N将更改busy处理程序，从而清除以前设置的任何busy处理程序。

busy回调不应执行任何修改调用busy处理程序的数据库连接的操作。换句话说，忙碌的处理程序是不可重入的。任何此类操作都会导致未定义的行为。

不能通过关闭数据库连接数或prepared语句关闭busy handler。



### sqlite3_wal_checkpoint_v2

```sql
int sqlite3_wal_checkpoint_v2(
  sqlite3 *db,                    /* Database handle */
  const char *zDb,                /* Name of attached database (or NULL) */
  int eMode,                      /* SQLITE_CHECKPOINT_* value */
  int *pnLog,                     /* OUT: Size of WAL log in frames */
  int *pnCkpt                     /* OUT: Total number of frames checkpointed */
);
```

`sqlite3_wal_checkpoint_v2（D，X，M，L，C）` 接口在模式M下对数据库连接D的数据库X运行检查点操作。状态信息被写回由L和C指向的整数中。M参数必须是有效的检查点模式：

- SQLITE_CHECKPOINT_PASSIVE

  在不等待任何数据库读写器完成的情况下检查尽可能多的帧，如果日志中的所有帧都被选中，则同步数据库文件。在SQLITE_CHECKPOINT_被动模式下从不调用繁忙处理程序回调。另一方面，如果存在并发读写器，被动模式可能会使检查点未完成。

- SQLITE_CHECKPOINT_FULL

  此模式将阻塞（它调用busy handler回调），直到没有数据库写入程序并且所有读卡器都在读取最新的数据库快照。然后检查日志文件中的所有帧并同步数据库文件。此模式在挂起时阻止新的数据库写入程序，但允许新的数据库读取程序继续不受阻碍。

- SQLITE_CHECKPOINT_RESTART

  此模式的工作方式与SQLITE_CHECKPOINT_FULL相同，另外，在检查日志文件之后，它会阻塞（调用busy handler回调），直到所有读卡器都只从数据库文件读取为止。这将确保下一个写入程序从头开始重新启动日志文件。与SQLITE_CHECKPOINT_FULL类似，此模式在挂起时阻止新的数据库编写器尝试，但不会妨碍读卡器。

- SQLITE_CHECKPOINT_TRUNCATE

  此模式的工作方式与SQLITE_CHECKPOINT_RESTART相同，它还将在成功返回之前将日志文件截断为零字节。



## 参考资料

[sqlite c/c++ api 官方文档](https://sqlite.org/c3ref/intro.html)
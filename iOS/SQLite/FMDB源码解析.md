---
title: FMDB源码解析
categories: SQL
---



## 概要

FMDB是iOS上针对sqlite3封装的数据库框架，简单使用，支持多线程安全操作，以OC的方式封装了sqlite3的C语言api。

![](https://user-gold-cdn.xitu.io/2020/5/11/1720425e508d88f9?w=2012&h=1404&f=png&s=257183)



FMDB代码并不算多，只有几个类。 下面分别介绍几个类的主要作用

- `FMDatabase` 

  表示单个SQLite数据库，用于执行sql语句。不要在多线程下使用单个的FMDatabase，多线程下应使用FMDatabaseQueue

- `FMResultSet`

   表示在FMDatabase上执行查询的结果

- `FMDatabaseQueue`

   如果要在多线程上执行查询和更新，则需要使用此类。

- `FMStatement`

  sqlite_stmt 的包装类
  
- `FMDatabasePool` 

   FMDatabase对象池，建议使用FMDatabaseQueue，不要使用此类



## FMDatabase

FMDatabase是主要的类，负责和数据库交互。 FMDatabaseQueue、FMDatabasePool内部都是调用的这个类处理sql。

主要从四个方面来讲解FMDatabase类

- 初始化
- query (查询)
- update (更新、删除、插入等)
- transaction (事务)



### 1. FMDatabase初始化

#### init

先来看一个初始化的例子

``` objective-c
NSString *docsPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
NSString *dbPath   = [docsPath stringByAppendingPathComponent:@"test.db"];
FMDatabase *db     = [FMDatabase databaseWithPath:dbPath];
```

找到沙盒内的db文件，然后调用databaseWithPath进行初始化，传入db路径path。

初始化主要有以下几种方式：

```objective-c
+ databaseWithPath:
+ databaseWithURL:
– initWithPath:
– initWithURL:
```

但最后调用的是 `- initWithURL:` 。内部主要是初始化一些变量。

#### open

初始化完毕之后就要打开数据库，调用 `- open`

``` objective-c
FMDatabase *db = [FMDatabase databaseWithPath:[self path]];

if (![db open]) {
  NSLog(@"数据库打开失败");
  return;
};
```

``` c
- (BOOL)open {
    if (_isOpen) {
        return YES;
    }
    
    // if we previously tried to open and it failed, make sure to close it before we try again
    
    if (_db) {
        [self close];
    }
    
    // now open database

    // 打开数据库
    int err = sqlite3_open([self sqlitePath], (sqlite3**)&_db );
    if(err != SQLITE_OK) {
        NSLog(@"error opening!: %d", err);
        return NO;
    }
    
    if (_maxBusyRetryTimeInterval > 0.0) {
        // set the handler
        [self setMaxBusyRetryTimeInterval:_maxBusyRetryTimeInterval];
    }
    
    _isOpen = YES;
    
    return YES;
}
```

首先判断是否已经打开。接着如果db已经存在就先进行close。

打开数据库调用的 `sqlite3_open` 方法，此方式第一个参数为数据库path，第二个参数返回数据库连接句柄*ppDb，存储数据库连接句柄在_db内。

``` c
SQLITE_API int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
```

数据库打开成功返回`SQLITE_OK`，如果失败，可通过`sqlite3_errmsg`获取失败描述。



此外还有两个方法用于打开数据库：

``` objective-c
– openWithFlags:
– openWithFlags:vfs:
```

这个方法内部是通过调用 `sqlite3_open_v2` 来打开数据库。

``` c
SQLITE_API int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
```

`sqlite3_open_v2()` 接口的工作方式与 `sqlite3_open()` 类似，只是它接受两个额外的参数来控制新的数据库连接。`sqlite3_open_v2()` 的flags参数必须至少包含以下三种标志组合之一：

- SQLITE_OPEN_READONLY

  数据库以只读模式打开。如果数据库不存在，则返回错误。

- SQLITE_OPEN_READWRITE

  如果可能，将打开数据库进行读写，或者仅当文件受操作系统写保护时才进行读操作。无论哪种情况，数据库必须已经存在，否则将返回错误。

- SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE

  打开数据库进行读写操作，如果数据库不存在，则创建数据库。这是始终用于sqlite3_open() 和 sqlite3_open16() 的行为。

`sqlite3_open_v2()` 的第四个参数是定义新数据库连接应使用的操作系统接口的sqlite3_vfs对象的名称。如果第四个参数是空指针，则使用默认的sqlite3 vfs对象。

#### close

无论打开数据库连接时是否发生错误，当不再需要时，都应调用 `sqlite3_close` 来释放相关联的资源

``` objective-c
- (BOOL)close {
    // 1. 清空statement对象，释放prepared对象
    [self clearCachedStatements];
    // 2. 清空resultset
    [self closeOpenResultSets];
    
    if (!_db) {
        return YES;
    }
    
    int  rc;
    BOOL retry;
    BOOL triedFinalizingOpenStatements = NO;
    
    // 3. 关闭数据库连接
    do {
        retry   = NO;
        rc      = sqlite3_close(_db);
        if (SQLITE_BUSY == rc || SQLITE_LOCKED == rc) {
            if (!triedFinalizingOpenStatements) {
                triedFinalizingOpenStatements = YES;
                sqlite3_stmt *pStmt;
                while ((pStmt = sqlite3_next_stmt(_db, nil)) !=0) {
                    NSLog(@"Closing leaked statement");
                    sqlite3_finalize(pStmt);
                    retry = YES;
                }
            }
        }
        else if (SQLITE_OK != rc) {
            NSLog(@"error closing!: %d", rc);
        }
    }
    while (retry);
    
    _db = nil;
    _isOpen = false;
    
    return YES;
}
```

close方法内部主要是清空prepared语句和结果集，最后关闭数据库连接



### 2. query(查询)

数据库打开之后，我们就可以执行sql操作了。首先来讲查询操作

``` objective-c
FMResultSet *rs = [db executeQuery:@"select * from city"];
while ([rs next]) {
    NSLog(@"%@", [rs resultDictionary]);
}
```

这里的例子是从city表内查找所有数据，然后执行遍历打印操作

首先来看 `executeQurey:` 方法

``` objective-c
- (FMResultSet *)executeQuery:(NSString*)sql, ... {
    va_list args;
    va_start(args, sql);
    
    id result = [self executeQuery:sql withArgumentsInArray:nil orDictionary:nil orVAList:args];
    
    va_end(args);
    return result;
}
```

内部调用的是 `- executeQuery:withArgumentsInArray:orDictionary:orVAList:` 这个方法，把可变参数列表传给VAList。 到最后我们会发现实际上所有的query操作最后都是调用的这个方法。

``` objective-c
- (FMResultSet *)executeQuery:(NSString *)sql withArgumentsInArray:(NSArray*)arrayArgs orDictionary:(NSDictionary *)dictionaryArgs orVAList:(va_list)args {
    
    // 数据库是否打开
    if (![self databaseExists]) {
        return 0x00;
    }
    
    // 数据库是否正在使用
    if (_isExecutingStatement) {
        [self warnInUse];
        return 0x00;
    }
    
    _isExecutingStatement = YES;
    
    int rc                  = 0x00;
    sqlite3_stmt *pStmt     = 0x00;
    FMStatement *statement  = 0x00;
    FMResultSet *rs         = 0x00;
    
    // 是否需要追踪sql执行状态
    if (_traceExecution && sql) {
        NSLog(@"%@ executeQuery: %@", self, sql);
    }
    
    // 查找缓存
    if (_shouldCacheStatements) {
        statement = [self cachedStatementForQuery:sql];
        pStmt = statement ? [statement statement] : 0x00;
        // reset prepared
        [statement reset];
    }
    
    // 若缓存中没有sql对应的prepared语句，使用sqlite3_prepare_v2函数进行预处理
    if (!pStmt) {
        
        rc = sqlite3_prepare_v2(_db, [sql UTF8String], -1, &pStmt, 0);
        
        // 预处理失败
        if (SQLITE_OK != rc) {
            if (_logsErrors) {
                NSLog(@"DB Error: %d \"%@\"", [self lastErrorCode], [self lastErrorMessage]);
                NSLog(@"DB Query: %@", sql);
                NSLog(@"DB Path: %@", _databasePath);
            }
            
            if (_crashOnErrors) {
                NSAssert(false, @"DB Error: %d \"%@\"", [self lastErrorCode], [self lastErrorMessage]);
                abort();
            }
            
            sqlite3_finalize(pStmt);
            _isExecutingStatement = NO;
            return nil;
        }
    }
    
    id obj;
    int idx = 0;
    // 获取pStmt中需要绑定的参数个数
    int queryCount = sqlite3_bind_parameter_count(pStmt); // pointed out by Dominic Yu (thanks!)
    
    // If dictionaryArgs is passed in, that means we are using sqlite's named parameter support
    if (dictionaryArgs) {
        
        for (NSString *dictionaryKey in [dictionaryArgs allKeys]) {
            
            // Prefix the key with a colon.
            NSString *parameterName = [[NSString alloc] initWithFormat:@":%@", dictionaryKey];
            
            if (_traceExecution) {
                NSLog(@"%@ = %@", parameterName, [dictionaryArgs objectForKey:dictionaryKey]);
            }
            
            // Get the index for the parameter name.
            int namedIdx = sqlite3_bind_parameter_index(pStmt, [parameterName UTF8String]);
            
            FMDBRelease(parameterName);
            
            if (namedIdx > 0) {
                // Standard binding from here.
                [self bindObject:[dictionaryArgs objectForKey:dictionaryKey] toColumn:namedIdx inStatement:pStmt];
                // increment the binding count, so our check below works out
                idx++;
            }
            else {
                NSLog(@"Could not find index for %@", dictionaryKey);
            }
        }
    }
    else {
        
        while (idx < queryCount) {
            
            if (arrayArgs && idx < (int)[arrayArgs count]) {
                obj = [arrayArgs objectAtIndex:(NSUInteger)idx];
            }
            else if (args) {
                obj = va_arg(args, id);
            }
            else {
                //We ran out of arguments
                break;
            }
            
            if (_traceExecution) {
                if ([obj isKindOfClass:[NSData class]]) {
                    NSLog(@"data: %ld bytes", (unsigned long)[(NSData*)obj length]);
                }
                else {
                    NSLog(@"obj: %@", obj);
                }
            }
            
            idx++;
            
            [self bindObject:obj toColumn:idx inStatement:pStmt];
        }
    }
    
    // 若参数个数不对，释放资源 返回
    if (idx != queryCount) {
        NSLog(@"Error: the bind count is not correct for the # of variables (executeQuery)");
        sqlite3_finalize(pStmt);
        _isExecutingStatement = NO;
        return nil;
    }
    
    FMDBRetain(statement); // to balance the release below
    
    // statement不为空，进行缓存
    if (!statement) {
        statement = [[FMStatement alloc] init];
        [statement setStatement:pStmt];
        
        // 使用sql作为key来缓存statement（即sql对应的prepare语句）
        if (_shouldCacheStatements && sql) {
            [self setCachedStatement:statement forQuery:sql];
        }
    }
    
    // the statement gets closed in rs's dealloc or [rs close];
    // 创建FMResultSet对象, 拿此对象获取结果集
    rs = [FMResultSet resultSetWithStatement:statement usingParentDatabase:self];
    [rs setQuery:sql];
    
    NSValue *openResultSet = [NSValue valueWithNonretainedObject:rs];
    [_openResultSets addObject:openResultSet];
    
    [statement setUseCount:[statement useCount] + 1];
    
    FMDBRelease(statement);
    
    _isExecutingStatement = NO;
    
    return rs;
}
```

总结这个方法，简单的分为几步来看

1. 首先会根据sql语句来查找对应的缓存，这个缓存内部存储的是 `FMStatement`这个对象，如果有缓存，根据缓存找到sql对应的编译对象 `sqlite3_stmt` 

   (ps: 每一条sql语句的执行之前，都会将sql编译为对应的sqlite3_stmt对象，也叫做prepared语句对象，prepared语句就像是目标代码一样。由于编译sql比较消耗性能，所以这里进行缓存，下一次可以直接使用编译后的对象)

2. 如果没有缓存，则调用 `sqlite3_prepare_v2()` 对sql进行预编译，生成sqlite3_stmt对象。`sqlite3_prepare_v2()`返回SQLITE_OK则代表成功，如果失败的话调用 `sqlite3_finalize()`释放prepared语句。   

3. 绑定参数。根据arrayArgs、dictionaryArgs、args三个参数来确定绑定参数。

   绑定操作执行完成之后，判断绑定参数是否正确，如果有问题，则报错，然后释放prepared语句。

4. 如果经过上述操作之后，基本已经完成了sql的前期工作。 接下来就是对FMStatement进行缓存，以便下一次操作。

5. 缓存完毕之后，创建FMResultSet对象，绑定对应的sql、prepared语句和当前database对象。至此query方式就完毕了。

   （FMResultSet对象主要是用来存储查询返回的结果集，我们后面获取返回的数据都是在和FMResultSet打交道）



接下来看另外几个query方法，都是针对上面方法的包装

``` 
– executeQuery:
– executeQueryWithFormat:
– executeQuery:withArgumentsInArray:
– executeQuery:values:error:
– executeQuery:withParameterDictionary:
```

最后来看一下几种方法的简单使用就可以更好的理解了

``` objective-c
				// 可变参数 executeQuery
        FMResultSet *rs0 = [db executeQuery:@"select * from city where name=?", @"北京"];
        while ([rs0 next]) {
            NSLog(@"%@", [rs0 resultDictionary]);
        }
        
        // 可以使用oc的格式化进行操作 executeQueryWithFormat
        FMResultSet *rs1 = [db executeQueryWithFormat:@"select * from city where name=%@", @"北京"];
        while ([rs1 next]) {
            NSLog(@"%@", [rs1 resultDictionary]);
        }
        
        // array  executeQuery:withArgumentsInArray:
        FMResultSet *rs2 = [db executeQuery:@"select * from city where name=?" withArgumentsInArray:@[@"北京"]];
        while ([rs2 next]) {
            NSLog(@"%@", [rs2 resultDictionary]);
        }
        
        // dic  executeQuery:withParameterDictionary:
        NSMutableDictionary *dic = [NSMutableDictionary dictionary];
        dic[@"name"] = @"北京";
        FMResultSet *rs3 = [db executeQuery:@"select * from city where name=:name" 	withParameterDictionary:dic];
        while ([rs3 next]) {
            NSLog(@"%@", [rs3 resultDictionary]);
        }
```



### 3. update(插入、更新、删除)

上面第二部分的query主要是针对select方法。 如果我们要进行update、insert into、delete操作，则要调用 `executeUpdate:` 方法，换句话说，除了select语句，其他的都调用 `executeUpdate:` 方法 

``` objective-c
BOOL rs = [db executeUpdate:@"insert into city (id, code, name, level, parentCode, first_char) values (?, ?, ?, ?, ?, ?)" , @1, @(1), @"测试城市", @3, @0, @"c"];
```

这里例子是向city表内插入一行数据，调用的 `executeUpdate:` 方法

``` objective-c
- (BOOL)executeUpdate:(NSString*)sql, ... {
    va_list args;
    va_start(args, sql);
    
    BOOL result = [self executeUpdate:sql error:nil withArgumentsInArray:nil orDictionary:nil orVAList:args];
    
    va_end(args);
    return result;
}
```

由于不是查询语句，不需要返回数据。所以此方法的返回值是BOOL，只是告诉调用方成功与否。

可以看到这个方法内部调用的是 `- executeUpdate:error: withArgumentsInArray:orDictionary:orVAList: `

``` objective-c
- (BOOL)executeUpdate:(NSString*)sql error:(NSError * _Nullable __autoreleasing *)outErr withArgumentsInArray:(NSArray*)arrayArgs orDictionary:(NSDictionary *)dictionaryArgs orVAList:(va_list)args {
    
    // 检查db打开状态
    if (![self databaseExists]) {
        return NO;
    }
    
    // 是否正在执行中
    if (_isExecutingStatement) {
        [self warnInUse];
        return NO;
    }
    
    _isExecutingStatement = YES;
    
    int rc                   = 0x00;
    sqlite3_stmt *pStmt      = 0x00;  // 编译好的sql准备语句的指针
    FMStatement *cachedStmt  = 0x00;
    
    // 是否跟踪sql执行状态
    if (_traceExecution && sql) {
        NSLog(@"%@ executeUpdate: %@", self, sql);
    }
    
    // 缓存
    if (_shouldCacheStatements) {
        cachedStmt = [self cachedStatementForQuery:sql];
        pStmt = cachedStmt ? [cachedStmt statement] : 0x00;
        [cachedStmt reset];
    }
    
    // 没有缓存指针，创建
    if (!pStmt) {
        // 预编译 将sql命令转为成一个准备对象，并返回这个对象的指针 &pStmt
        rc = sqlite3_prepare_v2(_db, [sql UTF8String], -1, &pStmt, 0);
        
        // 不成功，则执行一系列日志打印操作，释放指针
        if (SQLITE_OK != rc) {
            if (_logsErrors) {
                NSLog(@"DB Error: %d \"%@\"", [self lastErrorCode], [self lastErrorMessage]);
                NSLog(@"DB Query: %@", sql);
                NSLog(@"DB Path: %@", _databasePath);
            }
            
            if (_crashOnErrors) {
                NSAssert(false, @"DB Error: %d \"%@\"", [self lastErrorCode], [self lastErrorMessage]);
                abort();
            }
            
            if (outErr) {
                *outErr = [self errorWithMessage:[NSString stringWithUTF8String:sqlite3_errmsg(_db)]];
            }
            
            // 销毁sqlite3_prepare_v2的准备语句，防止内存泄露
            sqlite3_finalize(pStmt);
            
            _isExecutingStatement = NO;
            return NO;
        }
    }
    
    id obj;
    int idx = 0;
    int queryCount = sqlite3_bind_parameter_count(pStmt);
    
    // If dictionaryArgs is passed in, that means we are using sqlite's named parameter support
    if (dictionaryArgs) {
        
        for (NSString *dictionaryKey in [dictionaryArgs allKeys]) {
            
            // Prefix the key with a colon.
            NSString *parameterName = [[NSString alloc] initWithFormat:@":%@", dictionaryKey];
            
            if (_traceExecution) {
                NSLog(@"%@ = %@", parameterName, [dictionaryArgs objectForKey:dictionaryKey]);
            }
            // Get the index for the parameter name.
            // sqlite3_bind_parameter_inde() 根据name找出index
            int namedIdx = sqlite3_bind_parameter_index(pStmt, [parameterName UTF8String]);
            
            FMDBRelease(parameterName);
            
            // 若找出索引，拿index和值绑定
            if (namedIdx > 0) {
                // Standard binding from here.
                [self bindObject:[dictionaryArgs objectForKey:dictionaryKey] toColumn:namedIdx inStatement:pStmt];
                
                // increment the binding count, so our check below works out
                idx++;
            }
            else {
                NSString *message = [NSString stringWithFormat:@"Could not find index for %@", dictionaryKey];
                
                if (_logsErrors) {
                    NSLog(@"%@", message);
                }
                if (outErr) {
                    *outErr = [self errorWithMessage:message];
                }
            }
        }
    }
    else {
        
        while (idx < queryCount) {
            
            // 如果是数组，分别取出每一个元素绑定
            if (arrayArgs && idx < (int)[arrayArgs count]) {
                obj = [arrayArgs objectAtIndex:(NSUInteger)idx];
            }
            else if (args) { // 可变列表取出每一个绑定
                obj = va_arg(args, id);
            }
            else {
                //We ran out of arguments
                break;
            }
            
            if (_traceExecution) {
                if ([obj isKindOfClass:[NSData class]]) {
                    NSLog(@"data: %ld bytes", (unsigned long)[(NSData*)obj length]);
                }
                else {
                    NSLog(@"obj: %@", obj);
                }
            }
            
            idx++;
            
            // 绑定参数
            [self bindObject:obj toColumn:idx inStatement:pStmt];
        }
    }
    
    
    // 实际绑定的参数个数 != 实际参数个数
    if (idx != queryCount) {
        NSString *message = [NSString stringWithFormat:@"Error: the bind count (%d) is not correct for the # of variables in the query (%d) (%@) (executeUpdate)", idx, queryCount, sql];
        if (_logsErrors) {
            NSLog(@"%@", message);
        }
        if (outErr) {
            *outErr = [self errorWithMessage:message];
        }
        
        sqlite3_finalize(pStmt);
        _isExecutingStatement = NO;
        return NO;
    }
    
    /* Call sqlite3_step() to run the virtual machine. Since the SQL being
     ** executed is not a SELECT statement, we assume no data will be returned.
     */
    
    // 预编译之后就可以通过setup执行
    rc      = sqlite3_step(pStmt);
    
    if (SQLITE_DONE == rc) { // 执行完成
        // all is well, let's return.
    }
    else if (SQLITE_INTERRUPT == rc) { // 执行中断
        if (_logsErrors) {
            NSLog(@"Error calling sqlite3_step. Query was interrupted (%d: %s) SQLITE_INTERRUPT", rc, sqlite3_errmsg(_db));
            NSLog(@"DB Query: %@", sql);
        }
    }
    else if (rc == SQLITE_ROW) { // 报错，说明这是一条select语句
        NSString *message = [NSString stringWithFormat:@"A executeUpdate is being called with a query string '%@'", sql];
        if (_logsErrors) {
            NSLog(@"%@", message);
            NSLog(@"DB Query: %@", sql);
        }
        if (outErr) {
            *outErr = [self errorWithMessage:message];
        }
    }
    else { // 其他失败情况
        if (outErr) {
            *outErr = [self errorWithMessage:[NSString stringWithUTF8String:sqlite3_errmsg(_db)]];
        }
        
        if (SQLITE_ERROR == rc) {
            if (_logsErrors) {
                NSLog(@"Error calling sqlite3_step (%d: %s) SQLITE_ERROR", rc, sqlite3_errmsg(_db));
                NSLog(@"DB Query: %@", sql);
            }
        }
        else if (SQLITE_MISUSE == rc) {
            // uh oh.
            if (_logsErrors) {
                NSLog(@"Error calling sqlite3_step (%d: %s) SQLITE_MISUSE", rc, sqlite3_errmsg(_db));
                NSLog(@"DB Query: %@", sql);
            }
        }
        else {
            // wtf?
            if (_logsErrors) {
                NSLog(@"Unknown error calling sqlite3_step (%d: %s) eu", rc, sqlite3_errmsg(_db));
                NSLog(@"DB Query: %@", sql);
            }
        }
    }
    
    // 需要缓存且未被缓存，则执行缓存
    if (_shouldCacheStatements && !cachedStmt) {
        cachedStmt = [[FMStatement alloc] init];
        
        [cachedStmt setStatement:pStmt];
        
        [self setCachedStatement:cachedStmt forQuery:sql];
        
        FMDBRelease(cachedStmt);
    }
    
    int closeErrorCode;
    
    // 被缓存之后，恢复pStmt对象
    if (cachedStmt) {
        [cachedStmt setUseCount:[cachedStmt useCount] + 1];
        closeErrorCode = sqlite3_reset(pStmt);
    }
    else {
        /* Finalize the virtual machine. This releases all memory and other
         ** resources allocated by the sqlite3_prepare() call above.
         */
        // 释放pStmt
        closeErrorCode = sqlite3_finalize(pStmt);
    }
    
    if (closeErrorCode != SQLITE_OK) {
        if (_logsErrors) {
            NSLog(@"Unknown error finalizing or resetting statement (%d: %s)", closeErrorCode, sqlite3_errmsg(_db));
            NSLog(@"DB Query: %@", sql);
        }
    }
    
    _isExecutingStatement = NO;
    return (rc == SQLITE_DONE || rc == SQLITE_OK);
}
```

此方法是query的方法调用逻辑大体是一样的，也大致分为了几步：

1. 查找缓存FMStatement对象，给prepared语句赋值

2. 若prepared语句对象为空，则调用 `sqlite3_prepare_v2()` 进行编译，获取prepared语句对象（sqlite3_stmt）

3. 根据参数列表绑定sql参数

4. 这一步跟query就不一样了。由于这里是立马就要执行，所以调用了 `sqlite3_step()` 方法

   (sqlite3_step: 在生成prepared语句之后，不要调用此函数一次或多次以计算sql语句，返回 `SQLITE_DONE` 表示语句已经完成执行，返回 `SQLITE_ROW` 代表这是一条select语句，还会有下一条结果)

5. 执行完step之后就表示操作已经完成了，这时候跟query一样，进行缓存

6. 如果step之后返回值为 `SQLITE_DONE` || `SQLITE_OK` 则此方法返回YES，反之为NO，执行失败。



接下来看另外几个update方法，也是根据上面的方法进行的包装方法

``` 
– executeUpdate:
– executeUpdateWithFormat:
– executeUpdate:withArgumentsInArray:
– executeUpdate:values:error:
– executeUpdate:withParameterDictionary:
– executeUpdate:withVAList:
```

``` objective-c
        [db executeUpdate:@"insert into city (id, code, name, level, parentCode, first_char) values (?, ?, ?, ?, ?, ?)" , @1, @(1), @"测试城市", @3, @0, @"c"];

        
        NSMutableDictionary *dictionaryArgs = [NSMutableDictionary dictionary];
        dictionaryArgs[@"id"] = @1;
        dictionaryArgs[@"code"] = @1;
        dictionaryArgs[@"name"] = @"哈哈哈";
        dictionaryArgs[@"level"] = @3;
        dictionaryArgs[@"parentCode"] = @0;
        dictionaryArgs[@"first_char"] = @"h";

        [db executeUpdate:@"insert into city (id, code, name, level, parentCode, first_char) values (:id, :code, :name, :level, :parentCode, :first_char)" withParameterDictionary:dictionaryArgs];
        
        
        
        NSArray *ary = @[@1, @(1), @"测试城市", @3, @0, @"c"];
        [db executeUpdate:@"insert into city (id, code, name, level, parentCode, first_char) values (?, ?, ?, ?, ?, ?)" withArgumentsInArray:ary];


        
        BOOL success = [db executeUpdateWithFormat:@"insert into city (id, code, name, level, parentCode, first_char) values (%d, %d, %@, %d, %d, %@)", 1, 1, @"哈哈哈", 2, 0, @"h"];
```



还有一个方法需要单独看一下

`- executeStatements:`

``` objective-c
- (BOOL)executeStatements:(NSString *)sql {
    return [self executeStatements:sql withResultBlock:nil];
}

/// 可以执行多条sql语句，内部是调用sqlite3_exec()
- (BOOL)executeStatements:(NSString *)sql withResultBlock:(__attribute__((noescape)) FMDBExecuteStatementsCallbackBlock)block {
    
    int rc;
    char *errmsg = nil;
    
    // 返回结果在 FMDBExecuteBulkSQLCallback 函数内部处理
    rc = sqlite3_exec([self sqliteHandle], [sql UTF8String], block ? FMDBExecuteBulkSQLCallback : nil, (__bridge void *)(block), &errmsg);
    
    if (errmsg && [self logsErrors]) {
        NSLog(@"Error inserting batch: %s", errmsg);
    }
    if (errmsg) {
        sqlite3_free(errmsg);
    }
    
    return (rc == SQLITE_OK);
}
```

这个方法是用来执行多条sql语句的，内部调用的是 `sqlite3_exec()`  函数。

`sqlite3_exec`： 是prepare、step、finalize几个方法的包装，用来运行多个sql语句，无需使用大量c代码

``` c
SQLITE_API int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
```

第三个参数是执行结果的回调

``` objective-c
int FMDBExecuteBulkSQLCallback(void *theBlockAsVoid, int columns, char **values, char **names); // shhh clang.
int FMDBExecuteBulkSQLCallback(void *theBlockAsVoid, int columns, char **values, char **names) {
    
    if (!theBlockAsVoid) {
        return SQLITE_OK;
    }
    
    int (^execCallbackBlock)(NSDictionary *resultsDictionary) = (__bridge int (^)(NSDictionary *__strong))(theBlockAsVoid);
    
    NSMutableDictionary *dictionary = [NSMutableDictionary dictionaryWithCapacity:(NSUInteger)columns];
    
    for (NSInteger i = 0; i < columns; i++) {
        NSString *key = [NSString stringWithUTF8String:names[i]];
        id value = values[i] ? [NSString stringWithUTF8String:values[i]] : [NSNull null];
        value = value ? value : [NSNull null];
        [dictionary setObject:value forKey:key];
    }
    
    return execCallbackBlock(dictionary);
}
```

回调中第二个参数columns代表返回结果的列数

第三个参数`**values` 指向字段值的指针数组，是通过`sqlite3_column_text()`获取的

第四个参数`**names` 指向字段名称的指针数组，是通过`sqlite3_column_name()` 获取的



### 4. Transaction (事务)

``` objective-c
- (BOOL)rollback {
    BOOL b = [self executeUpdate:@"rollback transaction"];
    
    if (b) {
        _isInTransaction = NO;
    }
    
    return b;
}

- (BOOL)commit {
    BOOL b =  [self executeUpdate:@"commit transaction"];
    
    if (b) {
        _isInTransaction = NO;
    }
    
    return b;
}

- (BOOL)beginTransaction {
    
    BOOL b = [self executeUpdate:@"begin exclusive transaction"];
    if (b) {
        _isInTransaction = YES;
    }
    
    return b;
}
```

这里的事务处理比较简单，主要就是调用executeUpdate方法执行事务sql。 



## FMStatement

``` objective-c
/** Usage count */

@property (atomic, assign) long useCount;

/** SQL statement */

@property (atomic, retain) NSString *query;

/** SQLite sqlite3_stmt
 
 @see [`sqlite3_stmt`](http://www.sqlite.org/c3ref/stmt.html)
 */

@property (atomic, assign) void *statement;

/** Indication of whether the statement is in use */

@property (atomic, assign) BOOL inUse;

///----------------------------
/// @name Closing and Resetting
///----------------------------

/** Close statement */

- (void)close;

/** Reset statement */

- (void)reset;
```

主要是针对sqlite3_stmt的包装类，FMDatabase中缓存的类就是此类

### close

``` objective-c
/// 释放prepared语句资源，避免内存泄露
- (void)close {
    if (_statement) {
        sqlite3_finalize(_statement);
        _statement = 0x00;
    }
    
    _inUse = NO;
}
```

close释放prepared语句，使用sqlite3_finalize()释放资源

### reset

``` objective-c
/// 重置prepared语句
- (void)reset {
    if (_statement) {
        sqlite3_reset(_statement);
    }
    
    _inUse = NO;
}
```

reset重置prepared语句，回到编译之后的状态，可重新绑定参数使用



## FMResultSet

表示在FMDatabase上执行查询的结果

### next

主要方法就是next

``` objective-c
- (BOOL)next {
    return [self nextWithError:nil];
}

/*
 * 调用sqlite3_step获取一次结果，
 如果结果是 SQLITE_DONE 说明获取完成
 如果结果是 SQLITE_ROW 说明是查询数据，一行一行返回，还有下一次结果
 */
- (BOOL)nextWithError:(NSError * _Nullable __autoreleasing *)outErr {
    
    int rc = sqlite3_step([_statement statement]);
    
    if (SQLITE_BUSY == rc || SQLITE_LOCKED == rc) {
        NSLog(@"%s:%d Database busy (%@)", __FUNCTION__, __LINE__, [_parentDB databasePath]);
        NSLog(@"Database busy");
        if (outErr) {
            *outErr = [_parentDB lastError];
        }
    }
    else if (SQLITE_DONE == rc || SQLITE_ROW == rc) {
        // all is well, let's return.
    }
    else if (SQLITE_ERROR == rc) {
        NSLog(@"Error calling sqlite3_step (%d: %s) rs", rc, sqlite3_errmsg([_parentDB sqliteHandle]));
        if (outErr) {
            *outErr = [_parentDB lastError];
        }
    }
    else if (SQLITE_MISUSE == rc) {
        // uh oh.
        NSLog(@"Error calling sqlite3_step (%d: %s) rs", rc, sqlite3_errmsg([_parentDB sqliteHandle]));
        if (outErr) {
            if (_parentDB) {
                *outErr = [_parentDB lastError];
            }
            else {
                // If 'next' or 'nextWithError' is called after the result set is closed,
                // we need to return the appropriate error.
                NSDictionary* errorMessage = [NSDictionary dictionaryWithObject:@"parentDB does not exist" forKey:NSLocalizedDescriptionKey];
                *outErr = [NSError errorWithDomain:@"FMDatabase" code:SQLITE_MISUSE userInfo:errorMessage];
            }
            
        }
    }
    else {
        // wtf?
        NSLog(@"Unknown error calling sqlite3_step (%d: %s) rs", rc, sqlite3_errmsg([_parentDB sqliteHandle]));
        if (outErr) {
            *outErr = [_parentDB lastError];
        }
    }
    
    
    if (rc != SQLITE_ROW) {
        [self close];
    }
    
    return (rc == SQLITE_ROW);
}
```

用于执行查询操作之后，调用next获取数据

``` objective-c
FMResultSet *rs = [db executeQuery:@"select * from city"];
while ([rs next]) {
    NSLog(@"%@", [rs resultDictionary]);
}
```

每调一次next成功之后，就可以用rs获取数据，有很多种方式来获取数据

### resultDictionary

``` objective-c
/// 结果转换为dictionary
- (NSDictionary*)resultDictionary {
    
    // sqlite3_data_count 返回当前正在执行的sql的列数，没有结果则返回0
    NSUInteger num_cols = (NSUInteger)sqlite3_data_count([_statement statement]);
    
    if (num_cols > 0) {
        // 根据列数创建字典
        NSMutableDictionary *dict = [NSMutableDictionary dictionaryWithCapacity:num_cols];
        
        // sqlite3_column_count 返回所有列数，跟sql执行状态没关系
        int columnCount = sqlite3_column_count([_statement statement]);
        
        int columnIdx = 0;
        for (columnIdx = 0; columnIdx < columnCount; columnIdx++) {
            // 获取列名称
            NSString *columnName = [NSString stringWithUTF8String:sqlite3_column_name([_statement statement], columnIdx)];
            // 获取value
            id objectValue = [self objectForColumnIndex:columnIdx];
            [dict setObject:objectValue forKey:columnName];
        }
        
        return dict;
    }
    else {
        NSLog(@"Warning: There seem to be no columns in this set.");
    }
    
    return nil;
}
```

遍历所有行，然后获取列名和值，包装成为一个dictionary返回

### xxxForColumn:

``` 
– intForColumn:
– longForColumn:
– longLongIntForColumn:
– unsignedLongLongIntForColumn:
– boolForColumn:
– doubleForColumn:
– stringForColumn:
– dateForColumn:
– dataForColumn:
...
```

这一系列方法是根据列名称获取结果值，但实际内部是把列名转换为index，然后再获取值

``` objective-c
- (int)columnIndexForName:(NSString*)columnName {
    columnName = [columnName lowercaseString];
    
    NSNumber *n = [[self columnNameToIndexMap] objectForKey:columnName];
    
    if (n != nil) {
        return [n intValue];
    }
    
    NSLog(@"Warning: I could not find the column named '%@'.", columnName);
    
    return -1;
}

- (NSMutableDictionary *)columnNameToIndexMap {
    if (!_columnNameToIndexMap) {
        int columnCount = sqlite3_column_count([_statement statement]);
        _columnNameToIndexMap = [[NSMutableDictionary alloc] initWithCapacity:(NSUInteger)columnCount];
        int columnIdx = 0;
        for (columnIdx = 0; columnIdx < columnCount; columnIdx++) {
            [_columnNameToIndexMap setObject:[NSNumber numberWithInt:columnIdx]
                                      forKey:[[NSString stringWithUTF8String:sqlite3_column_name([_statement statement], columnIdx)] lowercaseString]];
        }
    }
    return _columnNameToIndexMap;
}
```



### xxxForColumnIndex:

```
– intForColumnIndex:
– longForColumnIndex:
– longLongIntForColumnIndex:
– unsignedLongLongIntForColumnIndex:
– boolForColumnIndex:
– doubleForColumnIndex:
– stringForColumnIndex:
– dateForColumnIndex:
– dataForColumnIndex:
```

通过index获取对应的值。内部调用的是 `sqlite3_column_xxx()` 方法

``` c
SQLITE_API const void *sqlite3_column_blob(sqlite3_stmt*, int iCol);
SQLITE_API double sqlite3_column_double(sqlite3_stmt*, int iCol);
SQLITE_API int sqlite3_column_int(sqlite3_stmt*, int iCol);
SQLITE_API sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol);
SQLITE_API const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);
SQLITE_API const void *sqlite3_column_text16(sqlite3_stmt*, int iCol);
SQLITE_API sqlite3_value *sqlite3_column_value(sqlite3_stmt*, int iCol);
SQLITE_API int sqlite3_column_bytes(sqlite3_stmt*, int iCol);
SQLITE_API int sqlite3_column_bytes16(sqlite3_stmt*, int iCol);
SQLITE_API int sqlite3_column_type(sqlite3_stmt*, int iCol);
```

第一个参数就是prepared对象

第二个参数是index，最左边为0

```
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

上图为转换的结果



## FMDatabaseQueue

如果需要在多线程中执行操作，就需要使用FMDatabaseQueue，实际项目中建议都使用此类，它是线程安全的。

``` objective-c
    FMDatabaseQueue *dbQueue = [FMDatabaseQueue databaseQueueWithPath:[self path]];
    
    [dbQueue inDatabase:^(FMDatabase * _Nonnull db) {
        NSLog(@"11111111");
        sleep(1);
        
        FMResultSet *rs = [db executeQuery:@"select rowid, * from city"];
        while ([rs next]) {
            NSLog(@"%@", [rs resultDictionary]);
        }
        
        NSLog(@"222222222");
    }];
    
    [dbQueue inDatabase:^(FMDatabase * _Nonnull db) {
        NSLog(@"333333");
        
        // 可变参数
        FMResultSet *rs0 = [db executeQuery:@"select * from city where name=?", @"北京"];
        while ([rs0 next]) {
            NSLog(@"%@", [rs0 resultDictionary]);
        }

        NSLog(@"444444");
    }];

    
    [dbQueue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
      
        BOOL success = [db executeUpdateWithFormat:@"insert into city (id, code, name, level, parentCode, first_char) values (%d, %d, %@, %d, %d, %@)", 1, 1, @"哈哈哈", 2, 0, @"h"];
        
        
        if (!success) {
            *rollback = YES;
            return;
        }
        
        FMResultSet *rs = [db executeQuery:@"SELECT * FROM city"];
        while ([rs next]) {
            NSLog(@"%@", [rs resultDictionary]);
        }
        [rs close];
    }];
```

分为两部分讲

- 初始化
- 常用方法

### FMDatabaseQueue初始化

``` 
+ databaseQueueWithPath:
+ databaseQueueWithURL:
+ databaseQueueWithPath:flags:
+ databaseQueueWithURL:flags:
– initWithPath:
– initWithURL:
– initWithPath:flags:
– initWithURL:flags:
– initWithPath:flags:vfs:
– initWithURL:flags:vfs:
+ databaseClass
```

可以看到这么多方法，但最后都是调用的 `– initWithURL:flags:vfs:` 

``` objective-c
- (instancetype)initWithPath:(NSString*)aPath flags:(int)openFlags vfs:(NSString *)vfsName {
    self = [super init];
    
    if (self != nil) {
        
        _db = [[[self class] databaseClass] databaseWithPath:aPath];
        FMDBRetain(_db);
        
#if SQLITE_VERSION_NUMBER >= 3005000
        BOOL success = [_db openWithFlags:openFlags vfs:vfsName];
#else
        BOOL success = [_db open];
#endif
        if (!success) {
            NSLog(@"Could not create database queue for path %@", aPath);
            FMDBRelease(self);
            return 0x00;
        }
        
        _path = FMDBReturnRetained(aPath);
        
        _queue = dispatch_queue_create([[NSString stringWithFormat:@"fmdb.%@", self] UTF8String], NULL);
        dispatch_queue_set_specific(_queue, kDispatchQueueSpecificKey, (__bridge void *)self, NULL);
        _openFlags = openFlags;
        _vfsName = [vfsName copy];
    }
    
    return self;
}
```

1. 调用FMDatabase生成数据库对象_db
2. 执行 `[_db open]` 打开数据库
3. 创建一个串行队列 _queue
4. dispatch_queue_set_specific设置队列的私有数据，把self和`_queue`绑定一起，后面可以通过dispatch_get_specific获取到当前`_queue`队列绑定的self对象

### 常用操作方法

#### - inDatabase:

``` objective-c
- (void)inDatabase:(__attribute__((noescape)) void (^)(FMDatabase *db))block {
#ifndef NDEBUG
    /* Get the currently executing queue (which should probably be nil, but in theory could be another DB queue
     * and then check it against self to make sure we're not about to deadlock. */
    FMDatabaseQueue *currentSyncQueue = (__bridge id)dispatch_get_specific(kDispatchQueueSpecificKey);
    assert(currentSyncQueue != self && "inDatabase: was called reentrantly on the same queue, which would lead to a deadlock");
#endif
    
    FMDBRetain(self);
    
    // 同步串行队列
    dispatch_sync(_queue, ^() {
        
        // db 已经open
        FMDatabase *db = [self database];
        
        block(db);
        
        if ([db hasOpenResultSets]) {
            NSLog(@"Warning: there is at least one open result set around after performing [FMDatabaseQueue inDatabase:]");
            
#if defined(DEBUG) && DEBUG
            NSSet *openSetCopy = FMDBReturnAutoreleased([[db valueForKey:@"_openResultSets"] copy]);
            for (NSValue *rsInWrappedInATastyValueMeal in openSetCopy) {
                FMResultSet *rs = (FMResultSet *)[rsInWrappedInATastyValueMeal pointerValue];
                NSLog(@"query: '%@'", [rs query]);
            }
#endif
        }
    });
    
    FMDBRelease(self);
}
```

执行数据库操作

1. 开启一个同步串行队列
2. 获取db对象，如果没有打开数据库，则执行open操作
3. 把db对象通过block传给调用方执行相应操作



#### - inTransaction:

``` objective-c
/// 开启事务
- (void)inTransaction:(__attribute__((noescape)) void (^)(FMDatabase *db, BOOL *rollback))block {
    [self beginTransaction:FMDBTransactionExclusive withBlock:block];
}

- (void)inDeferredTransaction:(__attribute__((noescape)) void (^)(FMDatabase *db, BOOL *rollback))block {
    [self beginTransaction:FMDBTransactionDeferred withBlock:block];
}

- (void)inExclusiveTransaction:(__attribute__((noescape)) void (^)(FMDatabase *db, BOOL *rollback))block {
    [self beginTransaction:FMDBTransactionExclusive withBlock:block];
}

- (void)inImmediateTransaction:(__attribute__((noescape)) void (^)(FMDatabase * _Nonnull, BOOL * _Nonnull))block {
    [self beginTransaction:FMDBTransactionImmediate withBlock:block];
}
```

使用inTransaction可以执行事务操作

``` objective-c
/// 开始事务处理
- (void)beginTransaction:(FMDBTransaction)transaction withBlock:(void (^)(FMDatabase *db, BOOL *rollback))block {
    FMDBRetain(self);
    dispatch_sync(_queue, ^() { 
        
        BOOL shouldRollback = NO;

        // 根据事务模式，执行事务begin操作
        switch (transaction) {
            case FMDBTransactionExclusive:
                [[self database] beginTransaction];
                break;
            case FMDBTransactionDeferred:
                [[self database] beginDeferredTransaction];
                break;
            case FMDBTransactionImmediate:
                [[self database] beginImmediateTransaction];
                break;
        }
        
        // begin 之后 交给block处理数据
        block([self database], &shouldRollback);
        
        // 如果rollback=yes，执行 `rollback transaction`
        // 反之正常commit，执行 `commit transaction`
        if (shouldRollback) {
            [[self database] rollback];
        }
        else {
            [[self database] commit];
        }
    });
    
    FMDBRelease(self);
}
```

1. 开启同步串行队列
2. 根据事务的type执行对应的事务开启
3. 把db对象和&rollback通过blcok传给调用方
4. 根据rollback状态，如果rollback，则执行db的rollback，`rollback transaction`回退操作。如果rollback==NO，则执行commit，`commit transaction`正常提交

下面看一下事务的几种模式：

``` objective-c
typedef NS_ENUM(NSInteger, FMDBTransaction) {
    /// 事务开始执行，就获取EXCLUSIVE锁。这时，其他连接无法进行任何读写操作
    FMDBTransactionExclusive,
    /// 事务开始执行时，不预先获取任何锁。当进行读操作，获取SHARED LOCK锁；当进行第一次写操作，获取RESERVED锁
    FMDBTransactionDeferred,
    /// 事务开始执行，就获取RESERVED锁。这时，其他连接只能进行读操作
    FMDBTransactionImmediate,
};
```

`inTrannsaction:` 默认调用的是FMDBTransactionExclusive锁，其他连接无法执行



#### - inSavePoint:

事务设置保留点，可以自定义方便回退

``` objective-c
- (NSError*)inSavePoint:(__attribute__((noescape)) void (^)(FMDatabase *db, BOOL *rollback))block {
#if SQLITE_VERSION_NUMBER >= 3007000
    static unsigned long savePointIdx = 0;
    __block NSError *err = 0x00;
    FMDBRetain(self);
    dispatch_sync(_queue, ^() { 
        
        NSString *name = [NSString stringWithFormat:@"savePoint%ld", savePointIdx++];
        
        BOOL shouldRollback = NO;
        
        // 1. 执行 " savepoint 'dbSavePointx' " 设置保留点
        if ([[self database] startSavePointWithName:name error:&err]) {
            
            // 2. 回调是否rollback
            block([self database], &shouldRollback);
            
            // 3. 回退 执行 " rollback transaction to savepoint 'dbSavePointx' "
            if (shouldRollback) {
                // We need to rollback and release this savepoint to remove it
                [[self database] rollbackToSavePointWithName:name error:&err];
            }
            // 4. 释放保留点 " release savepoint 'dbSavePintx' "
            [[self database] releaseSavePointWithName:name error:&err];
            
        }
    });
    FMDBRelease(self);
    return err;
#else
    NSString *errorMessage = NSLocalizedStringFromTable(@"Save point functions require SQLite 3.7", @"FMDB", nil);
    if (_db.logsErrors) NSLog(@"%@", errorMessage);
    return [NSError errorWithDomain:@"FMDatabase" code:0 userInfo:@{NSLocalizedDescriptionKey : errorMessage}];
#endif
}
```

1. 执行 `savepoint 'dbSavePointx' ` 设置保留点
2. 回调是否rollback
3. 回退 执行 ` rollback transaction to savepoint 'dbSavePointx' `
4. 释放保留点 ` release savepoint 'dbSavePintx' `



## 参考资料

[FMDB github](https://github.com/ccgus/fmdb)

[FMDB Reference](http://ccgus.github.io/fmdb/html/index.html)

[sqlite3](https://sqlite.org/c3ref/intro.html)

[sqlite常用api](https://github.com/gaonian/HexoDocument/blob/master/iOS/SQLite/SQLite%E5%B8%B8%E7%94%A8API(C).md)


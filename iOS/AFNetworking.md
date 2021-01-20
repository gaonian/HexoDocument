## 简述

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cecd02d283dc4addaddab2f9ac5c2e84~tplv-k3u1fbpfcp-watermark.image)

AFN是iOS上的网络封装库，以下解析的源码版本为3.2.1。AFN共分为五个模块，`NSURLSession`、 `Reachability`、 `Security` 、`Serialization` 、`UIKit` 



## NSURLSession

NSURLSession模块是整个库的核心。主要是封装了苹果NSURLSession的api。其中`AFURLSessionManager`是主要实现类。`AFHTTPSessionManager` 是 `AFURLSessionManager` 的子类，内部主要封装了http调用的简单方法。若想实现其他应用层协议，可仿照 `AFHTTPSessionManager` 的方式来自行实现

我们首先通过一个简单的使用从 `AFHTTPSessionManager` 类入手开始分析



### AFHTTPSessionManager

从类的头文件可以看出封装了一系列的请求方法，支持GET、POST、HEAD、PUT、PATCH、DELETE六种method。由于平时用的最多的就是GET和POST，所以供我们选择的方法就剩下了6个

```objective-c
// 两个GET请求方法
- (nullable NSURLSessionDataTask *)GET:(NSString *)URLString
                   parameters:(nullable id)parameters
                      success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                      failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure DEPRECATED_ATTRIBUTE;

。。。

 
// 4个POST请求方法 不一一列出了
- (nullable NSURLSessionDataTask *)POST:(NSString *)URLString
                    parameters:(nullable id)parameters
                       success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                       failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure DEPRECATED_ATTRIBUTE;
。。。。
。。。。
```

我们这里选择一个最简单的get请求方法

```objective-c
    AFHTTPSessionManager *mgr = [AFHTTPSessionManager manager];
    [mgr GET:@"http://c.m.163.com/nc/video/home/1-10.html" parameters:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
    }];
```

1. 首先进行初始化，最终都调用下面方法，内部调用父类`AFURLSessionManager` 的初始化方法

   ```objective-c
   - (instancetype)initWithBaseURL:(NSURL *)url
              sessionConfiguration:(NSURLSessionConfiguration *)configuration
   {
       self = [super initWithSessionConfiguration:configuration];
       if (!self) {
           return nil;
       }
   
       // Ensure terminal slash for baseURL path, so that NSURL +URLWithString:relativeToURL: works as expected
       if ([[url path] length] > 0 && ![[url absoluteString] hasSuffix:@"/"]) {
           url = [url URLByAppendingPathComponent:@""];
       }
   
       self.baseURL = url;
   
       self.requestSerializer = [AFHTTPRequestSerializer serializer];
       self.responseSerializer = [AFJSONResponseSerializer serializer];
   
       return self;
   }
   ```

2. 执行具体get请求，跟源码进去可看到最终调用的也是父类的dataTaskWithHTTPMethod：方法

   ```
   - (NSURLSessionDataTask *)GET:(NSString *)URLString
                      parameters:(id)parameters
                        progress:(void (^)(NSProgress * _Nonnull))downloadProgress
                         success:(void (^)(NSURLSessionDataTask * _Nonnull, id _Nullable))success
                         failure:(void (^)(NSURLSessionDataTask * _Nullable, NSError * _Nonnull))failure
   {
   
       NSURLSessionDataTask *dataTask = [self dataTaskWithHTTPMethod:@"GET"
                                                           URLString:URLString
                                                          parameters:parameters
                                                      uploadProgress:nil
                                                    downloadProgress:downloadProgress
                                                             success:success
                                                             failure:failure];
   
       [dataTask resume];
   
       return dataTask;
   }
   ```

总结：`AFHTTPSessionManager` 类主要就是针对 `AFURLSessionManager` 封装的方便业务方使用的子类。可根据需要选择请求的方法以及请求格式和返回格式的自定义等



### AFURLSessionManager

```objective-c
- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration {
    self = [super init];
    if (!self) {
        return nil;
    }

    if (!configuration) {
        configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    }

    self.sessionConfiguration = configuration;

    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;

    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];

    self.responseSerializer = [AFJSONResponseSerializer serializer];

    self.securityPolicy = [AFSecurityPolicy defaultPolicy];

#if !TARGET_OS_WATCH
    self.reachabilityManager = [AFNetworkReachabilityManager sharedManager];
#endif

    self.mutableTaskDelegatesKeyedByTaskIdentifier = [[NSMutableDictionary alloc] init];

    self.lock = [[NSLock alloc] init];
    self.lock.name = AFURLSessionManagerLockName;

    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        for (NSURLSessionDataTask *task in dataTasks) {
            [self addDelegateForDataTask:task uploadProgress:nil downloadProgress:nil completionHandler:nil];
        }

        for (NSURLSessionUploadTask *uploadTask in uploadTasks) {
            [self addDelegateForUploadTask:uploadTask progress:nil completionHandler:nil];
        }

        for (NSURLSessionDownloadTask *downloadTask in downloadTasks) {
            [self addDelegateForDownloadTask:downloadTask progress:nil destination:nil completionHandler:nil];
        }
    }];

    return self;
}
```







## Serialization

### AFURLRequestSerialization

### AFURLResponseSerialization



## Security



## Reachability



## UIKit
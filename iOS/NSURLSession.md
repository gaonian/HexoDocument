NSURLSessionDelegate

```objective-c
@optional
  
/* 会话接收到的最后一条消息。会话只有在系统错误或显式无效时才会失效，在这种情况下，error参数将为零。
 */

- (void)URLSession:(NSURLSession *)session didBecomeInvalidWithError:(nullable NSError *)error;



/* 如果实现了此delegate，则当发生连接级身份验证质询时，此delegate将有机会向基础连接提供身份验证证书。某些类型的身份验证将应用于与服务器的给定连接上的多个请求（SSL服务器信任询问）。如果未实现此delegate，则行为将使用默认处理，这可能涉及用户交互。
 */
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
                                             completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * _Nullable credential))completionHandler;
```



NSURLSessionDataDelegate <NSURLSessionTaskDelegate>

``` objective-c
@optional
  

/* 任务已经收到响应，并且在调用completion块之前不会再收到任何消息。允许您取消请求或将数据任务转换为下载任务。此委托消息是可选的——如果没有实现，可以从task的属性中获取response

此方法不会被后台上传任务调用(不能转换为下载任务)。
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
                                 didReceiveResponse:(NSURLResponse *)response
                                  completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler;


/* 通知data task已成为download task。将来不会向data task发送任何消息。
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
                              didBecomeDownloadTask:(NSURLSessionDownloadTask *)downloadTask;


/* 当数据可供委托使用时发送。假定委托将保留数据而不复制数据。由于数据可能是不连续的，你应该使用[NSData enumerateByteRangesUsingBlock:]来访问它。v
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
                                     didReceiveData:(NSData *)data;



/* 使用一个有效的NSCachedURLResponse调用completion例程来允许结果数据被缓存，或者传递nil来防止缓存。请注意，不能保证会对给定的资源尝试缓存，并且您不应该依赖此消息来接收资源数据。
 */
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
                                  willCacheResponse:(NSCachedURLResponse *)proposedResponse 
                                  completionHandler:(void (^)(NSCachedURLResponse * _Nullable cachedResponse))completionHandler;
```





NSURLSessionTaskDelegate <NSURLSessionDelegate>

``` objective-c
@optional
  
/* HTTP请求试图执行到另一个URL的重定向。您必须调用完成例程来允许重定向，允许使用修改后的请求进行重定向，或者将nil传递给completionHandler，以导致重定向响应的主体作为此请求的有效负载传递。默认情况下是遵循重定向。

对于后台会话中的任务，将始终遵循重定向，并且不会调用此方法。
 */

- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                     willPerformHTTPRedirection:(NSHTTPURLResponse *)response
                                     newRequest:(NSURLRequest *)request
                              completionHandler:(void (^)(NSURLRequest * _Nullable))completionHandler;



/* 任务收到请求特定的身份验证质询。
如果这个委托没有实现，特定于会话的身份验证质询将不被调用，行为将与使用默认处理配置相同。
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                            didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge 
                              completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * _Nullable credential))completionHandler;


/* 如果一个任务需要一个新的、未打开的body流时发送。当对任何涉及body流的请求进行身份验证失败时，这可能是必要的。
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                              needNewBodyStream:(void (^)(NSInputStream * _Nullable bodyStream))completionHandler;


/* 定期发送，以通知委托上传进度。此信息也可以作为任务的属性使用。
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                                didSendBodyData:(int64_t)bytesSent
                                 totalBytesSent:(int64_t)totalBytesSent
                       totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend;


/*
 * 当为任务收集完整的统计信息时发送。
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didFinishCollectingMetrics:(NSURLSessionTaskMetrics *)metrics API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));


/* task完成之后的回调，成功和失败都会回调。
 注意这里的error不会报告服务期端的error，他表示的是客户端这边的eroor，比如无法解析hostname或者连不上host主机。
 */
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
                           didCompleteWithError:(nullable NSError *)error;
```



NSURLSessionDownloadDelegate <NSURLSessionTaskDelegate>

``` objective-c
/* 当完成下载任务时调用。
委托应该复制或移动给定位置的文件到一个新位置，因为当委托消息返回时它将被删除。URLSession:task:didCompleteWithError:仍将被调用。
 */
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                              didFinishDownloadingToURL:(NSURL *)location;


@optional

/* 定期发送，以通知委托下载进度。
bytesWritten 表示自上次调用该方法后，接收到的数据字节数

totalBytesWritten表示目前已经接收到的数据字节数

totalBytesExpectedToWrite 表示期望收到的文件总字节数，是由Content-Length header提供。如果没有提供，默认是NSURLSessionTransferSizeUnknown。

*/
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                                           didWriteData:(int64_t)bytesWritten
                                      totalBytesWritten:(int64_t)totalBytesWritten
                              totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite;


/* 当一个下载被恢复时发送。如果下载失败，错误的-userInfo字典将包含一个NSURLSessionDownloadTaskResumeData键，它的值是resume数据。
 */
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask
                                      didResumeAtOffset:(int64_t)fileOffset
                                     expectedTotalBytes:(int64_t)expectedTotalBytes;

```



## 参考资料

[NSURLSession最全攻略](https://www.jianshu.com/p/ac79db251cbf)


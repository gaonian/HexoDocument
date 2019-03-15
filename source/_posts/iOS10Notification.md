---
title : iOS 10 Notification Extension
time : 2017-6-13
tag : 
---
## 简介
iOS10对推送通知增加了一些功能，用户可以对推送的内容增加附件（图片／音频／视频），也可以自定义视图更美观的展示推送内容。
- UNNotificationServiceExtension （通知服务扩展）
- UNNotificationContentExtension （通知内容扩展）
ps：简单来说，UNNotificationServiceExtension是用来下载附件的扩展，UNNotificationContentExtension用来自定义视图的扩展

## 1. UNNotificationAttachment （通知附件）
A UNNotificationAttachment object contains audio, image, or video content to display alongside the notification content. Your app always supplies attachments. For local notifications, the app adds attachments when creating the rest of the notification’s content. You can add attachments to a remote notification by implementing a notification service extension, as represented by the UNNotificationServiceExtension class.
用来管理通知附件，格式支持图片，音频，视频。格式的大小也有限制

![image](http://upload-images.jianshu.io/upload_images/1868661-9bafd854693406d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 本地通知
    直接把要展示的附件添加到content里面即可
```
    //设置content
    UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
    content.title = @"这是title";
    content.subtitle = @"这是subtitle";
    content.body = @"这是body";
    //设置附件
    UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"atta1" URL:*这里为本地文件地址* options:nil error:nil];
    if (attachment) {
        //添加附件到content
        content.attachments = @[attachment];
    }
    //添加推送到request
    UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:@"request_id1" content:content trigger:nil];
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
        NSLog(@"添加推送 : %@", error ? [NSString stringWithFormat:@"error : %@", error] : @"success");
    }];
```
- 远程推送
  远程推送需要实现UNNotificationServiceExtension这个扩展，在扩展里面监听通知内容，拦截通知内的附件url，进行下载，然后在添加到attachment内，回调，通知展示。一会会专门讲这个。（ ps：平常项目开发中肯定会有远程推送，所以必须是要实现UNNotificationServiceExtension这个扩展才能展示附件。）

## 2. UNNotificationServiceExtension （通知服务扩展）
A UNNotificationServiceExtension object, the principle class for a Notification Service app extension, lets you process the payload of a remote (sometimes called push) notification before it is delivered to the user.
![image](http://upload-images.jianshu.io/upload_images/1868661-21c02300372eabb1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原来的逻辑是服务器发送apns给苹果，苹果直接把内容推给用户展示。现在增加了ServiceExtension这一步骤，等于是苹果推给用户过程中被拦截了，可以对推送内容增加一些扩展，例如附件的下载等，然后处理完毕之后再展示到用户手机。
需要注意的一点就是在处理推送内容的时候，只有30s的时间去下载一些多媒体信息。如果因为网络状态或者一些其他的原因，30s内没有处理完毕，则推送会按原来的形式直接展示处理。就是说，本来你准备去下载图片然后展示，如果超时，则展示出来的是没有图片的。

具体实现步骤如下：
- 确定服务器推送格式
```
    {
        "aps" : {
            "alert": {
                "title": "这是title",
                "body": "这是body"       //推送主体内容
            },
            "badge": 1,
            "sound": "default",
            "mutable-content": "1",        //可变内容
            "imageUrl":  "http://***.jpg"  //图片url
        }
    }
```
需要注意的是，如果想改变推送内容，必须要在aps内增加`"mutable-content": "1"`字段，如果没有此字段，则直接显示推送内容，不会处理扩展里的实现。
`imageUrl`是附件的链接，这里以图片为例。此字段可以写在aps内外都可以。

- 创建UNNotificationServiceExtension扩展
  1.File -> New -> Target
  2.
![](http://upload-images.jianshu.io/upload_images/1868661-f605d7657dcfe63b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  3.填上名字，finish即可

![](http://upload-images.jianshu.io/upload_images/1868661-2459e746f2aece21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功之后会自动增加这么多东西，而且在`NotificationService.m`里重写了下面两个方法
```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    // Modify the notification content here...
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];
    
    self.contentHandler(self.bestAttemptContent);
}

- (void)serviceExtensionTimeWillExpire {
    // Called just before the extension will be terminated by the system.
    // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
    self.contentHandler(self.bestAttemptContent);
}
```
之后就是主要重写这两个方法来实现扩展，等下会专门讲解这方法。

- 为UNNotificationServiceExtension添加证书
创建成功扩展之后会看到它的bundle id是主程序的bundle id + .扩展名(例如：com.123.456.Notification)。在苹果证书平台新建appID，并添加对应的描述文件，安装即可。

- 实现扩展方法
收到推送的时候会自动调用
```
 - (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent *contentToDeliver))contentHandler {
    //在这个方法里面，重写通知内容，在此下载附件并展示
    // Modify the notification content here...
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@", self.bestAttemptContent.title];

    //取出推送的数据
    NSDictionary *dic = self.bestAttemptContent.userInfo;
    //获取要显示的图片url
    NSString *imgUrl = dic[@"aps"][@"imageUrl"];
    NSURL *url = [NSURL URLWithString:imgUrl];
    //取出后缀名
    NSString *ext = [imgUrl pathExtension];
    //下载图片并存储到本地沙盒
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (error) {
            NSLog(@"download - %@",error);
        } else {
            //沙盒创建download文件夹，存储附件信息
            NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"download"];
            NSFileManager *manager = [NSFileManager defaultManager];
            if (![manager fileExistsAtPath:path]) {
                [manager createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:nil];
            }
            //移动下载的路径到本地创建的路径
            NSURL *localUrl = [NSURL fileURLWithPath:[path stringByAppendingPathExtension:ext]];
            NSError *moveError = nil;
            [manager moveItemAtURL:location toURL:localUrl error:&moveError];
            if (moveError) {
                NSLog(@"move - %@",moveError);
            }
            //给通知的内容设置附件内容
            UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"remote-attachment1" URL:localUrl options:nil error:nil];
            if (attachment) {
                //如果有附件，传递给通知
                self.bestAttemptContent.attachments = @[attachment];
            }
        }
        self.contentHandler(self.bestAttemptContent);
    }];
    
    [task resume];
}
```
如果处理时间太长，超过30s，则会调用以下的方法，把通知内容直接回调出去，不做处理。
```
//如果处理时间太长，直接把apns展示出来
 - (void)serviceExtensionTimeWillExpire {
    // Called just before the extension will be terminated by the system.
    // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
    self.contentHandler(self.bestAttemptContent);
}
```
至此，UNNotificationServiceExtension这个扩展就基本实现完毕了，可以按照第一步的推送格式推一个试试。有3DTouch的设备，重按推送内容 可以看到一个大图直接在最上面，下面是推送的内容。
附图：
![
![](http://upload-images.jianshu.io/upload_images/1868661-0ac358ca44c1d63d.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
](http://upload-images.jianshu.io/upload_images/1868661-cc3d9a1f4df0771a.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3. UNNotificationContentExtension （通知内容扩展）
The UNNotificationContentExtension protocol lets you present a custom interface for your app’s notifications. You adopt this protocol in custom UIViewController subclasses, using the view controller’s view to display the notification contents. You deliver your view controller class inside a Notification Content extension.
iOS10推送提供了一块区域让用户自定义view去更好的展示推送内容，信息量更大。想要自定义view则必须要实现`UNNotificationContentExtension`这个扩展，但是貌似只有支持3DTouch的设备通过重压或者下拉才能展示出来自定义的view。注意，这个自定义的view只支持展示功能，不能交互。

具体实现步骤如下：
- 创建UNNotificationContentExtension扩展
  1.File -> New -> Target
  2.
![](http://upload-images.jianshu.io/upload_images/1868661-c8f21b20cfc0bca2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  3.填上名字，finish即可

![](http://upload-images.jianshu.io/upload_images/1868661-e013b44d5f284692.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
和通知服务扩展一样，系统会生成一些文件，这个需要注意的是plist里有很多可以配置的内容。
```
- (void)didReceiveNotification:(UNNotification *)notification {
    //在这个方面里给自定义的控件赋值
    NSDictionary *dic = notification.request.content.userInfo;
    self.lbTitle.text = @"哈哈哈";
    
    self.imgLogo.backgroundColor = [UIColor redColor];
}
```

- 为UNNotificationContentExtension添加证书
创建成功扩展之后会看到它的bundle id是主程序的bundle id + .扩展名(例如：com.123.456.NotificationContent)。在苹果证书平台新建appID，并添加对应的描述文件，安装即可。
和UNNotificationServiceExtension扩展一样，都需要新创建证书


- 实现扩展方法
收到推送的时候会自动调用
```
 - (void)didReceiveNotification:(UNNotification *)notification {
    //在这个方面里给自定义的控件赋值
    NSDictionary *dic = notification.request.content.userInfo;
    self.lbTitle.text = @"哈哈哈";

    self.imgLogo.backgroundColor = [UIColor redColor];
}
```
拿到推送的内容，对自定义的控件赋值展示。
至此，通知内容扩展创建完毕，下面来讲一下plist，以及需要注意的事项

![通知内容扩展plist](http://upload-images.jianshu.io/upload_images/1868661-45810cf9227078d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图是创建扩展的时候自带的plist文件，`UNNotificationExtensionDefaultContentHidden`这个属性是自己添加的，意思就是隐藏系统自己的推送内容。当自定义的内容跟系统的重合了，可以用这个属性把系统的隐藏了，只展示自定义的。
`UNNotificationExtensionInitialContentSizeRatio`这个属性是自定义view的比例，默认1是宽高比例相同。可以根据自定义view的尺寸在这里改变比例。
`UNNotificationExtensionCategory`这个categoryID很重要，想要展示自定义的内容扩展，必须要保证aps里面的category和plist配置的一样。
```
    {
        "aps" : {
            "alert": {
                "title": "这是title",
                "body": "这是body"       //推送主体内容
            },
            "badge": 1,
            "sound": "default",
            "mutable-content": "1",        //可变内容
            "category": "myNotificationCategory",    //categoryId
            "imageUrl":  "http://***.jpg"  //图片url
        }
    }
```
根据这个id来展示不同的自定义内容。`UNNotificationExtensionCategory`也可以是一个数组，多个catogoryId对应一个自定义的view。
![](http://upload-images.jianshu.io/upload_images/1868661-724cc8d03e7db59b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. 结合使用两个扩展
在自定义view的时候，可以获取到推送内容以及附件信息，放到自定义的view中。简单的运用如下
```
- (void)didReceiveNotification:(UNNotification *)notification {
    
    UNNotificationContent *content = notification.request.content;
    //获取title
    self.lbTitle.text = content.title;
    
    //获取到附件信息
    UNNotificationAttachment *attachment = content.attachments.firstObject;
    if (attachment) {
        //告诉系统，我们要使用附件信息
        if ([attachment.URL startAccessingSecurityScopedResource]) {
            self.imgLogo.image = [UIImage imageWithContentsOfFile:attachment.URL.path];
            //使用完毕之后，告诉系统，使用完毕，可以关闭了
            [attachment.URL stopAccessingSecurityScopedResource];
        }
    }
}
```

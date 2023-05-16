---
title: my-ios-day19
date: 2016-09-01 16:12:15
categories: 
- technology
- Objective-C 
tags: 
- ios
- gitbook
---

这个是16年重新复习的iOS笔记，当时是放在了[gitbook](https://sleenxiu.gitbooks.io/my-learning-ios/content/)，现在转移过来！
<!-- more -->

# Day 19
## 网络
- 相关概念
- NSURLConnection
- 数据解析
- NSURLConnection文件传输
- NSURLSession
- AFNetworking
---
#### 相关概念
- 基础
- 客户端，手机所使用的app
- 服务器，远程服务器(Linux系列，windows)
- 请求，客户端向服务器索取数据的方式
- 响应，服务器返回、客户端需要解析的数据
- 统一资源定位符URL
- 定位主机和资源
- 格式（协议/主机地址/资源路径）
- 协议：资源的查找方式和传输方式（HTTP、FTP...）
- 主机地址：存放资源的服务器主机的IP地址、域名
- 路径：资源在服务器主机中的位置
- 常见协议
- file：本地计算机的资源，file://
- ftp：共享主机的资源，ftp://
- mailto：电子邮件地址，mailto:
- http：超文本传输协议，访问远程的网络资源，http://
- http协议
- 简介
- 超文本传输协议
- 规定客户端和服务器之间数据传输格式
- 让客户端和服务器有效的进行数据沟通
- 请求通信过程
- 请求
请求头 + 请求体(非必须)
- 响应
响应头 + 响应体
- 通信过程
客户端发送请求时，先把请求头和请求体（非必须）封装成一个请求对象
服务器端对请求进行响应，在响应信息中包含响应头和响应体，响应信息是对服务器端的描述，具体的信息放在响应体中传递给客户端
- 常见状态码
200：请求成功
400：请求语法错误，服务器无法解析
404：资源路径不存在
500：服务器内部错误
- 请求
- GET请求
http://www.test.com/login?username=123&pwd=234
请求的参数跟在URL的后面，以?隔开，使用&连接
服务器对URL的长度有限制，所以对参数长度有限制，通常不超过1KB
- POST请求
http://www.test.com/login
请求的参数放在请求体里面，POST的数据量一般无限制，主要看服务器
- 如何选择
如果有大量数据传输，上传、下载等，只能使用POST
GET中的参数过于暴露，设计敏感、机密需要用POST
增加、修改、删除数据，建议使用POST
只有在简单的查询数据，使用GET
- iOS中的请求
- 系统原生
- NSURLConnection 03年的古老技术
- NSURLSession 13年iOS7之后推出，替代NSURLConnection
- CFNetwork 底层所依赖的技术，纯C
- 第三方框架
- ASIHttpRequest 已不再维护
- AFNetworking 主流
---
#### NSURLConnection
- 同步请求GET-sync
```objc
- (void)syncG {
// 确定URL
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];
// 封装请求
NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url];
// 声明响应
NSURLResponse *reponse = nil;
// 发送请求，接收数据
NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&reponse error:nil];
// 简单打印
NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
}
```
- 异步请求GET-async
```objc
- (void)asyncG {
// 确定URL
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=JSON"];
// 封装请求
NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url];
// 发送请求
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
// 简单打印数据
NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
// 响应头信息
NSHTTPURLResponse *re = (NSHTTPURLResponse *)response;
NSLog(@"%ld", re.statusCode);
}];
}
```
- 异步请求GET-delegate
```objc
- (void)delegateG {
// 确定URL
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/video?type=JSON"];
// 封装请求
NSURLRequest *request = [[NSURLRequest alloc] initWithURL:url];
// 设置代理
// [NSURLConnection connectionWithRequest:request delegate:self];
// [[NSURLConnection alloc] initWithRequest:request delegate:self];
NSURLConnection *con = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
// 可以特定时间开始发送请求
[con start];
[con cancel];
}

#pragma mark - NSURLConnectionDataDelegate
/**
* 接收服务器响应时调用
*/
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
NSLog(@"%s", __func__);
}
/**
* 接收到服务器返回数据时调用
* 可能会带哦与功能多次
*/
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
NSLog(@"%s", __func__);
[self.allData appendData:data];
}
/**
* 请求失败会调用
*/
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
NSLog(@"%s", __func__);
}
/**
* 请求结束时会调用
*/
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
NSLog(@"%s", __func__);
NSLog(@"%@", [[NSString alloc] initWithData:self.allData encoding:NSUTF8StringEncoding]);
}
```
- POST请求
- 步骤
1. 确定URL
2. 创建可变的请求对象
3. 修改请求对象的方法为POST
4. 设置请求体
5. 发送异步请求
6. 设置超时，处理错误信息，设置请求头（可选）
- 代码
```objc
- (void)asyncP {
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url];
// 修改请求类型，POST必须大写
request.HTTPMethod = @"POST";
// 设置请求体信息
NSData *bodyData = [@"username=52it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
request.HTTPBody = bodyData;
// 发送请求
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
}];
}
```
---
#### 数据解析
- 常用数据格式
- JSON：轻量级，比较流行
- XML：可拓展标记语言，包含声明、元素和属性
- JSON解析
- NSJSONSerialization
- JSON-->OC （反序列化）
```objc
- (void)JOSNToOC {
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=123&pwd=123&type=JSON"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
/*
第一个参数：要解析的JSON数据，NSData，二进制数据
第二个参数：解析JSON的配置参数（可选）
NSJSONReadingMutableContainers 解析出来的字典和数组是可变的
NSJSONReadingMutableLeaves 解析出来的对象中的字符串是可变的 iOS7以后有问题，不用
NSJSONReadingAllowFragments 被解析的JSON数据如果既不是字典也不是数组， 那么就必须使用这个
kNilOptions == 0 哪个也不选，高效
*/
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil];
NSLog(@"%@",dict);
}];
}
```
- OC-->JSON （序列化）
```objc
/*
OC-->JSON有一定限制
使用+ (BOOL)isValidJSONObject:(id)obj;方法判断当前OC对象能否转换为JSON数据
1. obj是NSArray或NSDictionary以及他们的子类
2. obj包含的所有对象是NSString、NSNumber、NSArray、NSDictionary、NSNull
3. 字典中的所有key必须是NSString
4. NSNumber的对象不能是Nan或无穷大
*/
// 需要序列化的数据
NSDictionary *dictM = @{
@"name":@"sleen",
@"age":@18,
@"height":@1.80
};
/*
第一个参数：需要序列化的数据
第二个参数：NSJSONWritingPrettyPrinted对转换之后的JSON对象进行排版，无意义
*/
NSData *data = [NSJSONSerialization dataWithJSONObject:dictM options:NSJSONWritingPrettyPrinted error:nil];
// 查看是否序列化
NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
NSLog(@"%@",str);
```
- JSON数据与OC对象对应
{} --> NADictionary
[] --> NSArray
"" --> NSString
10/10.1 --> NSNumber
true/false --> NSNumber
null --> NSNull
- XML解析
- 两种方式
1. SAX：从根节点开始，按顺序一个元素一个元素的向下解析
2. DOM：一次性将整个xml文档加载到内存中，使用与较小文件
- 工具
1. NSXMLParser：苹果原生，采用SAX，使用简单
2. libxml2：纯C，默认在iOS SDK中，同时支持SAX和DOM
3. GDataXML：Google的基于libxml2，采用DOM方式
- NSXMLParser
```objc
// 创建一个解析器
NSXMLParser *parser = [[NSXMLParser alloc]initWithData:data];
// 设置代理
parser.delegate = self;
// 开始解析
[parser parse];
// 开始解析XML文档
- (void)parserDidStartDocument:(nonnull NSXMLParser *)parser {
}
// 开始解析XML中某个元素的时候调用，比如<video>
- (void)parser:(nonnull NSXMLParser *)parser didStartElement:(nonnull NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName attributes:(nonnull NSDictionary<NSString *,NSString *> *)attributeDict
{
if ([elementName isEqualToString:@"videos"]) {
return;
}
// 字典转模型
SLVideo *video = [SLVideo objectWithKeyValues:attributeDict];
[self.videos addObject:video];
}
// 当某个元素解析完成之后调用，比如</video>
- (void)parser:(nonnull NSXMLParser *)parser didEndElement:(nonnull NSString *)elementName namespaceURI:(nullable NSString *)namespaceURI qualifiedName:(nullable NSString *)qName {
}
// XML文档解析结束
- (void)parserDidEndDocument:(nonnull NSXMLParser *)parser {
}
```
---
#### 文件传输
- 小文件下载
- 直接下载，无多线程
```objc
// 确定资源路径
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"];
// 根据URL加载对应的资源
NSData *data = [NSData dataWithContentsOfURL:url];
// 转换并显示数据
UIImage *image = [UIImage imageWithData:data];
self.imageView.image = image;
```
- NSURLConnection - sendAsync
```objc
// 确定请求路径
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"];
// 创建请求对象
NSURLRequest *request = [NSURLRequest requestWithURL:url];
// 使用NSURLConnection发送一个异步请求
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
// 设置数据
UIImage *image = [UIImage imageWithData:data];
self.imageView.image = image;
}];
```
- NSURLConnection - delegate
```objc
- (void)delegateDownload {
// 确定请求路径
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];
//2.创建请求对象
NSURLRequest *request = [NSURLRequest requestWithURL:url];
//3.使用NSURLConnection设置代理并发送异步请求
[NSURLConnection connectionWithRequest:request delegate:self];
}
#pragma mark - NSURLConnectionDataDelegate
// 当接收到服务器响应的时候调用，该方法只会调用一次
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
// 创建一个容器，用来接收服务器返回的数据
self.fileData = [NSMutableData data];

// 获得当前要下载文件的总大小（通过响应头得到）
NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;
self.totalLength = res.expectedContentLength;
// 拿到服务器端推荐的文件名称
self.fileName = res.suggestedFilename;
}
// 当接收到服务器返回的数据时会调用
// 该方法可能会被调用多次
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
// 拼接每次下载的数据
[self.fileData appendData:data];
// 计算当前下载进度并刷新UI显示
self.currentLength = self.fileData.length;
NSLog(@"%f",1.0* self.currentLength/self.totalLength);
self.progressView.progress = 1.0* self.currentLength/self.totalLength;
}
//当网络请求结束之后调用
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
// 文件下载完毕把接受到的文件数据写入到沙盒中保存
// 确定要保存文件的全路径
// caches文件夹路径
NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
NSString *fullPath = [caches stringByAppendingPathComponent:self.fileName];
// 写数据到文件中
[self.fileData writeToFile:fullPath atomically:YES];
}
// 当请求失败的时候调用该方法
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
NSLog(@"%s",__func__);
}
```
- 大文件下载
- 文件句柄下载
```objc
#pragma mark - NSURLConnectionDataDelegate
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
// 当已经下载过，就不在创建文件
if (self.currentSize > 0) {
return ;
}
// 获取文件总大小
NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;
self.totalSize = res.expectedContentLength;
// 创建一个新的文件
// 得到文件管理者
NSFileManager *man = [NSFileManager defaultManager];
// 搜索cache路径
NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
// 拼接全路径
NSString *path = [caches stringByAppendingPathComponent:res.suggestedFilename];
self.fullPath = path;
// 创建文件
/*
* 第一个参数：文件路径
* 第二个参数：文件内容
* 第三个参数：文件属性
*/
[man createFileAtPath:path contents:nil attributes:nil];
}
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
// 文件句柄
self.handle = [NSFileHandle fileHandleForWritingAtPath:self.fullPath];
// 移动句柄
[self.handle seekToEndOfFile];
// 追加数据
[self.handle writeData:data];
// 得到已下载大小
self.currentSize += data.length;
self.progressV.progress = 1.0 * self.currentSize / self.totalSize;
self.progressL.text = [NSString stringWithFormat:@"%.2f", self.progressV.progress];
NSLog(@"%f", self.progressV.progress);
}
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
}
- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
[self.handle closeFile];
self.handle = nil;
NSLog(@"%s", __func__);
NSLog(@"%@", self.fullPath);
}

```
- 输出流下载
```objc
// 创建一个数据输出流
/*
* 第一个参数：二进制的流数据要写入到哪里
* 第二个参数：采用什么样的方式写入流数据，如果YES则表示追加，如果是NO则表示覆盖
*/
NSOutputStream *stream = [NSOutputStream outputStreamToFileAtPath:fullPath append:YES];
// 只要调用了该方法就会往文件中写数据
// 如果文件不存在，那么会自动的创建一个
[stream open];
self.stream = stream;
// 当接收到数据的时候写数据
// 使用输出流写数据
/*
第一个参数：要写入的二进制数据
第二个参数：要写入的数据的大小
*/
[self.stream write:data.bytes maxLength:data.length];
// 当文件下载完毕的时候关闭输出流
// 关闭输出流
[self.stream close];
self.stream = nil;
```
- 文件上传
- 步骤
1. 确定请求的URL
2. 创建可变的请求对象
3. 修改请求方式为POST
4. 设置请求头，通知服务器将要上传文件Content-Type
5. 设置请求体（严格按确定格式拼接数据）
6. 发送异步请求上传文件
7. 解析服务器返回数据
- 请求体数据格式
```
分隔符：----WebKitFormBoundaryhBDKBUvFdCAgsr5b
格式：
--分隔符
Content-Dispostion:参数
Content-Type:参数
空行
文件参数
--分隔符
Content-Disposition:参数
空行
非文件的二进制数据
--分隔符--
```
- 代码
```objc
#define Kboundary @“----WebKitFormBoundaryhBDKBUvFdCAgsr5b”
- (void)upload
{
// 1.确定请求路径
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/upload"];

// 2.创建一个可变的请求对象
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

// 3.设置请求方式为POST
request.HTTPMethod = @"POST";

// 4.设置请求头
NSString *filed = [NSString stringWithFormat:@"multipart/form-data; boundary=%@",Kboundary];
[request setValue:filed forHTTPHeaderField:@"Content-Type"];

// 5.设置请求体
NSMutableData *data = [NSMutableData data];
// 5.1 文件参数
/*
--分隔符
Content-Disposition:参数
Content-Type:参数
空行
文件参数
*/
[data appendData:[[NSString stringWithFormat:@"--%@",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];
[data appendData:[@"Content-Disposition: form-data; name=\"file\"; filename=\"test.png\"" dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];
[data appendData:[@"Content-Type: image/png" dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];
[data appendData:KnewLine];
[data appendData:KnewLine];

UIImage *image = [UIImage imageNamed:@"test"];
NSData *imageData = UIImagePNGRepresentation(image);
[data appendData:imageData];
[data appendData:KnewLine];
// 5.2 非文件参数
/*
--分隔符
Content-Disposition:参数
空行
非文件参数的二进制数据
*/

[data appendData:[[NSString stringWithFormat:@"--%@",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];
[data appendData:[@"Content-Disposition: form-data; name=\"username\"" dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];
[data appendData:KnewLine];
[data appendData:KnewLine];

NSData *nameData = [@"wendingding" dataUsingEncoding:NSUTF8StringEncoding];
[data appendData:nameData];
[data appendData:KnewLine];

// 5.3 结尾标识
// --分隔符--
[data appendData:[[NSString stringWithFormat:@"--%@--",Kboundary] dataUsingEncoding:NSUTF8StringEncoding]];
[data appendData:KnewLine];

request.HTTPBody = data;

// 6.发送请求
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {

// 7.解析服务器返回的数据
NSLog(@"%@",[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]);
}];
}
```
- 获取文件的MIMEType类型
```objc
- (NSString *)getMIMEType
{
NSString *filePath = @"/Users/xiuxiu/Desktop/sleen.txt";
// 同步请求获取
NSURLResponse *response = nil;
[NSURLConnection sendSynchronousRequest:[NSURLRequest requestWithURL:[NSURL fileURLWithPath:filePath]] returningResponse:&response error:nil];
// 异步请求获取
[NSURLConnection sendAsynchronousRequest:[NSURLRequest requestWithURL:[NSURL fileURLWithPath:@"/Users/xiuxiu/Desktop/sleen.png"]] queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {
// response.MIMEType
NSLog(@"%@",response.MIMEType);
}];
return response.MIMEType;
}
```
---
#### NSURLSession
- 使用
- 通过NSURLSession创建task，执行task
- NSURLSessionTask
- 本身是一个抽象类，使用三个子类
- NSURLSessionDataTask
- NSURLSessionUploadTask
- NSURLSessionDownloadTask
- 发送请求
- GET请求
```objc
// block方式
- (void)get {
// 创建session，获得单例/自定义
NSURLSession *session = [NSURLSession sharedSession];
// 创建请求对象
NSURL *url = [NSURL URLWithString:@""];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
// 根据session创建task
NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
// 解析数据
}];
// GET请求可以直接使用url创建
// NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
// // 解析数据
// }];
// 执行task
[task resume];
}
```
- POST请求
```objc
- (void)post {
// 创建session，获得单例/自定义
NSURLSession *session = [NSURLSession sharedSession];
// 创建请求对象
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
// 设置请求
request.HTTPMethod = @"POST";
request.HTTPBody = [@"username=123&pwd=123&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
// 根据session创建task
NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
// 解析数据
}];
// 执行task
[task resume];
}
```
- 文件传输
- 文件下载
```objc
- (void)download {
// 代理方式需要使用自定义的session
NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
// 创建请求对象
NSURL *url = [NSURL URLWithString:@""];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
// 根据session创建task
NSURLSessionDataTask *task = [session dataTaskWithRequest:request];
// 执行task
[task resume];
}
#pragma mark - NSURLSessionDataDelegate
/**
* 收到服务器响应时调用
*
* @param session 发送请求的session对象
* @param dataTask 创建的task任务
* @param response 响应信息（响应头）
* @param completionHandler 回调，是否要接收数据
*/
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
// 默认情况下，当接收到服务器响应之后，服务器认为客户端不需要接收数据，所以后面的代理方法不会调用
// 如果需要继续接收服务器返回的数据，那么需要调用block，并传入对应的策略
/*
NSURLSessionResponseCancel = 0, 取消任务
NSURLSessionResponseAllow = 1, 接收任务
NSURLSessionResponseBecomeDownload = 2, 转变成下载
NSURLSessionResponseBecomeStream NS_ENUM_AVAILABLE(10_11, 9_0) = 3, 转变成流
*/
completionHandler(NSURLSessionResponseAllow);
}
/**
* 收到服务器返回数据是调用
* 方法会被调用多次
*/
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
}
/**
* 请求完成后会调用
* 成功和失败都会调用，失败后error会保存失败信息
*/
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
}
```
- 大文件下载（离线缓存）
```objc
#pragma mark - lazy
- (NSString *)fullPath {
if (_fullPath == nil) {
NSString *path = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
_fullPath = [path stringByAppendingPathComponent:Filename];
}
return _fullPath;
}
- (NSURLSession *)session {
if (!_session) {
// 代理方式需要使用自定义的session
_session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
}
return _session;
}
- (NSURLSessionDataTask *)task {
if (!_task) {
// 创建请求对象
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
// 设置请求头
self.currentSize = [self oldFileSize];
NSString *range = [NSString stringWithFormat:@"bytes=%zd-", self.currentSize];
[request setValue:range forHTTPHeaderField:@"range"];
// 创建人物
_task = [self.session dataTaskWithRequest:request];
}
return _task;
}
#pragma mark - other
/**
* 获取本地文件大小
*/
- (NSInteger)oldFileSize {
NSDictionary *fileInfo = [[NSFileManager defaultManager] attributesOfItemAtPath:self.fullPath error:nil];
return [fileInfo[@"NSFileSize"] integerValue];
}
#pragma mark - NSURLSessionDataDelegate
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
self.totalSize = response.expectedContentLength + self.currentSize;
if (self.currentSize == 0) {
[[NSFileManager defaultManager] createFileAtPath:self.fullPath contents:nil attributes:nil];
}
self.handle = [NSFileHandle fileHandleForWritingAtPath:self.fullPath];
completionHandler(NSURLSessionResponseAllow);
}
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
[self.handle seekToEndOfFile];
[self.handle writeData:data];
self.currentSize += data.length;
self.progressV.progress = 1.0 * self.currentSize / self.totalSize;
self.progressL.text = [NSString stringWithFormat:@"%.2f", self.progressV.progress];
}
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
[self.handle closeFile];
self.handle = nil;
NSLog(@"didCompleteWithError%@", error);
NSLog(@"%@", self.fullPath);
}
```
- 文件上传
```objc
- (void)upload {
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/upload"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
request.HTTPMethod = @"POST";
[request setValue:[NSString stringWithFormat:@"multipart/form-data;boundary=%@", Kboundary] forHTTPHeaderField:@"Content-Type"];
NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
NSURLSessionUploadTask *task = [session uploadTaskWithRequest:request fromData:[self getBodyData] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
}];
[task resume];
}
- (NSData *)getBodyData {
// 拼接数据
}
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend {
NSLog(@"%f",1.0 *totalBytesSent / totalBytesExpectedToSend);
}
```
---
#### AFNetworking



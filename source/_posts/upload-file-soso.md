---
title: 图片上传的问题
date: 2019-05-15 19:48:51
tags: [iOS]
categories: [technology]
---

项目中上传图片的一个特例

<!-- more -->

- 后台要求的一个格式
```
idcards[0].image      // 图片文件
idcards[0].side       // 图片参数
```

- 开始理解是，数组里的两个字典，但这种无法转换为 data 设置给 http.body
```
idcards : [
    {
        image: data,
        side: front
    },
    {
        image: data,
        side: back
    }
]
```

- 通过postman 来测试接口，发现传的数据原来是这样，瞬间明白
![img](//img-uss.likehub.top/hexo/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-05-15%20%E4%B8%8B%E5%8D%888.02.48.png)

- 最终构造的请求长这样，应该就是把图片放在字典里上传(╰\_╯)

```
    [manager POST:url parameters:@{@"idCards[0].side":@"front",@"idCards[1].side":@"back"} constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        [formData appendPartWithFileData:UIImagePNGRepresentation(images[0]) name:@"idCards[0].image" fileName:@"front.png" mimeType:@"image/png"];
        [formData appendPartWithFileData:UIImagePNGRepresentation(images[1]) name:@"idCards[1].image" fileName:@"back.png" mimeType:@"image/png"];

    } progress:^(NSProgress * _Nonnull uploadProgress) {

    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        !success ? : success(responseObject);
        [self ic_apiLogWithUrl:url params:params error:nil];
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        !failure ? : failure([self myErrorWithNSError:error]);
        [self ic_apiLogWithUrl:url params:params error:error];
    }];
```

- 后台是用 java 写的，具体怎么解析还有待研究


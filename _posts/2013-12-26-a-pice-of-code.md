---
layout: post
title: "一小段代码的背后"
description: ""
category: "thoughts"
tags: [restful]
---

{% include JB/setup %}

上周四晚上，看到剑总背着自己的pair，望着一小坨代码纠结无比，我好奇的凑了上去，这便是这坨代码：

<!--more-->

```c#
private Guid CreateOrUpdateAccount(int oldUserId)
{
    var migrationProcess = apiClient.GetEnsureSuccess<MigrationProcessDto>(string.Format("migration-process/{0}", oldUserId));
    var userId = migrationProcess.UserId;
    if (!string.IsNullOrEmpty(userId))
    {
        var updateDto = repository.ExtendUpdateUserDto(oldUserId, apiClient.Address);
        var response = apiClient.PutAsJson(string.Format("users/migration-process/{0}", userId), updateDto);
        response.EnsureSuccessStatusCode("Reset User");
    }
    else
    {
        userId = CreateUser(oldUserId);
    }
    var accountId = new Guid(apiClient.GetEnsureSuccess<UserInfo>(String.Format("users/{0}", userId)).AccountId);;
    return accountId;
}

private string CreateUser(int oldUserId)
{
    var userDto = repository.FindById(oldUserId, apiClient.Address);
    var response = apiClient.PostAsJson("users", userDto);
    response.EnsureSuccessStatusCode("Create User");
    var userUri = response.Headers.Location;
    UpdateUserInformation(oldUserId, userUri);
    var last = userUri.AbsoluteUri.Split(new[] {'/'}, StringSplitOptions.RemoveEmptyEntries).Last();
    return last;
}
```
打眼儿一看，没看出什么问题，然后我便问剑总，肿么了？剑总指着`CreateOrUpdateAccount`这个函数中的`userId`说，“这个userId让我觉得好别扭，感觉在用它绕圈子。”

这段代码是为了把老系统中的用户迁移到新系统中去，`oldUserId`是用户在老系统中的id，`migrationProcess.UserId`是用户在新系统中的id。为了保持迁移过程的幂等性（调用两次产生同样的结果，也就是不会产生两个用户），我们建立了`MigrationProcess`这个对象，在其中记录了迁移的过程。如果由于什么因素，用户没有被创建出来，我们重试就能创建出来（发Post请求）；而如果上次user已经被创建出来了，这个过程就变成了更新（发Put请求）。在整个过程中，为了区分用户是否被创建出来，于是对这个对象中的`userId`进行了判空操作，然后接着又一次使用`userId`进行更新操作，最后调用API获得用户的accountId。

的确是这样的，用`userId`绕了半天的圈子，然而它只是为了得到`accountId`的一个中间变量，而且，在调用`CreateUser`的时候（Post），从Location上分离出userId的逻辑也显得十分诡异。

我的第一直觉是应该可以连接数据库直接访问MigrationProcess，而不是通过API，这要求把MigrationAPI的实现放在当前程序中。这样的话，代码可能会变成这个样子

```c#
private Guid CreateOrUpdateAccount(int oldUserId)
{
    var migrationProcess = migrationRepo.GetByGoId(oldUserId);
    var userAccountId = migrationProcess.UserAccountId;
    if (!string.IsNullOrEmpty(userAccountId))
    {          
        var updateDto = repository.ExtendUpdateUserDto(oldUserId, apiClient.Address);
        var response = apiClient.PutAsJson(string.Format("users/migration-process/{0}", userAccountId), updateDto);
        response.EnsureSuccessStatusCode("Reset User");
    }
    else
    {
        migrationProcess.UserAccountId = CreateUser(oldUserId);
        goMigrationRepo.Save(migrationProcess)
    }
    return userAccountId;    
}

private Guid CreateUser(int oldUserId)
{
    var userDto = repository.FindById(oldUserId, apiClient.Address);
    var response = apiClient.PostAsJson("users", userDto);
    response.EnsureSuccessStatusCode("Create User");
    var userAccountId = response.ReadAsAsync<UserDto>().AccountId;
    var userUri = response.Headers.Location;
    UpdateUserInformation(oldUserId, userUri);
    return userAccountId;
}
```
修改后的代码的确消除了对userId的使用，然而，这坨代码仍然让人觉得非常别扭，如果无法找到这段代码的坏味道，也就没办法使它变得更好。

那么，坏味道究竟是什么呢？究竟是什么让剑总和我感到如此的别扭呢？
我周末苦苦思索了半饷，突然间冒出来了个主意，可能很多人早就想到了，show代码先

```c#
private Guid CreateOrUpdateAccount(int oldUserId)
{
    var migrationProcess = apiClient.GetEnsureSuccess<MigrationProcessDto>(string.Format("migration-process/{0}", oldUserId));
    var userLink = migrationProcess.Links.FirstOrDefault(l => l.Rel == "user");

    var response = userLink.Methods.Contains("Put") ? 
        ResetUser(oldUserId, userLink.Url) : CreateUser(oldUserId, userLink.Url);
    return response.GetAsAsync<UserInfo>().Result.AccountId;
}

private HttpResponseMessage ResetUser(int oldUserId, string userUrl)
{
    var updateDto = repository.ExtendUpdateUserDto(oldUserId, apiClient.Address);
    var response = apiClient.PutAsJsonByLink(userUrl, updateDto);
    response.EnsureSuccessStatusCode("Reset User");
    return response;
}

private HttpResponseMessage CreateUser(int oldUserId, string assingeeUrl)
{
    var userDto = repository.FindById(oldUserId, apiClient.Address);
    var response = apiClient.PostAsJson(assingeeUrl, userDto);
    response.EnsureSuccessStatusCode("Create User");
    var userUri = response.Headers.Location;

    var request = repository.GetUserForUpdateDto(oldUserId, apiClient.Address);
    var responseMessage = apiClient.PutAsJsonByLink(userUri.ToString(), request);
    responseMessage.EnsureSuccessStatusCode("Update User Information");
    return responseMessage;
}
```
再和第一个版本比较一下，这里唯一做出的修改是把`userId`变换成了`userLink`，剩下的都是水到渠成的了，这才是**HATEOAS (hypermedia as the engine of application state)**嘛。

在Rest的世界里，如果你在使用url template，那么，这就是坏味道，如果你在解析url，那么这也是坏味道。这些个坏味道说明了server端没有提供足够的Hypermedia Links（或者提供了你不知道），使得整个业务过程连不起来，只能依赖于out of blue的url template。

看上去像是一个`userId`的问题，然而跳出这段代码本身，却可以看到却是提供端的API设计不完善，在这背后折
射出我对Rest的理解还不够深刻。

-------------------------------------------------------------------------------

##后记

跟剑总就这个问题交流之后，他的回复更发人深省，也摘录在这里



>自己这两天也在反思为啥现在看是如此自然而明显的解决方案，而当时自己却仅仅是闻到了有味道，但还是无法很自然地想到或识别出问题，只能在那揪着头发背着pair黯然纠结？后来自己找到的答案很简单，就是动手动的少……话说REST那本书也看过几遍了，平时的讨论中也在跟人侃着REST如何如何，什么是Resource什么是Service，什么是Follow Link，引经据典用着买咖啡的例子给别人讲着什么才是真正的REST，什么是Post什么是Put什么又叫做Path……但是以为自己懂了其实还是没懂，因为不会用

>-

>很自然的联想到了光磊的一篇博客[《知行合一》](http://liguanglei.name/blogs/2012/05/19/on-knowledge-interface/)  ，记得读《明朝那些事儿》的时候看到王守仁悟到了知行合一后就像七龙珠中的孙悟空领悟到了如何变成超级赛亚人一样所向披靡，很是不解，觉得这只是一个很简单的道理，不就是说要学以致用么，为什么会有如此大的威力，但是现在越来越觉得做到这点真的很难，从接收到信息的那一刻开始到最终融会贯通化于无形最终转化为自己的智慧的一部分是一个艰难卓绝的蜕变的过程，而这其中很重要的一点就是“行动”或是“运用”。

>-

  >>*输入只是娱乐, 输出才是学习. (不那么极端的话, 输入是学, 输出是习). 听课是娱乐, 做作业才是学习. 看书是娱乐, 写东西才是学习. 听人聊天是娱乐, 问问题回答问题才是学习*  --@liguanglei


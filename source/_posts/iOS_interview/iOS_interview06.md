---
title: iOS_interview06 | 网络协议 和 网络优化
p: iOS_interview/iOS_interview06.md
date: 2020-03-31 18:42:56
tags:
---

## 网络分层

物理层：物理层是建立在物理通信介质的基础上， 作为系统和通信介质的接口，用来实现数据链路实体间透明的bit流传输。只有物理层是真实物理通信，其他各层都是虚拟通信
数据链路层：在物理层提供比特流服务的基础上，建立相邻结点之间的数据链路，通过差错控制提供数据帧（Frame）在信道上无差错的传输，并进行各电路上的动作系列。数据的单位称为帧（frame）
网络层： 选择合适的路由，使数据分组packet可以交付到目的的主机
传输层：负责主机间进程间通信（ipc）从网卡将数据传给监听端口的进程
应用层： 直接为用户的应用提供服务

http/https DNS SMTP RMTP FTP

TCP/UDP

IP

传输层是在进程间传输报文，TCP和UDP

#### TCP和UDP的区别

TCP: 面向连接、可靠的流协议
* 连接是指两个应用程序为了相互传输消息而建立专有的、虚拟的通信线路，也叫做虚拟电路。流是指不间断的数据结构，和水流一样的概念
* 可靠是指TCP提供可靠性传输, 实行顺序控制和重发控制机制，还提供流量控制，拥堵控制和提高网络利用率


UDP: 不具备可靠性的数据报协议
* 只确保发送，其他处理由上层应用来处理

Socket：Linux/Unix的嵌套字，是传输层协议TCP/UDP上的抽象层， iOS中提供BSD Socket编程接口

TCP适用于准确性高的服务： 文件传输，邮件和远程登录
UDP适用于IM, 在线视频，通话和游戏 要求延迟低，效率高，准确率可以忍

#### TCP的三次握手和四次挥手


#### httpdns 


##### httpdns 域名不匹配的问题 kCFStreamErrorDomainSSL
SSL shakehand的过程：
  1. 客户端发起握手请求，携带随机数，支持算法列表等参数
  2. 服务端收到请求，选择合适的算法，下发publickey公钥 和 随机数
  3. 客户端对服务端证书进行校验，并发送随机数，使用公钥加密
  4. 服务端通过私钥获取随机数信息
  5. C/S根据上面的交互信息生成session ticket，用作该连接后续数据传输的加密秘钥

第三部C校验S的证书
1. 客户端用保存在本地的够根证书解开证书链条，确认服务端下发的证书是由可信任的机构颁发
2. 客户端需要校验证书的demain域和扩展域，看是否包括本次的host

当客户端使用httpdns解析域名时，请求url中的host会被替代为httpdns解析出来的ip，所以在证书校验的第二步会出现demain不匹配的情况，导致ssl/tls握手失败

解决方法：
hook证书校验的第二步，将ip直接替换为原来的域名，进行证书校验（在发起请求在header里设置host为原来的域名）

``` C
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * __nullable credential))completionHandler
{
    if (!challenge) {
        return;
    }
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    NSURLCredential *credential = nil;
    /*
     * 获取原始域名信息。
     */
    NSString* host = [[self.request allHTTPHeaderFields] objectForKey:@"host"];
    if (!host) {
        host = self.request.URL.host;
    }
    // 检查质询的验证方式是否是服务器端证书验证，HTTPS的验证方式就是服务器端证书验证
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        if ([self evaluateServerTrust:challenge.protectionSpace.serverTrust forDomain:host]) {
            disposition = NSURLSessionAuthChallengeUseCredential;
            credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        } else {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        }
    } else {
        disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    }
    // 对于其他的challenges直接使用默认的验证方案
    completionHandler(disposition,credential);
}

- (BOOL)evaluateServerTrust:(SecTrustRef)serverTrust
                  forDomain:(NSString *)domain {
    /*
     * 创建证书校验策略
     */
    NSMutableArray *policies = [NSMutableArray array];
    if (domain) {
        [policies addObject:(__bridge_transfer id)SecPolicyCreateSSL(true, (__bridge CFStringRef)domain)];
    } else {
        [policies addObject:(__bridge_transfer id)SecPolicyCreateBasicX509()];
    }
    /*
     * 绑定校验策略到服务端的证书上
     */
    SecTrustSetPolicies(serverTrust, (__bridge CFArrayRef)policies);
    /*
     * 评估当前serverTrust是否可信任，
     * 官方建议在result = kSecTrustResultUnspecified 或 kSecTrustResultProceed
     * 的情况下serverTrust可以被验证通过，https://developer.apple.com/library/ios/technotes/tn2232/_index.html
     * 关于SecTrustResultType的详细信息请参考SecTrust.h
     */
    SecTrustResultType result;
    SecTrustEvaluate(serverTrust, &result);
    return (result == kSecTrustResultUnspecified || result == kSecTrustResultProceed);
}
```



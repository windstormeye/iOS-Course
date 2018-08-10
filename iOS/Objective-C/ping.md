**这篇文章是我在项目中需要判断内外网环境根据网上的资料及自己的改造所得结果，有些不足之处望指出。**

使用`ping`命令来检测数据包（ICMP，Internet Control Message Protocol，互联网控制报文协议）能够通过IP协议到达特定主机，并收到主机的应答，以检查网络是否连通和网络连接速度，帮助我们分析和判定网络故障。因为互联网操作是路由器严密监控的。当路由器端处理报文时若有意外发生，事件通过ICMP报告给发送端。

SimplePing是Appl给开发者提供的一套封装了底层`BSD Sockets ping`函数的类，SimplePing下载地址：[https://developer.apple.com/library/content/samplecode/SimplePing/Introduction/Intro.html](https://developer.apple.com/library/content/samplecode/SimplePing/Introduction/Intro.html)


下面我们一一介绍 SimplePing 类的各个属性、方法以及`delegate`回调方法的含义及作用。

### 初始化方法

```ObjC
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithHostName:(NSString *)hostName NS_DESIGNATED_INITIALIZER;
```

SimplePing中，禁用了`init`方法，只提供`initWithHostName:`这个指定构造方法，它可以用于初始化一个`ping`指定的主机实例对象。其中`hostName`参数可以是主机的`DNS`域名，或者是`IPv4/IPv6`地址的字符串形式。

```Objc
@property (nonatomic, copy, readonly) NSString * hostName;
```
hostName：只读，保存由初始化方法`initWithHostName:`传入的`ping`操作要连接的主机域名或`IP`地址。

```ObjC
@property (nonatomic, assign, readwrite) SimplePingAddressStyle addressStyle;
```
addressStyle：主机的`IP`地址类型，如`IPv4/IPv6`等，其中`SimplePingAddressStyle`枚举类型的定义如下：

```ObjC
typedef NS_ENUM(NSInteger, SimplePingAddressStyle) {
        SimplePingAddressStyleAny,    // IPv4 或 IPv6
        SimplePingAddressStyleICMPv4, // IPv4
        SimplePingAddressStyleICMPv6  // IPv6
    };
```

```ObjC
@property (nonatomic, copy, readonly, nullable) NSData * hostAddress;
```

hostAddress：只读，在`start`方法调用之后，根据`hostName`得到的要`ping`的主机的`IP`地址，它是`struct sockaddr`形式的`NSData`数据。当`SimplePing`实例处于`stopped`状态，或者实例调用了`start`方法，但在`simplePing:didStartWithAddress:`方法被调用之前，hostAddress 的值都是`nil`。

```ObjC
@property (nonatomic, assign, readonly) sa_family_t hostAddressFamily;
```

hostAddressFamily：只读，`hostAddress`的地址族，如果`hostAddress`为`nil`，则其值为：`AF_UNSPEC`。

```ObjC
@property (nonatomic, assign, readonly) uint16_t identifier;
```

identifier：只读，当创建一个`SimplePing`实例对象时，会自动生成一个的随机的标识符，用来唯一标识当前`ping`对象。

```ObjC
@property (nonatomic, assign, readonly) uint16_t nextSequenceNumber;
```

nextSequenceNumber：只读，`ping`每发送一次数据包都会有一个对应的序列号`sequence number`，此值为下一次`ping`操作要发送数据时的序列号，从0开始递增，当`ping`成功发送一次数据到主机并收到应答时，该值+1。而对于本次`ping`的`sequence number`在成功发送数据(request)和成功接收到响应(response)的`delegate`回调方法里都会以方法参数返回，以便进行`ping`操作耗时的计算等等。

```ObjC
@property (nonatomic, weak, readwrite, nullable) id delegate;
```
delegate：当前对象的回调，`delegate`中的回调方法将在对象调用`start`方法所在的线程对应的`run loop`中以默认的`run loop model`执行。

### 实例方法:

```ObjC
- (void)start;
```

start 方法：开始一个`ping`操作，在调用此方法前，必须给`SimplePing`实例对象的`delegete`以及其他参数赋值。当`start`方法成功执行时，会回调`delegate`中的`simplePing:didStartWithAddress:`方法，在该回调方法里，就可以通过`sendPingWithData:`开始发送`ICMP`数据包，并等待接受主机应答的数据包。另外需要注意的是，当一个实例已经`started`，又一次调用此`start`方法会出错。

```ObjC
- (void)sendPingWithData:(nullable NSData *)data;
```

sendPingWithData: 方法：向主机发送特定格式的`ICMP`数据包，调用此方法前必须保证实例已经`started`并且要等待`simplePing:didStartWithAddress:`回调执行才能开始发送数据。参数`data`为要向主机发送的`ICMP`数据包，可以为`nil`，默认会发一个标准的`64 byte`数据包。

```ObjC
- (void)stop;
```

stop 方法：当结束要`ping`操作时，调用此方法。与`start`方法不同的是，当一个实例已经`stopped`，再次调用此方法也没事。

`delegate`回调方法:

`start`方法执行结果的回调:

```ObjC
// start 方法成功执行，可在此开始发送数据，其中 address 为主机的 IP 地址；
- (void)simplePing:(SimplePing *)pinger didStartWithAddress:(NSData *)address;
// start 方法执行失败，返回错误信息；
- (void)simplePing:(SimplePing *)pinger didFailWithError:(NSError *)error;
```
sendPingWithData: 方法执行结果的回调，每发送一次数据，都会同步地回调以下两个方法其中一个（除非你在发送途中调用了`stop`方法）：

```ObjC
// 成功发送 ICMP 数据包到指定主机，在此传回已发送的数据包以及本次 ping 对应的序列号；
- (void)simplePing:(SimplePing *)pinger didSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber;
// 发送数据失败，并返回错误信息，绝大部分原因由于 hostName 解析失败。另，当此方法调用时，ping 实例状态会自动转为stopped，不用再显示调用stop方法；
- (void)simplePing:(SimplePing *)pinger didFailToSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber error:(NSError *)error;
```

接收到主机返回应答数据的回调：
```ObjC
// 成功接收到主机回传的与之前发送相匹配的 ICMP 数据包；
- (void)simplePing:(SimplePing *)pinger didReceivePingResponsePacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber;
// 收到的未知的数据包。
- (void)simplePing:(SimplePing *)pinger didReceiveUnexpectedPacket:(NSData *)packet;
```

注：以上回调方法中的 packet 数据包只包含了 ICMP header 和 sendPingWithData: 中传入的数据，但不包含任何 IP 层的 header。



封装了一个简单`SimplePing`类，因为只有一个类，就不开repo了：

```ObjC
//
//  PJPingManager.h
//  DiDiData
//
//  Created by PJ on 2018/5/25.
//  Copyright © 2018年 pjhubs All rights reserved.
//

#import <Foundation/Foundation.h>

typedef void(^PingSuccessCallback)();
typedef void(^PingFailureCallback)();

@interface IPLPingManager : NSObject

@property (nonatomic, copy) PingSuccessCallback pingSuccessCallback;
@property (nonatomic, copy) PingFailureCallback pingFailureCallback;

- (void)startPing;

@end

```

```ObjC
//
//  PJPingManager.m
//  DiDiData
//
//  Created by PJ on 2018/5/25.
//  Copyright © 2018年 pjhubs All rights reserved.
//

#import "PJPingManager.h"
#import "SimplePing.h"
#include <netdb.h>

@interface PJPingManager ()<SimplePingDelegate>

@property (nonatomic, strong) SimplePing *pinger;
@property (nonatomic, strong) NSTimer *sendTimer;


@end

@implementation PJPingManager

- (instancetype)init {
    self = [super init];
    if (self) {
        NSString *hostName = @"your hostName";
        self.pinger = [[SimplePing alloc] initWithHostName:hostName];
        self.pinger.addressStyle = SimplePingAddressStyleAny;
        self.pinger.delegate = self;
    }
    return self;
}

- (void)startPing {
    [self start];
}

- (void)start {
    [self.pinger start];
}

- (void)stop {
    [self.pinger stop];
    self.pinger = nil;
    
    if ([self.sendTimer isValid])
    {
        [self.sendTimer invalidate];
    }
    self.sendTimer = nil;
}

- (void)sendPing {
    [self.pinger sendPingWithData:nil];
}

#pragma mark - pinger delegate

- (void)simplePing:(SimplePing *)pinger didStartWithAddress:(NSData *)address {
    NSLog(@"pinging %@", [self displayAddressForAddress:address]);
    
    [self sendPing];
    
    
}

- (void)simplePing:(SimplePing *)pinger didFailWithError:(NSError *)error {
    NSLog(@"failed: %@", [self shortErrorFromError:error]);
    
    [self stop];
    
    if (self.pingFailureCallback) {
        self.pingFailureCallback();
    }
}

- (void)simplePing:(SimplePing *)pinger didSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber {
    NSLog(@"#%u sent", (unsigned int) sequenceNumber);
}

- (void)simplePing:(SimplePing *)pinger didFailToSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber error:(NSError *)error {
    NSLog(@"#%u send failed: %@", (unsigned int) sequenceNumber, [self shortErrorFromError:error]);
    
    [self stop];
    
    if (self.pingFailureCallback) {
        self.pingFailureCallback();
    }
}

- (void)simplePing:(SimplePing *)pinger didReceivePingResponsePacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber {
    NSLog(@"#%u received, size=%zu", (unsigned int) sequenceNumber, (size_t) packet.length);
    
    [self stop];
    
    if (self.pingSuccessCallback) {
        self.pingSuccessCallback();
    }
}

- (void)simplePing:(SimplePing *)pinger didReceiveUnexpectedPacket:(NSData *)packet {
    NSLog(@"unexpected packet, size=%zu", (size_t) packet.length);
    
    [self stop];
    
    if (self.pingSuccessCallback) {
        self.pingSuccessCallback();
    }
}

#pragma mark - Others mothods

/**
 * 将ping接收的数据转换成ip地址
 * @param address 接受的ping数据
 */
- (NSString *)displayAddressForAddress:(NSData *)address {
    int err;
    NSString *result;
    char hostStr[NI_MAXHOST];
    
    result = nil;
    
    if (address != nil) {
        err = getnameinfo([address bytes], (socklen_t)[address length], hostStr, sizeof(hostStr),
                          NULL, 0, NI_NUMERICHOST);
        if (err == 0) {
            result = [NSString stringWithCString:hostStr encoding:NSASCIIStringEncoding];
        }
    }
    
    if (result == nil) {
        result = @"?";
    }
    
    return result;
}

/*
 * 解析错误数据并翻译
 */
- (NSString *)shortErrorFromError:(NSError *)error {
    NSString *result;
    NSNumber *failureNum;
    int failure;
    const char *failureStr;
    
    result = nil;
    
    // Handle DNS errors as a special case.
    
    if ([[error domain] isEqual:(NSString *)kCFErrorDomainCFNetwork] &&
        ([error code] == kCFHostErrorUnknown)) {
        failureNum = [[error userInfo] objectForKey:(id)kCFGetAddrInfoFailureKey];
        if ([failureNum isKindOfClass:[NSNumber class]]) {
            failure = [failureNum intValue];
            if (failure != 0) {
                failureStr = gai_strerror(failure);
                if (failureStr != NULL) {
                    result = [NSString stringWithUTF8String:failureStr];
                }
            }
        }
    }
    
    if (result == nil) {
        result = [error localizedFailureReason];
    }
    if (result == nil) {
        result = [error localizedDescription];
    }
    if (result == nil) {
        result = [error description];
    }
    
    return result;
}

@end
```

给出的代码中使用到的`netdb`库为Unix和Linux特有的头文件，主要定义了与网络有关的结构、变量类型、宏、函数等。这里有篇在Unix中该函数库相关介绍：[http://pubs.opengroup.org/onlinepubs/7908799/xns/netdb.h.html](http://pubs.opengroup.org/onlinepubs/7908799/xns/netdb.h.html)

`NI_MAXHOST`给出主机字符串存储空间的最大长度，值为1025；

`NI_MAXSERV`给出服务字符串存储空间的最大长度，值为32.

其中还有一些会使用到的宏，如下所示：
宏|解释
--|--
#define    NI_NOFQDN            0x00000001   |   只返回FQDN的主机名部分 
#define    NI_NUMERICHOST       0x00000002   |   以数串格式返回主机字符串                  
#define    NI_NAMEREQD          0x00000004   |   若不能从地址解析出名字则返回错误
#define    NI_NUMERICSERV       0x00000008   |   以数串格式返回服务字符串
#define    NI_NUMERICSCOPE      0x00000100   |   以数串格式返回范围标识字符串
#define    NI_DGRAM             0x00000010   |   数据报服务


使用方法：
1. 新建`ping`类，复制以上代码；
2. 创建一个`NSTimer`类型属性且初始化timer。`self.timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(pingNetWork) userInfo:nil repeats:YES];`。在这里为啥要初始化一个NSTimer呢？因为如果`ping`失败后，也就是发送的测试报文成功，但一直没收到响应的报文，此时却不会有任何的回调方法告知我们，而只`ping`一次结果也不准确，更何况`ping`花费的时间非常之短，故我在此加了个`NSTimer`多次进行`ping`，或者也可以使用`performSelector`进行延时判断，一般0.3~0.8s的延时即可。如果在这期间内未收到响应则可视为超时。
3. 对应的方法为：
```ObjC
- (void)pingNetWork{    
    self.pingManager = [[IPLPingManager alloc] init];
    self.pingManager.pingSuccessCallback = ^{
        
    };
    
    self.pingManager.pingFailureCallback = ^{

    };
    
    [self.pingManager startPing];
}
```

跑起工程后每秒都会`ping`目标主机，如果你不并想每秒都执行一次，再自定义一个属性去标记吧。当然，你也可也完全不必使用我提供的这个封装，直接使用`SimplePing`自行编写也行。
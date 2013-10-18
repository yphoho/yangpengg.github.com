---
layout: post
title: "Connect to LEGO EV3 under Mac OS X with Bluetooth"
date: 2013-10-13 20:29
comments: true
categories: [Misc, EV3]
---


最近买了一套 LEGO Mindstorms EV3 45544，本来买的时候想照着[Tilted Twister](http://www.tiltedtwister.com/tiltedtwister2.html)做一个解魔方的机器人，没想到买的机器人版本太新，不只building instruction不能照搬，因为[NXC](http://bricxcc.sourceforge.net/nbc/)还没有发布支持EV3的版本，连代码也用不了。

## EV3与NXT的不同

LEGO前两代的机器人叫NXT，我没有查过NXT的资料，但是估计只是在硬件上自己写了个简单的OS，能够解释LEGO Software编译出来的程序。所以在这层关系上LEGO的OS相当于JVM，Software编译出来的程序相当于class文件，源码文件(一种图形语言，被称为G语言，也叫nxt-g)相当于JAVA源码。

EV3与前两代最大的不同在于，EV3底层运行了一个标准的Linux，在Linux之上LEOG自己开发了一个VM，称为lms2012。换句话说，EV3现在在软件上完全与JVM对应。

EV3的firmware是开源的，源码托管在github上：

```
git clone https://github.com/mindboards/ev3sources.git
```

关于源码结构，这里有[一篇文章](http://www.chihayafuru.jp/etrobo/?p=2778)可以了解个大概，只不过是日文的，不过有google，日文也是小事。

目录结构大致如下：

* extra下是Linux内核，lms2012是 LEGO VM 相关。lms2012中的内容：
* open_first是编译脚本；
* d_xxxx是 VM 要用到的内核模块；
* lms2012和c_xxxx是 LEGO VM 的源码和库函数；
* lmssrc好像是EV3显示屏上对应的程序；

关于EV3更多的内容，可以在[这个wiki](http://www.mindstorms.rwth-aachen.de/trac/wiki/EV3)上找到。


## LEGO Mindstorms编程语言

### nxt-g

LEGO的机器人是给小孩子玩的，所以LEGO官方只支持一种图形编程语言，叫nxg-g(也就是G语言)。这个东西被LEGO称为凡是C和其它语言可以实现的，G语言也可以实现，不过可以想象一下如果用G实现解魔方的算法，是一件多么dan疼的事情。

### [nxc](http://bricxcc.sourceforge.net/nbc)

这应该是玩LEGO机器人用的最多的语言。nxc实现了一个c语法的子集。从编译过程看，nxc把用c写出来的的程序翻译成一种汇编语言，然后交给nbc再翻译成LEGO VM可以识别的字节码。

虽然只是一个c的子集，但是对程序员而言，这实在是太棒了。而且因为nxc直接把程序翻译成VM可以运行的字节码，也就是说可以在不重新刷EV3官方firmware的情况下直接使用。之所以这么说，是为了引出：

### [lejos](http://www.lejos.org/)

j指JAVA，名字里有os，也就是说，这完全是重新实现的firmware，想用这个东西写程序，只能刷lejos的firmware。

这个东西在EV3下的实现是用自己的java程序取代了LEGO VM(也就是上边提到的lms2012)。因为自己实现VM，而且编程语言用的是JAVA，所以对程序员而言也是方便了很多。

### robotc

一个商业的闭源玩意儿，不多说。


## Mac下使用蓝牙连接EV3

好吧，言归正传，我想用EV3搭出一个可以解魔方的机器人。理论上这事可行，因为有人用NXT搭出来过，NXT都可以，更不要说EV3了。关于使用NXT搭解魔方机器人，有详细资料的至少有两个：一个是上边提到的[Tilted Twister](http://www.tiltedtwister.com/tiltedtwister2.html)，另一个是[MindCuber](http://www.mindcuber.com/)。

既然理论上可行，那现在面临的困难有两个：

* 硬件部分，让我这种初玩LEGO，又没有任何艺术细胞的人拼出一个，实在是太难了；
* 软件部分，到这篇blog写作时间为止，nxc, lejos都没有正式的版本放出来，如我之前所言，用nxt-g写算法是多么的，呵呵…

好吧，实际上要搭这样的一个机器人，不会遇到上述两个难题之外的问题。所以我好像什么也没有做...

一个好的消息是，MindCuber正在出EV3的机器人版本，[MindCub3r](http://www.mindcuber.com/mindcub3r/mindcub3r.html)。如果偷个懒，我可以等MindCub3r出来直接参考(MindCub3r会用31313拼，而我的EV3是45544，所以硬件部分只能参考，软件部分应该可以完全Copy)。不过作为一个程序员，至少要做点什么吧。

好吧，我决定试着先写出软件部分，毕竟这个对一个程序员而言，还是比较容易的。但是肯定不是用nxt-g这种玩具语言写。

大体的思路是，扫描魔方的代码用nxt-g实现，这个应该很容易。EV3扫描完魔方之后，扫描的数据传到另一个地方，在另一个地方我可以用正常点的语言实现解魔方的算法，然后回传解法给EV3，EV3再按照解法执行。那么按照这个思路，实现方案有以下两个：

* 如之前说的，EV3底层运行着一个标准的Linux，这样我可以编译好一个解魔方算法的c程序(如何让EV3运行native programs见[这里](http://robotnav.wordpress.com/ev3/))，让nxg-g扫描完之后，启动这个程序，然后nxg-g取回解法执行。这个方案可以说perfect，虽然编译native程序比较费事，但是实际解魔方时，除了EV3，再加一个sd卡就是所有的硬件了。但是世界总是很不完美的，这个方案的问题在于：到目前为止，我还没有发现如何在nxt-g的程序里调用native程序。毕竟nxt-g程序是运行在LEGO VM里的，它不是shell，可以一条命令就调用外部程序。好吧，在这个接口没有找到之前，这个方案只好放弃了。
* nxt-g扫描完魔方之后，把数据传到一台pc，我可以直接在pc上写算法，然后回传解法。EV3与外界的无线通讯方式有两种：wifi和bluetooth。好吧，nxt-g是没有wifi通信接口的，所以没有的选，只能用bluetooth了。谢天谢地，总算有个还能用。

写到这里总算是跟题目有点关系了。

在Linux下，蓝牙的协议栈用的是bluez，编程的接口用的也是标准的socket编程。可以在[这里](http://lirobo.blogspot.com/2012/08/linux-nxt.html)找到Linux下连接NXT的源码。本来问题在这里也就解决，但是...

* 我家里用的是MacBook，不是Linux；
* (这个是之后才发现的)EV3与NXT蓝牙通信协议不一样，而且EV3没有官方的蓝牙协议文档。所以上边提到的程序即使在Mac OS X下实现，也不能正常通信；

先解决第一个问题，这个问题要解决两个方面，(1)找到Mac下蓝牙的编程接口，(2)编译一个命令行objc程序。

第一个问题的第一方面，Apple实现了自己蓝牙的协议栈，主要的参考文档包括，[Introduction to Bluetooth Device Access Guide](https://developer.apple.com/library/mac/documentation/devicedrivers/Conceptual/Bluetooth/BT_Intro/BT_Intro.html), [Bluetooth Framework Reference](https://developer.apple.com/library/mac/documentation/DeviceDrivers/Reference/IOBluetooth/_index.html)。这个文档很坑的提到，bluetooth同时提供c和objc的接口，但是却没有在文档中写半个c接口的代码。好吧，只能用objc写了。

关于第二方面，实际上是由于没有找到c接口的引起的，因为用objc接口实现通信逻辑，就只能用delegate模型，实现一个事件驱动的程序。

关于第二个问题，直接上代码(代码中涉及的协议在EV3的源码文件 [c_com.h](https://github.com/mindboards/ev3sources/blob/master/lms2012/c_com/source/c_com.h)。说到这个h文件，协议的大部分内容是从注释中得到到，但是坑die的是，关于mailbox的注释有一处是错的，直接导致在这里卡了很长时间，不过还好最后搞定了。)：

```
//  ev3_mailbox.m

#import <Foundation/NSObject.h>
#import <IOBluetooth/objc/IOBluetoothDevice.h>
#import <IOBluetooth/objc/IOBluetoothDeviceInquiry.h>
#import <IOBluetooth/objc/IOBluetoothRFCOMMChannel.h>
#import <IOBluetooth/IOBluetoothUtilities.h>

#import <stdio.h>


#define MAX_MESSAGE_SIZE 64

int create_mailbox_send_data(const char *mailbox_name, const char *msg, unsigned char *out_buffer);
int parse_mailbox_data(const unsigned char *data, int size, char *mailbox_name, char *msg);

void hex_dump(const unsigned char *data, int size);


@interface ConnectionHandler : NSObject
{

}

- (void)rfcommChannelOpenComplete:(IOBluetoothRFCOMMChannel*)channel status:(IOReturn)status;
- (void)rfcommChannelData:(IOBluetoothRFCOMMChannel*)channel data:(void *)dataPointer length:(size_t)dataLength;

@end


@implementation ConnectionHandler

- (void)rfcommChannelOpenComplete:(IOBluetoothRFCOMMChannel*)channel status:(IOReturn)status
{
    if(status != kIOReturnSuccess) {
        printf("Connection error!\n");
        CFRunLoopStop(CFRunLoopGetCurrent());
        return;
    }
    printf("connection complete\n");

    char *mailbox_name = "abc";
    char *msg = "ping";
    // char *msg = "hello, ev3";
    unsigned char mailbox_cmd[MAX_MESSAGE_SIZE];
    int size = create_mailbox_send_data(mailbox_name, msg, mailbox_cmd);

    hex_dump(mailbox_cmd, size);

    if([channel writeSync:mailbox_cmd length:size] != kIOReturnSuccess) {
        printf("sendmsg failed\n");
        [channel closeChannel];
        CFRunLoopStop(CFRunLoopGetCurrent());
        return;
    }

    // CFRunLoopStop(CFRunLoopGetCurrent());
}

- (void)rfcommChannelData:(IOBluetoothRFCOMMChannel*)channel data:(void *)dataPointer length:(size_t)dataLength
{
    printf("data received length: %zd\n", dataLength);

    hex_dump((unsigned char *)dataPointer, dataLength);

    char mailbox_name[MAX_MESSAGE_SIZE];
    char msg[MAX_MESSAGE_SIZE];

    parse_mailbox_data((unsigned char *)dataPointer, dataLength, mailbox_name, msg);
    printf("mailbox name: %s\nmsg: %s\n", mailbox_name, msg);

    [channel closeChannel];
    CFRunLoopStop(CFRunLoopGetCurrent());
}

@end



int main( int argc, const char *argv[] )
{
    @autoreleasepool {
        NSString *ev3_addr_str = @"xx:xx:xx:xx:xx:xx";

        IOBluetoothDevice *ev3 = [IOBluetoothDevice deviceWithAddressString:ev3_addr_str];

        ConnectionHandler *handler = [[ConnectionHandler alloc] init];

        IOBluetoothRFCOMMChannel *channel;
        [ev3 openRFCOMMChannelAsync:&channel withChannelID:1 delegate: handler];

        CFRunLoopRun();
    }

    return 0;
}


int create_mailbox_send_data(const char *mailbox_name, const char *msg, unsigned char *out_buffer)
{
    // message counter
    out_buffer[2] = 0x12;
    out_buffer[3] = 0x34;

    // system command without reply, WRITEMAILBOX
    out_buffer[4] = 0x81;
    out_buffer[5] = 0x9e;

    int cur_p = 6;

    // mailbox name
    int size = strlen(mailbox_name) + 1;
    out_buffer[cur_p++] = size;
    memcpy(&out_buffer[cur_p], mailbox_name, size);
    cur_p += size;

    // payload
    size = strlen(msg) + 1;
    out_buffer[cur_p++] = size & 0xff;
    out_buffer[cur_p++] = (size & 0xff00) >> 8;
    memcpy(&out_buffer[cur_p], msg, size);
    cur_p += size;

    // command size
    out_buffer[0] = (cur_p - 2) & 0xff;
    out_buffer[1] = ((cur_p - 2) & 0xff00) >> 8;

    return cur_p;
}


int parse_mailbox_data(const unsigned char *data, int length, char *mailbox_name, char *msg)
{
    assert(data[0] + (data[1] << 8) + 2 == length);
    assert(data[4] == 0x81 && data[5] == 0x9e);

    int cur_p = 6;

    int size = data[cur_p++];
    memcpy(mailbox_name, &data[cur_p], size);
    cur_p += size;

    size = data[cur_p] + (data[cur_p+1] << 8);
    cur_p += 2;
    memcpy(msg, &data[cur_p], size);
    cur_p += size;

    assert(cur_p == length);

    return size;
}


void hex_dump(const unsigned char *data, int size)
{
    for(int i=0; i<size; i++) {
        printf("%02x", data[i]);
    }
    printf("\n");
}
```

编译如下(编译之前把源码中的 ev3_addr_str 改成EV3的蓝牙地址)：

```
clang -o ev3_mailbox ev3_mailbox.m -framework foundation -framework iobluetooth
```

打开LEGO Software写一个简单的程序：

![Bluetooth Mailbox](/images/blogpng/mailbox.png)

好了，在EV3上执行这个程序，然后在Mac上执行

```
./ev3_mailbox
```

如果不出意外，应该可以听到一声大象叫，而且EV3的显示屏上应该会显示："ping"，然后在pc端会显示："pong"。

"ping"是由Mac发送到EV3的，"pong"是EV3在收到"ping"之后发送回Mac的。

现在软件的问题解决了，就差拼一个解魔方的机器人出来了。


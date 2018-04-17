---
title: More-Audio
date: 2018-01-28 22:22:22
tags:
- iOS
- Audio
---

这段时间陆陆续续的在做一些关于iOS开发细节的东西，先是跟进了音频部分（以下简称为Audio），主要分为以下几大部分：
1. Audio的架构和框架
2. 编解码/文件封装格式
3. 播放系统声音/震动/提示声音
4. 综合demo
5. 使用AVFoundation框架进行中英文语音识别

说起iOS中的Audio，耳熟能详的就是AVFoundation，毕竟它是个全能型的框架，不过的AVFoundation现在的地位可以类比JavaScript现在的地位，JavaScript现在甚至都插手嵌入式开发了🙂。

但也就是这种什么所谓的全能型选手，拥有大而全的技能，却缺少了一些底蕴。也就是在这段时间中，我才发现，居然还有专门针对3D音效的openAL、擅长编解码过程的AudioToolBox等等一些非常优秀的音频处理框架，重点是这些框架都是iOS SDK中本身就提供了的。

根据网上资料，梳理了如下一张在iOS中的音频处理各个框架所处的位置，
<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww%20%283%29.png" width = "100%" height = "100%" align=center />

## 高层服务

### AVAudioPlayer

**基本操作：**播放、暂停、停止、循环等等一些基本的音频播放功能。

**控制：**可对音频进行任意时间位置播放；进度控制。

**其它：**可从文件或缓冲区播放声音；获取音视频关键参数，如音频标题、作者、功率等等。

如果我们并不想实现比如3D立体音效，精确的音频歌词同步等功能，那么这个框架所提供的API是完全足够的，但是如果我们想要的进行一些比如对音频流的捕获，捕获后还要进行一些RTSP、RTMP等流媒体协议的处理，再或者进行一些RAC、PCM或PCM转MP3等一些音频的转码方式处理，那这个框架就非常捉鸡了。🙂但是它能够非常轻松的进行简单的音频操作，如上所示基本操作、控制等。

### AudioQueue

相对于AVAudioPlayer来说，其更加强大！它不仅能够完成播放音频和录制音频，还能够通过AudioQueue拿到音频的原始信息，想想看！我们能够拿到音频的原始信息，那就可以做比如任意的编码解码、一些特效转化如变音等等骚操作！我们还可以进行任意的应用层封装，比如说封装成适用于RTMP、RTSP的流媒体协议处理。

使用Audio Queue，我们只需要进行三个步骤即可：
1. 初始化Audio Queue。添加一些播放源、音频格式等。
2. 管理回调方法。在回调方法中我们可以拿到音频的原始数据。
3. 实例化Audio Queue。使用AudioQueueOutput完成音频的最终播放。

### openAL

emmm，看到openAL我会想到openGL，openGL主要是用于处理一些3D的图像或变化，openAL主要是在声源物体、音效缓冲和收听者这三者之间进行设置来实现3D效果，比如可以设置声源的方向、速度、状态等，所以我们可以听到声音由远及近的这种3D效果。

总的来说，openAL主要有三个方面，
1. 声源的设置；
2. 接收者的控制；
3. 声源模式的设置。例如声源是由远及近运动，还是由近及远运动，我们还可以把声源设置在一个3D空间中。

### AudioFile

对音频文件的信息进行读取（注意不是对音频文件进行编解码），通过AudioFile框架的相关API对一个音频文件信息进行读取，主要有以下几大步骤：

1. AudioFileOpenURL。首先我们要通过一个URL打开音频文件。

2. AudioFileGetPropertyInfo。获取我们想要读取的音频文件信息类型。

3. AudioFileGetProperty。得到相关音频的属性NSLog出来即可。

4. AudioFileClose。关闭音频文件。（打开文件就要关闭文件🙂）

从上我们看到基本上都是归类于Get方法，但是AudioFile也提供了一个丰富的set方法，可以实时的修改对应音频相关信息。

举个🌰🍐！！！

我们首先得引入`#import <AudioToolbox/AudioToolbox.h>
`框架，从Xcode 7开始，我们就不需要手动引入framework了，因为当我们引入iOS SDK中对应的framework中的相关.h文件时，Xcode会自动帮我们导入对应的framework。

```ObjC
    // 首先从应用沙盒中提取音频文件路径
    NSString *audioPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"mp3"];
    // 转置成URL
    NSURL *audioURL = [NSURL fileURLWithPath:audioPath];
    // 打开音频
    // 设置音频文件标识符
    AudioFileID audioFile;
    // 通过转置后的音频文件URL，打开获取到的音频文件
    // kAudioFileReadPermission：只读方式打开音频文件；(__bridge CFURLRef)：只接受C语言风格类型变量，所以我们要用一个强转桥接类型转回去
    AudioFileOpenURL((__bridge CFURLRef)audioURL, kAudioFileReadPermission, 0, &audioFile);
    // 读取
    UInt32 dictionarySize = 0;
    AudioFileGetPropertyInfo(audioFile, kAudioFilePropertyInfoDictionary, &dictionarySize, 0);
    CFDictionaryRef dictionary;
    AudioFileGetProperty(audioFile, kAudioFilePropertyInfoDictionary, &dictionarySize, &dictionary);
    // 经过以上两步，我们就拿到了对应音频的相关信息。再强转桥接类型回去即可。
    NSDictionary *audioDic = (__bridge NSDictionary *)dictionary;
    for (int i = 0; i < [audioDic allKeys].count; i++) {
        NSString *key = [[audioDic allKeys] objectAtIndex:i];
        NSString *value = [audioDic valueForKey:key];
        NSLog(@"%@-%@", key, value);
    }
    CFRelease(dictionary);
    AudioFileClose(audioFile);
```
运行工程后，即可看到对应的log，

<img src="http://7xszq8.com1.z0.glb.clouddn.com/Audio%202.png" width = "60%" height = "60%" align=center />

与iOS Audio有关的framework有：

<!-- | Name | Academy | score | 
| - | :-: | -: | 
| Harry Potter | Gryffindor| 90 | 
| Hermione Granger | Gryffindor | 100 | 
| Draco Malfoy | Slytherin | 90 | -->

| framework Name | uses |
| - | -: |
| MediaPlayer.framework | VC，提供一些控制类ViewController，使用起来较为简单，致命缺点：功能单一，对底层API高度封装、高度集成，不利于自定义 |
| AudioIUnit.framework | 底层，提供核心音频处理插件，例如音频单元类型、音频组件接口、音频输入输出单元，用于控制音频的底层交互 |
| OpenAL.framework | 3D，提供3D音频效果 |
| AVFoundation.framework | 全能型，音频的录制、播放及后期处理等（基于C） |
| AudioToolbox.framework | 编解码，音频编解码格式转化 |


综上所述，在日常开发中我和大家也要重点关注iOS音频架构中的高层服务框架，这部分框架是日常开发中经常会手撸代码的地方，而在framework层面，我们要重点关注AVFoundation，虽然它是一个基于C的framework。🙂，但是它却能够对音频进行精细入微的控制，当我们使用AVFoundation进行录音和播放时，能够拿到音频的原始PCM解码之后的数据，拿到这些数据能够对音频进行特效的处理。如果我们要做一个音频播放类的产品，那么用到MediaPlayer.framework的次数会很多。

在中层服务中，如果大家有对音频做了一些比如RTMP、RTSP等流媒体处理的时，可能会用到Audio Convert Services（感觉我是用不到了😂）。比如这么个场景，当我们使用RTMP进行语音直播的时候，通过麦克风采集到的数据可能是原始的PCM数据，但是我们想在播放时候使用AAC格式进行播放，那就得把PCM转成AAC，那就得用Audio Convert Services这个中间层服务。

当我们想做一些音频加密算法或音频的加密声波，那可能就会使用到中间层的Audio Unit Services，它可以对硬件层进行一些精细的控制。而Audio File Services是对音频文件的封装和变化。因此啊，除了底层服务的相关框架外，中间层和高层服务是需要我们（尤其是我自己🙂）去重点掌握的。


## Audio SystemSound

SystemSound框架用于播放系统声音，比如某些特殊的提示音、震动等，若我们要使用该框架来播放自定义声音，要求对应的音频编码方式为PCM的原始音频，长度一般不超过30秒（你要想超过也没法，只不过不推荐🙂）。

当我们使用该框架调用震动功能时，只能用于iPhone系列设备，iPod和iPad系列均无效，因为只有iPhone系列设备的厚度能够允许塞下震动模块（而且还是改进后的Tapic Engine）。当我们使用该框架播放系统音乐效果时，静音情况下无效；播放提示音乐效果时，无论静音与否均有效。

因此使用SystemSound适用于播放提示音及游戏中的特殊短音效用处会更大。

举个🌰🍐！

```ObjC
    NSString *deviceType = [[UIDevice currentDevice] model];
    if ([deviceType isEqualToString:@"iPhone"]) {
        // 调用正常的震动模块，静音后无效
        AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
    } else {
        UIAlertController *alertVC = [UIAlertController alertControllerWithTitle:@"注意" message:@"您的设备不支持震动" preferredStyle:UIAlertControllerStyleAlert];
        [self presentViewController:alertVC animated:true completion:^{
            
        }];
    }
```
以上是我们进行调用震动模块的测试代码，上文已经说明只有iPhone系列设备中才能体现效果，因此我们最好是加上设备类型判断（当然你可以不加🙂），改框架也是基于C的（比较直接操作底层硬件），代码风格也是趋向于C，实际上就这一句话`    AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
`，大家可以从[这篇文章](http://blog.csdn.net/wlm0813/article/details/51170574)中找到其它SystemSoundID，如果系统提供的音效并不适合我们，那么我们可以载入自定义音效，

```ObjC
    NSURL *systemSoundURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"test" ofType:@"mp3"]];
    // 创建ID
    SystemSoundID systemSoundID;
    AudioServicesCreateSystemSoundID((CFURLRef)CFBridgingRetain(systemSoundURL), &systemSoundID);
    // 注册callBack
    AudioServicesAddSystemSoundCompletion(systemSoundID, nil, nil, soundFinishPlaying, nil);
    // 播放声音
    AudioServicesPlaySystemSound(systemSoundID);
```
分析以上测试代码发现一个有趣的现象，就算是自定义音效也是要通过AudioServicesPlaySystemSound去载入音频文件标识符，所以可以大胆的推测！之所以iOS系统占用这么大的存储空间是有相当大的一部分为系统音效音频资源。不用的音效还没法删除，估计也是怕其他App会用到吧。🙂

## 音频参数（了解的不多，先记录一波）

### 采样率：
常用的如44100，CD就是。还有一些其它的32千赫兹。采样频率越高，所能描绘的声波频率也就越高。

### 量化精度
精度嘛，衡量一个东西的精确程度。是将模拟信号分成多个等级的量化单位。量化的精度越高，声音的振幅就越接近原音。因为我们平时听到的音乐或者声音都是模拟信号，而经过计算机处理的都是数字信号，将模拟信号转换为数字信号的这个过程我们称之为量化。而量化，我们得需要一定的信号来逼近它，这种逼近的过程，也就是量化的过程，这种逼近的精度，也就成为量化精度。所以不管我们如何逼近，那也只是逼近而已，与原来的模拟信息还是有些不同。精度越高，听起来就越细腻

### 比特率
数字信号每秒钟传输的信号量。

看一个综合实例，🌰🍐

<img src="http://7xszq8.com1.z0.glb.clouddn.com/audio3.png" width = "40%" height = "40%" align=center />

通过使用`<AVFoundation/AVFoundation.h>`和` <AudioToolbox/AudioToolbox.h>`框架来完成这个实例，在这个实例中，讲读取一个音频文件，对其进行播放、暂停、停止等操作，并可设置是否静音、循环播放次数、调节音量、时间，并可看到当前音频播放进度。

界面的搭建非常简单，大家自定义即可，只需要拖拽出对应的相关控件属性及方法即可。


```ObjC
// 播放按钮点击事件
- (IBAction)playerBtnClick:(id)sender {
    // 设置音频资源路径
    NSString *playMusicPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"mp3"];
    if (playMusicPath) {
        // 开启Audio会话实例
        [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
        NSURL *musicURL = [NSURL fileURLWithPath:playMusicPath];
        audioPlayer = [[AVAudioPlayer alloc] initWithContentsOfURL:musicURL error:nil];
        audioPlayer.delegate = self;
        audioPlayer.meteringEnabled = true;
        // 设置定时器，每隔0.1秒刷新音频对应文件信息（伪装成实时🙂）
        timer = [NSTimer scheduledTimerWithTimeInterval:0.1 target:self selector:@selector(monitor) userInfo:nil repeats:true];
        [audioPlayer play];
    }
}
```

```ObjC
// 定时器任务
- (void)monitor {
    // numberOfChannels声道数，一般都是2吧，代表左右双声道
    NSUInteger channels = audioPlayer.numberOfChannels;
    NSTimeInterval duration = audioPlayer.duration;
    [audioPlayer updateMeters];
    NSString *peakValue = [NSString stringWithFormat:@"%f, %f\n channels=%lu duration=%lu\n currentTime=%f", [audioPlayer peakPowerForChannel:0], [audioPlayer peakPowerForChannel:1], (unsigned long)channels, (unsigned long)duration, audioPlayer.currentTime];
    self.audioInfo.text = peakValue;
    self.musicProgress.progress = audioPlayer.currentTime / audioPlayer.duration;
}
```

```ObjC
// 暂停按钮点击事件
- (IBAction)pauseBtnClick:(id)sender {
    // 再次点击暂停才会播放
    if ([audioPlayer isPlaying]) {
        [audioPlayer pause];
    } else {
        [audioPlayer play];
    }
}
```

```Objc
// 停止按钮点击事件
- (IBAction)stopBtnClick:(id)sender {
    self.volSlider.value = 0;
    self.timeSlider.value = 0;
    [audioPlayer stop];
}
```

```Objc
// 静音按钮点击方法
- (IBAction)muteSwitchClick:(id)sender {
    // 实际上音量为0即静音
    // 刚好这还是个Switch开关
    audioPlayer.volume = [sender isOn];
}
```

```ObjC
// 调节音频时间方法（UIProgress）
- (IBAction)timeSliderClick:(id)sender {
    [audioPlayer pause];
    // 防止归一化（Xcode默认都是0~1，转化为实际值）
    [audioPlayer setCurrentTime:(NSTimeInterval)self.timeSlider.value * audioPlayer.duration];
    [audioPlayer play];
}
```

```ObjC
// UIStepper点击事件（音频循环播放）
- (IBAction)cycBtnClick:(id)sender {
    audioPlayer.numberOfLoops = self.cyc.value;
}
```


<img src="http://7xszq8.com1.z0.glb.clouddn.com/Jan-29-2018%2023-11-32.gif" width = "40%" height = "40%" align=center />

## 语音识别

在iOS 7之后，AVFoundation提供了语音识别功能，使用它非常的简单，

```ObjC
// 语音识别控制器
AVSpeechSynthesizer* speechManager = [[AVSpeechSynthesizer alloc] init];
speechManager.delegate = self;
// 语音识别单元
AVSpeechUtterance* uts = [[AVSpeechUtterance alloc] initWithString:@"23333"];
uts.rate = 0.5;
[speechManager speakUtterance:uts];
```
需要注意，如果本机系统语言设置成了英文是不能够识别中文的喔！

[相关Demo见这。](https://github.com/windstormeye/iOSMorePractices/tree/master/audioPractices)
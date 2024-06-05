---
title: 适配器模式（Adapter）
date: 2021-06-22 14:22:38
tags:
- 设计模式
---

适配器模式属于结构型模式，具体有对于类或对象的不同形态。

<!-- more -->

有的时候，我们需要实现两个不同接口的类之间的通信。基于不修改这两个类的前提，我们需要某个**中间件**（适配器）完成衔接的过程，此时我们可以使用适配器模式。

《设计模式：可复用面向对象软件的基础》中的介绍：

    “将一个类的接口转换为客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作”

可让两个原本不兼容的接口完成无缝对接。


# 结构图
**类适配器**模式：

![](adapter/class-adapter.gif)

<br/>

**对象适配器**模式：

![](adapter/instance-adapter.gif)

<br/>

以上，可见适配器模式包括：

`Target`
* 目标抽象类

`Adapter`
* 适配器类
* 通过在内部包装一个 Adaptee，将源接口转换成目标接口

`Adaptee`
* 适配者类
* 需要适配的类

`Client`
* 客户类

<br/>

# 实现
继承或依赖（推荐）

以类适配器实现为例：

![](adapter/adapter-example.png)

说明：
* 想让 AudioPlayer 播放其它格式的音频文件
* AudioPlayer 使用适配器类 MediaAdapter 传递所需的音频类型，不需要知道能播放所需格式音频的实际类


## 示例代码

Adaptee 适配者类接口及其实现类：
```java
public interface AdvancedMediaPlayer {

    public void playVlc(String fileName);
    public void playMp4(String fileName);
}

// 实体类： 
public class VlcPlayer implements AdvancedMediaPlayer {

    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name: "+ fileName);
    }

    @Override
    public void playMp4(String fileName) {
        // 什么也不做
    }
}

public class Mp4Player implements AdvancedMediaPlayer {

    @Override
    public void playVlc(String fileName) {
        // 什么也不做
    }

    @Override
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file. Name: "+ fileName);
    }
}
```

适配器类接口及其实现：
```java
public interface MediaPlayer {
    public void play(String audioType, String fileName);
}

public class MediaAdapter implements MediaPlayer {

    AdvancedMediaPlayer advancedMusicPlayer;

    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc") ) {
            advancedMusicPlayer = new VlcPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer = new Mp4Player();
        }
    }

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer.playMp4(fileName);
        }
    }
}
```

封装对适配器类的调用：
```java
public class AudioPlayer implements MediaPlayer {

    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        // 播放 mp3 音乐文件的内置支持
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file. Name: "+ fileName);
        }
        // 适配器 mediaAdapter 提供了播放其他文件格式的支持
        else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);

        } else {
            System.out.println("Invalid medium. "+ audioType + " format not supported");
        }
    }
}
```

客户类的调用：
```java
audioPlayer.play("mp3", "beyond the horizon.mp3");
audioPlayer.play("mp4", "alone.mp4");
audioPlayer.play("vlc", "far far away.vlc");
audioPlayer.play("avi", "mind me.avi");
```

<br/>

# 适用性
1. 系统需要使用现有的类，而此类的接口不符合系统的需要。
2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。
3. 通过接口转换，将一个类插入另一个类系中。
    * 比如：轿车和皮卡，现在多了一个 SUV
    * 在不增加实体的需求下，增加一个适配器，在里面包容一个皮卡，实现轿车的接口


# 优缺点
优点
1. 可以让任何两个没有关联的类一起运行。
2. 提高了类的复用。
3. 增加了类的透明度。
4. 灵活性好。

缺点
1. 过多地使用适配器，会让系统非常零乱，不易整体进行把握。
    * 如：明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现
    * 一个系统如果太多出现这种情况，无异于一场灾难。
    * 因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。
2. 由于 Java 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。
# VideoClip
Java,Android实现MP4裁剪功能

### 概述
>主要用的是mp4Parser(根据关键帧进行裁剪) - 自己进行封装以及内部代码处理等,便于使用
>
> <a href="http://blog.csdn.net/sdvch/article/details/38348475">MP4的分割和拼接介绍</a>
>
> <a href="https://github.com/sannies/mp4parser">mp4Parser - github</a>


### 实现代码：

> 具体实现类
> 
> * TrimVideoUtils 裁减视频工具类

### 使用方法：

```java
package com.clip;

import java.io.File;
import java.io.IOException;

import com.clip.TrimVideoUtils.TrimFileCallBack;

public class ClipMain {

    public static void main(String[] args) {
        cutVideo();
    }

    public static void cutVideo() {
        TrimVideoUtils trimVideoUtils = TrimVideoUtils.getInstance();
        // 设置回调
        trimVideoUtils.setTrimCallBack(new TrimFileCallBack() {
            @Override // 裁剪失败回调
            public void trimError(int eType) {
                switch(eType){
                case TrimVideoUtils.FILE_NOT_EXISTS: // 文件不存在
                    System.out.println("视频文件不存在");
                    break;
                case TrimVideoUtils.TRIM_STOP: // 手动停止裁剪
                    System.out.println("停止裁剪");
                    break;
                case TrimVideoUtils.TRIM_FAIL:
                default: // 裁剪失败
                    System.out.println("裁剪失败");
                    break;
                }
            }
            @Override // 裁剪成功回调
            public void trimCallback(boolean isNew, int startS, int endS, int vTotal, File file, File trimFile) {
                /**
                 * 裁剪回调
                 * @param isNew 是否新剪辑
                 * @param starts 开始时间(秒)
                 * @param ends 结束时间(秒)
                 * @param vTime 视频长度
                 * @param file 需要裁剪的文件路径
                 * @param trimFile 裁剪后保存的文件路径
                 */
                // ===========
                System.out.println("isNew : " + isNew);
                System.out.println("startS : " + startS);
                System.out.println("endS : " + endS);
                System.out.println("vTotal : " + vTotal);
                System.out.println("file : " + file.getAbsolutePath());
                System.out.println("trimFile : " + trimFile.getAbsolutePath());
            }
        });
        // ========
        // 需要裁剪的视频路径
        String videoPath = "E:\\clip\\Xperia.mp4";
        // 保存的路径
        String savePath = "E:\\clip\\save\\Xperia_" + System.currentTimeMillis() + "_cut.mp4";
        // ==
        final File file = new File(videoPath); // 视频文件地址
        final File trimFile = new File(savePath);// 裁剪文件保存地址
        final int startS = 1; // 开始时间(秒数 - 非毫秒)
        final int endS = 20; // 结束时间(秒数 - 非毫秒)
        // 进行裁剪
        new Thread(new Runnable() {
            @Override
            public void run() {
                try { // 开始裁剪
                    TrimVideoUtils.getInstance().startTrim(true, startS, endS, file, trimFile);
                } catch (Exception e) {
                    e.printStackTrace();
                    // 设置回调为null
                    TrimVideoUtils.getInstance().setTrimCallBack(null);
                }
            }
        }).start();

        // 停止裁剪
        // TrimVideoUtils.getInstance().stopTrim();
    }
}
```


### 注意事项

> <b>首先该裁减视频是以关键帧进行裁减，例如视频时长 15秒 然后选择裁减 10-14秒这个时间段，如果该视频关键帧 0.x秒一帧, 则裁减出来接近就是10-14秒这个区间段</b>
> 
> <b>但是有部分视频关键帧间隔比较大，就会导致虽然选择裁减10-14区间段，但是可能并不是从10秒开始，而是自动找距离10秒最近的关键帧秒数进行裁剪</b>
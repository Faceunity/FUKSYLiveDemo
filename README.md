# FUKSYLiveDemo 快速接入文档

FUKSYLiveDemo 是集成了 Faceunity 面部跟踪和虚拟道具功能 和 [金山云直播 SDK](https://github.com/ksvc/KSYLive_iOS)  的 Demo。

本文是 FaceUnity SDK 快速对金山云直播的导读说明，关于 `FaceUnity SDK` 的详细说明，请参看 [FULiveDemo](https://github.com/Faceunity/FULiveDemo/tree/dev)


## 快速集成方法

### 一、导入 SDK

将  FaceUnity  文件夹全部拖入工程中，NamaSDK所需依赖库为 `OpenGLES.framework`、`Accelerate.framework`、`CoreMedia.framework`、`AVFoundation.framework`、`libc++.tbd`、`CoreML.framework`

- 备注: 上述NamaSDK 依赖库使用 Pods 管理 会自动添加依赖,运行在iOS11以下系统时,需要手动添加`CoreML.framework`,并在**TARGETS -> Build Phases-> Link Binary With Libraries**将`CoreML.framework`手动修改为可选**Optional**


### FaceUnity 模块简介
```C
-FUManager              //nama 业务类
-FUCamera               //视频采集类(示例程序未用到
-authpack.h             //权限文件
+FUAPIDemoBar     //美颜工具条,可自定义
+items       //贴纸和美妆资源 xx.bundel文件
      
```

### 二、加入展示 FaceUnity SDK 美颜贴纸效果的  UI

1、在 KSYStreamerVC.m中添加头文件，并创建页面属性

```C
/**faceU  */
#import "FUAPIDemoBar.h"
#import "FUManager.h"

@property (nonatomic, strong) FUAPIDemoBar *demoBar ;
```

2、初始化 UI，并遵循代理  FUAPIDemoBarDelegate ，实现代理方法 `bottomDidChange:` 切换贴纸 和 `filterValueChange:` 更新美颜参数。

```C
// demobar 初始化
-(FUAPIDemoBar *)demoBar {
    if (!_demoBar) {
        _demoBar = [[FUAPIDemoBar alloc] initWithFrame:CGRectMake(0, self.view.frame.size.height - 194 - 37 - 50, self.view.frame.size.width, 194)];
        
        _demoBar.mDelegate = self;
    }
    return _demoBar ;
}
```

#### 切换贴纸

```C
// 切换贴纸
-(void)bottomDidChange:(int)index{
    if (index < 3) {
        [[FUManager shareManager] setRenderType:FUDataTypeBeautify];
    }
    if (index == 3) {
        [[FUManager shareManager] setRenderType:FUDataTypeStrick];
    }
    
    if (index == 4) {
        [[FUManager shareManager] setRenderType:FUDataTypeMakeup];
    }
    if (index == 5) {
        [[FUManager shareManager] setRenderType:FUDataTypebody];
    }
}

```

#### 更新美颜参数

```C
// 更新美颜参数    
- (void)filterValueChange:(FUBeautyParam *)param{
    [[FUManager shareManager] filterValueChange:param];
}
```

### 三、在 `viewDidLoad:` 中初始化 SDK  并将  demoBar 添加到页面上

```C
    /**faceU  */
    [[FUManager shareManager] loadFilter];
    [FUManager shareManager].isRender = YES;
    [FUManager shareManager].flipx = YES;
    [FUManager shareManager].trackFlipx = YES;
    [self.view addSubview:self.demoBar ];
    /**faceU */

```


### 四、图像处理

在  `-(void) setCaptureCfg` 设置图像处理 Block ，并对图像进行处理：

```c
_kit.videoProcessingCallback = ^(CMSampleBufferRef buf){
    selfWeak.ctrlView.lblStat.capFrames += 1; // 统计预览帧率(实际使用时不需要)
    // 在此处添加自定义图像处理, 直接修改buf中的图像数据会传递到观众端
    // 或复制图像数据之后再做其他处理, 则观众端仍然看到处理前的图像

    /*** ------ 加入 FaceUnity 效果 ------ **/
    [selfWeak processFaceUnityWithBuffer:buf];
};

- (void)processFaceUnityWithBuffer:(CMSampleBufferRef)buffer {
    
    CVPixelBufferRef pixelBuffer = CMSampleBufferGetImageBuffer(buffer) ;
    [[FUManager shareManager] renderItemsToPixelBuffer: pixelBuffer];
}
```

### 五、销毁道具和切换摄像头

1 视图控制器生命周期结束时 `[[FUManager shareManager] destoryItems];`销毁道具。

2 切换摄像头需要调用 `[[FUManager shareManager] onCameraChange];`切换摄像头

#### 关于 FaceUnity SDK 的更多详细说明，请参看 [FULiveDemo](https://github.com/Faceunity/FULiveDemo/tree/dev)
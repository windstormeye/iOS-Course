这篇文章主要是用于记录我在使用iOS上进行相机开发的过程中的相关内容总结，因为多媒体是iOS中很大的一块内容，因此不太能够用一篇完整的文章进行描述，因此这篇文章将会持续更新。

在iOS中启用相机功能可以使用`UIImagePickerController`和`AVFoundation`两种做法。

### UIImagePickerController

从类名上可以看出，该类是一个应该是具备了较为完整的相机功能，但是从Apple以往的风格可以猜出这个类已经把相关属性做了高度的封装，开发者唯一能够自定义的地方除了中间4：3的相机画面下的区域，如下所示：

<img src="https://i.loli.net/2018/05/29/5b0d62897cdfb.jpeg" width = "20%" height = "20%" align=center />

如果你对`UIImagePickerController`设置了`showsCameraControls = NO`，此时运行起工程，会发现上图中红线所勾画的区域没了，如下所示：

<img src="https://i.loli.net/2018/05/29/5b0d6569ef365.jpeg" width = "20%" height = "20%" align=center />

对`UIImagePickerController`属性设置可参考如下：
```ObjC
- (UIImagePickerController *)imagePicker{
    if (!_imagePicker) {
        _imagePicker = [[UIImagePickerController alloc]init];
        // 判断现在可以获得多媒体的方式
        if ([UIImagePickerController availableMediaTypesForSourceType:UIImagePickerControllerSourceTypeCamera]) {
            // 设置image picker的来源，这里设置为摄像头
            _imagePicker.sourceType = UIImagePickerControllerSourceTypeCamera;
            // 设置使用哪个摄像头，这里默认设置为前置摄像头
            _imagePicker.cameraDevice = UIImagePickerControllerCameraDeviceRear;
            // 设置摄像头模式为照相
            _imagePicker.cameraCaptureMode = UIImagePickerControllerCameraCaptureModePhoto;
            }
        }
        // 允许编辑
//        _imagePicker.allowsEditing=YES;
        // 设置代理，检测操作
        _imagePicker.delegate=self;
        return _imagePicker;
}
```

其代理为：`UIImagePickerControllerDelegate`，我们创建好想要的自定义`view`后，重新复制给对应的`imagePickerController`变量的`cameraOverlayView`属性即可，接着在该`view`上我们自定义的拍照`Button`的点击事件中调用对应`[imagePickerController takePicture]`方法即可进行拍照，而获取的相机拍照图片会在，

```ObjC
-(void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info{
    UIImage * image = info[UIImagePickerControllerEditedImage];
}
```


### AVFoundation

如果我们想彻底的自定义相机界面，需要直接走`AVFoundation`框架，该框架给我们一系列完整做法，这是我在项目中自定义的一套方法，

```ObjC
//
//  PJCameraView.swift
//  Bonfire
//
//  Created by pjpjpj on 2018/5/27.
//  Copyright © 2018年 #incloud. All rights reserved.
//

import UIKit
import AVFoundation
import Photos

protocol PJCameraViewDelegate {
    func takePhotoImage(image: UIImage)
}

class PJCameraView: UIView, AVCapturePhotoCaptureDelegate {

    private var session: AVCaptureSession?
    private var videoInput: AVCaptureDeviceInput?
    private var imageOutput: AVCapturePhotoOutput?
    private var previewLayer: AVCaptureVideoPreviewLayer?

    private(set) var isFrontCamera: Bool?
    
    public var delegate: PJCameraViewDelegate?
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        initView()
        initAVCaptureSession()
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func initView() {
        self.backgroundColor = UIColor.black
        isFrontCamera = false
    }
    
    private func initAVCaptureSession() {
        session = AVCaptureSession.init()
        
        let device = AVCaptureDevice.default(for: AVMediaType.video)
        videoInput = try! AVCaptureDeviceInput.init(device: device!)
        
        imageOutput = AVCapturePhotoOutput.init()
        let setDic = [
            AVVideoCodecKey : AVVideoCodecType.jpeg
        ]
        let imageSetting = AVCapturePhotoSettings.init(format: setDic)
        imageOutput?.photoSettingsForSceneMonitoring = imageSetting
        
        
        if (session?.canAddInput(videoInput!))! {
            session?.addInput(videoInput!)
        }
        if (session?.canAddOutput(imageOutput!))! {
            session?.addOutput(imageOutput!)
        }
        
        previewLayer = AVCaptureVideoPreviewLayer.init(session: session!)
        previewLayer?.videoGravity = AVLayerVideoGravity.resizeAspect
        previewLayer?.frame = self.frame
        self.layer.addSublayer(previewLayer!)
        
        session?.startRunning()
    }
    
    public func switchCameraControl() {
        
        let animation = CATransition()
        animation.duration = 0.35
        animation.timingFunction = CAMediaTimingFunction.easeInOut
        animation.type = "oglFlip"
        
        var position: AVCaptureDevice.Position?
        if isFrontCamera! {
            position = AVCaptureDevice.Position.back
            animation.subtype = kCATransitionFromRight
        } else {
            position = AVCaptureDevice.Position.front
            animation.subtype = kCATransitionFromLeft
        }
        
        for d: AVCaptureDevice in AVCaptureDevice.DiscoverySession.init(deviceTypes: [AVCaptureDevice.DeviceType.builtInWideAngleCamera], mediaType: AVMediaType.video, position: position!).devices {
            if d.position == position {
                
                previewLayer?.add(animation, forKey: nil)
                
                previewLayer?.session?.beginConfiguration()
                let input = try? AVCaptureDeviceInput(device: d)
                for oldInput in (previewLayer?.session?.inputs)! {
                    previewLayer?.session?.removeInput(oldInput)
                }
                previewLayer?.session?.addInput(input!)
                previewLayer?.session?.commitConfiguration()
                break
            }
        }
        
        isFrontCamera = !isFrontCamera!
    }
    
    public func takePhoto() {
        imageOutput?.capturePhoto(with: AVCapturePhotoSettings.init(format: [
            AVVideoCodecKey : AVVideoCodecType.jpeg
            ]), delegate: self)
    }
    
    func photoOutput(_ output: AVCapturePhotoOutput, didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
        let data = photo.fileDataRepresentation()
        if data != nil {
            let image = UIImage.init(data: data!)
            delegate?.takePhotoImage(image: image!)
            PHPhotoLibrary.shared().performChanges({
                PHAssetChangeRequest.creationRequestForAsset(from: image!)
            }, completionHandler: nil)
        }
    }
}
```

以上是我的一个完整相机模块，直接调用`takePhoto`方法可拍照，调用`switchCameraControl`方法可切换前后相机。


以上为使用两种框架进行基础相机的搭建。


### 注意点

此节为在进行相机开发过程中需要注意的地方。




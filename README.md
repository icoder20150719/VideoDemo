# VideoDemo
[笔记链接](https://github.com/icoder20150719/VideoDemo/blob/master/AVFoundation学习.html)

![image](https://github.com/icoder20150719/VideoDemo/blob/master/image/001.PNG)
![image](https://github.com/icoder20150719/VideoDemo/blob/master/image/002.PNG)
![image](https://github.com/icoder20150719/VideoDemo/blob/master/image/003.PNG)

##iOS视频、音频的录制、播放##
iOS系统视频录制有两种方式，简单的录制使用 UIImagePickerController 就可以了，如果想要实现自定功能的录制就需要用AVFoundation框架AVCaptureSession 了。下面是UIImagePickerController 使用。

###一、使用UIImagePickerController录制视频
    -pragma mark - 弹出视频录制控制器

    - (void)useImagePickerController {
    //声明 UIImagePickerController 对象
    UIImagePickerController *picker = [[UIImagePickerController alloc]init];
    //使用摄像头
    picker.sourceType = UIImagePickerControllerSourceTypeCamera;
    {
    //这个两个属性要一起用 并导入头问题 #import <MobileCoreServices/MobileCoreServices.h>
    picker.mediaTypes = @[(id)kUTTypeMovie];
    picker.cameraCaptureMode = UIImagePickerControllerCameraCaptureModeVideo;
    }
    //使用后置摄像头
    picker.cameraDevice = UIImagePickerControllerCameraDeviceRear;
    //设置代理
    picker.delegate = self;
    [self presentViewController:picker animated:YES completion:nil];
    }
    #pragma mark - UIImagePickerControllerDelegate
    - (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info {
    //保存录制视频信息
    //1、获得视频路径
    NSURL *videoURL = info[UIImagePickerControllerMediaURL];
    //2、保存
    UISaveVideoAtPathToSavedPhotosAlbum(videoURL.absoluteString, self, @selector(video:didFinishSavingWithError:contextInfo:), nil);
    [picker dismissViewControllerAnimated:YES completion:nil];
    }
    -(void)video:(NSString *)videoPath didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo{
    NSLog(@"保存 complete");
    } 
基于AVFoundation框架AVCaptureSession我也模拟系统录制写了一个demo，由于时间原因，一些细节就没有处理。现在先看一下AVCaptureSession的工作流程。

![image](https://github.com/icoder20150719/VideoDemo/blob/master/image/a.png)
###创建AVCaptureSession
    - (void)setupVideoSession {
    AVCaptureSession *session = [[AVCaptureSession alloc]init];
    _session = session;
    [session setSessionPreset:AVCaptureSessionPresetMedium];

    AVCaptureDevice *videoDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    _videoDevice = videoDevice;
    AVCaptureDevice *audioDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeAudio];
    _audioDevice = audioDevice;
    //创建视频输入源
    AVCaptureDeviceInput *videoInput = [[AVCaptureDeviceInput alloc]initWithDevice:videoDevice error:nil];
    _videoInput = videoInput;
    //创建音频输入源
    AVCaptureDeviceInput *audioInput = [[AVCaptureDeviceInput alloc]initWithDevice:audioDevice error:nil];
    _audioInput = audioInput;
    AVCaptureMovieFileOutput *output = [[AVCaptureMovieFileOutput alloc]init];
    _output = output;

    if ([session canAddInput:videoInput]) {
    [session addInput:videoInput];
    }
    if ([session canAddInput:audioInput]) {
    [session addInput:audioInput];
    }
    if ([session canAddOutput:output]) {
    [session addOutput:output];
    }

    AVCaptureVideoPreviewLayer *previewLayer = [AVCaptureVideoPreviewLayer layerWithSession:session];
    _previewLayer = previewLayer;

    _previewLayer.bounds = self.view.bounds;
    _previewLayer.anchorPoint = CGPointZero;
    _previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
    [self.view.layer addSublayer:previewLayer];
    [session startRunning];

    }

### 代理方法 视频保存
    - (void)captureOutput:(AVCaptureFileOutput *)captureOutput didFinishRecordingToOutputFileAtURL:(NSURL *)outputFileURL fromConnections:(NSArray *)connections error:(NSError *)error {
    //保存到相册
    //1、获得视频路径
    //    NSURL *videoURL = outputFileURL;
    //2、保存 不进去
    //    UISaveVideoAtPathToSavedPhotosAlbum(videoURL.absoluteString, self, @selector(video:didFinishSavingWithError:contextInfo:), nil);
    //使用ALAssetsLibrary方法保存成功
    ALAssetsLibrary *library = [[ALAssetsLibrary alloc] init];
    // 将临时文件夹中的视频文件复制到 照片 文件夹中，以便存取
    [library writeVideoAtPathToSavedPhotosAlbum:outputFileURL

    completionBlock:^(NSURL *assetURL, NSError *error) {

    if (!error) {
    NSLog(@"保存 complete");
    [[[UIAlertView alloc]initWithTitle:@"提示" message:@"保存成功！" delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil, nil]show];
    }
    }];

    }

###摄像头切换

    - (void)change {
    [self stop];
    AVCaptureDevicePosition destPosition = self.videoDevice.position == AVCaptureDevicePositionBack? AVCaptureDevicePositionFront:AVCaptureDevicePositionBack;
    AVCaptureDevice *device = [self getVideoDevice:destPosition];
    AVCaptureDeviceInput *input = [[AVCaptureDeviceInput alloc]initWithDevice:device error:nil];

    // 移除之前摄像头输入设备
    [_session removeInput:_videoInput];

    // 添加新的摄像头输入设备
    [_session addInput:input];
    _videoInput = input ;
    _videoDevice = device;
    _previewLayer.session = _session;
    [_session startRunning];
    //开始动画
    [UIView beginAnimations:@"doflip" context:nil];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [UIView setAnimationDuration:0.5];
    [UIView setAnimationTransition:UIViewAnimationTransitionFlipFromLeft  forView:self.view cache:YES];
    [UIView commitAnimations];

    }
###开始录制

    #pragma mark - 开始录制
    - (void)start {
    //开始录制
    NSString *directory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
    NSString *path = [directory stringByAppendingPathComponent:@"demo.MOV"];
    [_output startRecordingToOutputFileURL:[NSURL fileURLWithPath:path] recordingDelegate:self];
    //时间显示
    NSTimer *timer = [NSTimer timerWithTimeInterval:0.5 target:self selector:@selector(recordingTiming) userInfo:nil repeats:YES];
    _timer = timer;
    [[NSRunLoop currentRunLoop]addTimer:_timer forMode:NSRunLoopCommonModes];
    }



# VideoDemo
![Aaron Swartz](https://github.com/younghz/Markdown/raw/master/Res/Aaron_Swartz.jpg)

##iOS视频、音频的录制、播放##
iOS系统视频录制有两种方式，简单的录制使用 UIImagePickerController 就可以了，如果想要实现自定功能的录制就需要用AVFoundation框架AVCaptureSession 了。下面是  UIImagePickerController 使用。

##一、使用UIImagePickerController录制视频
<code>
#pragma mark - 弹出视频录制控制器

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
</code>
基于AVFoundation框架AVCaptureSession我也模拟系统录制写了一个demo，由于时间原因，一些细节就没有处理。现在先看一下AVCaptureSession的工作流程。








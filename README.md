## 鉴于经常还是有人问我尽管有模型，但是还是不知道怎么修改工程，我传上之前的整个测试工程，仅供参考.

# Hisi_YoLoV5
海思nnie跑yolov5

## 模型修改。
### 参考官方的yolov5 4.0版本
+ git clone -b v4.0 https://github.com/ultralytics/yolov5.git/ 

### 修改处0, models/yolov5s.py  增加如下:

+    删除 ~~[[-1, 1, Focus, [64, 3]],  # 0-P1/2 ~~
+   添加 [[-1, 1, Conv, [64, 3, 2]],  # 0-P1/2

### 修改处1, utils/activations.py  增加如下:
    class ReLU(nn.Module): 
        @staticmethod
            def forward(x):
                return nn.ReLU(x)

### 修改处2，models/common.py 35行修改激活函数
+    ~~self.act = nn.SiLU() if act is True else (act if isinstance(act, nn.Module) else nn.Identity())~~

+    添加 self.act = nn.ReLU() if act is True else (act if isinstance(act, nn.Module) else nn.Identity())


### 修改处3, models/comon.py 98行修改maxPool ,修改ceil_mode方式
+    self.m = nn.ModuleList([nn.MaxPool2d(kernel_size=x, stride=1, padding=x // 2, ceil_mode = True) for x in k])

### 修改处4 modes/yolov5s_
+   ~~[-1, 1, nn.Upsample, [None, 2, 'nearest']],~~

+   添加 [-1, 1, nn.ConvTranspose2d, [256,256, 2, 2]],

### 修改处5, models/yolov5s.yaml
+   ~~[-1, 1, nn.Upsample, [None, 2, 'nearest']],~~

+   添加 [-1, 1, nn.ConvTranspose2d, [128 ,128, 2, 2]],

### 转换ONNX修改： models/export.py , 修改了opset_version
+    torch.onnx.export(model, img, f, verbose=False, opset_version=10, input_names=['images']...

### 参考 https://github.com/Wulingtian/yolov5_caffe
+    将ONNX 转为caffe模型。

### 最后caffe模型中的premute和reshape层，nnie不支持，去掉premute层(一共有三层)reshape改为需要的输出。

# yolov5学习
    yolov5关键idea的总结

## 数据增强
### mosaic数据增强
    对多张图进行融合，输出目标输入两倍大小的图片
+ 定义输入尺寸width、height，假定width == height
+ 计算最小scale，将图像缩放，将长边缩放到width
+ 定义一个2倍大小的空图像，用来存放masaic的结果
+ 随机选择四张图拼接的中心点，中心点的选择在$\frac{width}{2}$ ~ $\frac{3}{2}width$之间
+ 第一张图，中心点为右下点，中心点往左减去图片的宽高，与0进行比较，得到第一张图的左上点，然后计算需要截取的宽高，从第一张图的右下截取
+ 第二张图，中心点为左上点，$x_c + w$与$2w$比较，$y_c - h$与$0$比较，截断，得到宽高，从第二张图的左下截取
+ 第三张图，中心点为右上点，$x_c - w$与$0$比较，$y_c + h$与$2h$比较，截断，从第三张图的右上截取
+ 第四张图，中心点为左下点，$x_c + w$与$2w$比较，$y_c + h$与$2h$比较，截断，从第四张图的左下截取
#### 仿射变换
    对融合后的图片进行平移，仿射，旋转，缩放，剪切变换

+ 平移 
    - $x$和$y$左移$width$ $height$
+ 透射
    - 没有使用
+ 旋转
    - 角度设为0，旋转中心$(0, 0)$，scale在0.5~1.5直接随机选择
    
      $x_1 = x_c + \cos(\theta)(x_0 - x_c) - \sin(\theta)(y_0 - y_c)$
      
      $y_1 = y_c + \sin(\theta)(x_0 - x_c) + \cos(\theta)(y_0 - y_c)$
    
        加入scale，scale是乘在以旋转中心为起点，旋转点为终点的向量上，也就是半径上
    
      $x_1 = x_c + s*\cos(\theta)(x_0 - x_c) - s*\sin(\theta)(y_0 - y_c)$

      $y_1 = y_c + s*\sin(\theta)(x_0 - x_c) + s*\cos(\theta)(y_0 - y_c)$

      化简得旋转矩阵为：

      $$s*\cos(\theta)  \quad  -s*\sin(\theta) \quad  (1 - s*\cos(\theta))*x_c + s*\sin(\theta)*y_c \\
       s*\sin(\theta) \quad s*\cos(\theta) \quad  (1 - s*\cos(\theta))*y_c - s*\sin(\theta)x_c
      $$
+ 切变
    - 切变角度为90，未使用
+ 平移
    - 向右平移$0.4 * width$ ~ $0.6 * width$之间的随机值

+ 所有仿射、透射变换矩阵相乘
    - 如果用了透射就用warpperspective，仿射后的矩阵宽高为目标宽高

#### 总结
    多个仿射变换做的事主要还是基于mosaic后的图的中心区域进行裁剪，附带一些放缩以增强小目标检测，在增加数据多样性的同时可以让大目标不会变少，也能增强小目标，scale在0.5-1.5之间，也会让增强的目标在真实区间内波动，可以覆盖真实图片中目标的尺寸范围

## label分配

    相对于yolov3和yolov4，取gt坐标左上点计算锚点iou为正样本，yolov5做了改进

### gt分配思想

    设gt坐标的中心点的值(x, y)，(floor(x), floor(y))必取，若x的小数位小于0.5，则取(floor(x - 1), floor(y))，若x的小数位大于等于0.5，则取(floor(x + 1), floor(y))；同理若y的小数位小于0.5，则取(floor(x), floor(y - 1))，若y的小数位大于等于0，则取(floor(x), floor(y + 1))

    回归边框，gt与anchor的比例设置小于4

### 预测

    根据label的设计，回归的中心点用了$sigmoid$，然后乘以2 - 0.5，值域在-0.5-1.5之间；回归的边框也用了$sigmoid$，然后乘以2再乘以2，再乘anchor的宽高

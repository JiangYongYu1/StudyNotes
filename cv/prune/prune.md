# prune(剪枝)
    剪枝的一些思想总结

## 剪枝步骤
### 基于bn权重的剪枝
    1. 统计conv后面接了bn的op，这是需要裁剪的op
    2. 对这些的op的bn的$\alpha$进行统计，得到最小的最大值，
       然后的得到对应的裁剪率，避免将整个op都裁剪了
    3. 选择一个小于最大裁剪率的裁剪比例进行裁剪
        a. 先得到保留的channel的mask
        b. 然后根据conv的计算公式，输出的每个channel都是输入
           的channel与conv点乘+加，裁剪的channel只是\alpha值
           很小，可以计为0，但是\beta还是有值，如果把其也去掉，
           会导致精度下降特别明显，所以需要将\beta的值经过
           leakrule后通过卷积计算然后加入到下一个op的计算，
           a1. conv + bn，则可以在bn的running_mean上减去
               这个值，因为带了bn的conv一般不带bias
           a2. conv，则在bias上加上这个值
        c. 重新构建网络结构
        d. 赋值
        e. finetune
# OPEN_Nc_Power
MCU+OPA 控制DC-DC电源



### 引言

MCU：单片机（本设计需要输出PWM波、通信、ADC）

OPA: [运放](https://baike.baidu.com/item/运放/1527937)

DC-DC：[直流的电压转换器](https://baike.baidu.com/item/DC-DC?fromModule=lemma_search-box)

(图片看不清可以点击放大！！)



____

### DC-DC

​	**众所周知**，DC-DC分为两类，异步DC-DC、同步DC-DC他们一个相对便宜，一个相对贵（不是重点）。但是他们都是以负反馈的方式来控制**输出电压**（当然低于输入电压，高于参考电压），因为DC-DC的IC内部集成了一个比较器（OPA的一种运用）。

![image-20220907174252477](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907174252477.png)

​	说人话就是，FB是一个小人，R1和R2组成分压电路，FB读到的电压是 Vout*（R2 / (R1+ R2)）。如果（小人）FB发现电压低于自己的参考值（ Reference）就会想办法拉高Vout电压（因为根据电路原理，Vout上升之后FB的电压也就上升了）；反之，则会拉低Vout电压——这样就可以达到调压的目的。

​	**如果**，在R1和R2确定的情况下，我们再加入一个电阻，给这个电阻设定不同的电压，是否就能影响这个小人的判断呢？答案是肯定的！理论存在，实践开始！

![image-20220907192201673](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907192201673.png)

​	这就是我们想要的操控FB的电路。很明显这里的阻值需要去计算合适的值，需要满足以下条件：

1. 当Vx=3.3V时，Vy=1.2V
2. 当Vx=0.0V时，Vy=18V     （相当于r1并r2）
3. U = I*R
4. I(r2) = I(r1) + I(A)

对此建立数学模型：

①当Vx = 3.3V时：

​	此时Vy完全等于FB，所以没有电流，不参与此模型

​	r2 / (r1 + r2) * 3.3 = 1.2V	

②当Vx = 0 V时：

​	首先，0v可以看出接地，那么就是r1和r2并联，等效成电阻R（B），且Vy要达到最大值18V。

​	R（B） / (R（A） + R（B）) * 18 = 1.2V



**所以**！不难得出一组数据(这是有些误差的)

​	R(A) = 18k

​	R(B) = 1.27273k	-> (R1 = 3.5k，R2 = 2k)

​	不接R2，Vout = 12.831 V

这些电阻精度越高越好！ 0.5% - 1%





拓展：根据上面的描述，我们就能知道DC-DC几个重要的参数

1. FB的值（不一定叫FB，就是Reference）
2. 输入电压区间
3. 输出电压区间
4. 频率、是否有自保护机制、电流大小、转换效率、布线参考、延迟启动...

______

### MCU

你以为这就结束了？NO NO NO

便宜的单片机哪来的（0-3.3）V电压（DAC）给你编程呢？我们需要用PWM做一个DAC！

那么问题来了，如何把PWM变成DAC呢？

很简单，滤波！

为了最求便宜，我们采用RC滤波，利用10KHz以上的PWM通过RC滤波电路之后会变成一条直线的原理（如果一阶RC滤波不行，那就二阶RC滤波！），将PWM变成DAC！

![image-20220907194309229](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907194309229.png)



____

### OPA

这就结束了吗？显然没有的。

因为这个电路到输出端，串联了一个**10K的电阻**，这就是相当于这个输出的OUT内阻很大！

有什么办法减少内阻呢？换句话说，我们需要一个跟它电压一样，但是输出内阻小的东西，在电路中符合这个东西的运用就是——电压跟随器。

​	电压跟随器的特点就是，输入阻抗很大！输出阻抗小！正好我们输出阻抗大，给到跟随器的输入（OPA不嫌弃你的输出阻抗大）。

​	所以，我们现在需要用OPA（运放）搭建一个电压跟随器（所谓电压跟随不就是**把原电压放大一倍**吗？）

​	**放大电路**如下：
$$
β = 1 + R1 / (R1 + R2)
$$
![image-20220907203424086](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907203424086.png)



那么根据公式，放大一倍的话(**β = 1**)，R1要**很大**，R2要一个小的值。

![image-20220907203747901](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907203747901.png)

β = 1 + 1 / (1 + 100)

β = 1.0099

这样就大功告成了！

_______

### END

最终是这个样子的

![image-20220907204254379](https://raw.githubusercontent.com/SwiperWitty/img/main/img/image-20220907204254379.png)



我叫卡文迪许! 

我在这里 >>>  [GitHub](https://github.com/SwiperWitty/OPEN_Nc_Power)  [bilibili](https://space.bilibili.com/102898291)


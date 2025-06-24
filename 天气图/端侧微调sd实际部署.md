使用c站上的1.5模型，直接替换掉unet其余不变的情况下进行生成，效果并不好：
`1girl, makima, braided ponytail, ringed eyes, collared shirt, black necktie, black pants, red hair, yellow eyes, <lora:csm_makima:1>`
预期：
![](Pasted%20image%2020250620111523.png)
实际：
![](1ee3ea7a-68c3-46c3-aa9b-89c99b8db5b8.png)

主要原因是原模型使用的是DPM++ 2M Karras采样器，扩散了20步
我们这个模型使用lcm扩散了4步
手动把采样器换成dpm++ 扩散40步，效果有点改进：
![](d68f1e57-91ce-4694-b6d6-9dcbfb3058d2.png)




目前尝试的内容：
1、demo降分辨率跑通：sd1.5 + lcm-lora
剩下的方式跑c站模型全部不通：

1、sd1.5 + lora + dpm++采样器
2、sd1.5-lcm-safetensor


主要是下面两种情况的模型：
1、https://civitai.com/models/16014/anime-lineart-manga-like-style  这种基础sd1.5的lora，但是使用的是dpm++采样器，我按照lcm的方式生成固定的时间嵌入，生成对应的矫正集，推理还是异常的

2、https://civitai.com/models/71523/the-wondermix  一整个sd1.5-lcm-lora 的safetensor的加载和导出onnx，我直接替换基础sd1.5的权重导出来是不行的，或许应该用配置文件

c站上没有单独给你lcm-lora部分，像demo那样加载在basemodel上的..还有转模型实在是太慢了，一整天都能浪费完，稍微改个参数也不知道有没有影响，转一下就是n个小时，下面得开始搭建onnx推理pipeline，要不然都不知道是onnx的问题还是ax的问题


最终：
模型https://civitai.com/models/24779?modelVersionId=93208

| 卖家秀                                      | 买家秀                          |
| ---------------------------------------- | ---------------------------- |
| ![](企业微信截图_17507500681348.png)           | ![](txt2img_output_onnx.png) |
| ![](Pasted%20image%2020250624153444.png) |                              |


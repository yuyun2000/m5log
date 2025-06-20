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


```
huggingface-cli download --resume-download stable-diffusion-v1-5/stable-diffusion-v1-5 --local-dir stable-diffusion-v1-5/stable-diffusion-v1-5
```

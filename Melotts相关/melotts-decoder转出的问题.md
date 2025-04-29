运行命令：

pulsar2 build --input decoder-zh.onnx --config config_decoder_u16.json --output_dir decoder --output_name decoder-zh.axmodel --target_hardware AX620E 


原始量化，到最后两层相似度才变得很低：

![](../file/Pasted%20image%2020250428112216.png)


把leakyrelu后的层变为fp32，配置文件夹中添加：
```
     {
      "layer_name": "/dec/LeakyRelu_5",
      "data_type": "FP32"
    },
    {
      "layer_name": "/dec/conv_post/Conv",
      "data_type": "FP32"
    },
```
转出来还是这样：
![](../file/Pasted%20image%2020250428112319.png)

可以看到在relu前多了一个反量化的linear，从U16变成FP32，但是之后再过relu相似度并没有提升
并且后续步骤直接报错，版本号为：
```
version: 3.3
commit: f0b32d03
```
接下来使用：
```
version: 3.4
commit: 983bb35e
```
依然报错：
```C++
new_ddr_tensor = []
build op serially... ⠋ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╸━━ 7726/7940 0:11:28
Traceback (most recent call last):
  File "<frozen backend.ax620e.backend_impl>", line 106, in build
  File "<frozen opset.oprdef>", line 209, in build
  File "<frozen opset.reference.activation_builder>", line 172, in activation_builder
  File "<frozen opset.graph_builder>", line 162, in oprfunc
  File "<frozen opset.graph_builder>", line 49, in op
  File "<frozen opset.oprdef>", line 420, in GetOpr
KeyError: 'dont support lut_float opr in AXOPS'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<frozen yamain.common.error>", line 59, in guard_context
  File "<frozen yamain.command.build>", line 1281, in compile_ptq_model
  File "<frozen yamain.command.npu_backend_compiler>", line 231, in compile
  File "<frozen yasched.test_onepass>", line 3364, in test_onepass_ir
  File "<frozen yasched.test_onepass>", line 3348, in build
  File "<frozen yasched.test_onepass>", line 3491, in graph2models
  File "<frozen yasched.test_onepass>", line 1339, in graph2results
  File "<frozen yasched.test_onepass>", line 975, in build_op
  File "<frozen backend.ax620e.backend_impl>", line 117, in build
backend.base.OpBuildException: op: AxLeakyRelu, 'dont support lut_float opr in AXOPS'
        attrs = {'const_inputs': {}, 'negative_slope': 0.009999999776482582, 'output_dtype': 'FP32', 'prefix': '/dec/LeakyRelu_5_1129_s0_gop_extra'}
        input = {'x': Tensor(FP32, name=/dec/Div_4_output_0[:1,:16,:4096]1129_s0_gop, shape=(1, 16, 4096), offset=0, bit_strides=(2097152, 131072, 32)}
        output = {'r': Tensor(FP32, name=/dec/LeakyRelu_5_output_0[:1,:16,:4096]1129_s0_gop, shape=(1, 16, 4096), offset=0, bit_strides=(2097152, 131072, 32)}

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "</opt/pulsar2/yamain/main.py>", line 4, in <module>
  File "<frozen yamain.main>", line 342, in <module>
  File "<frozen yamain.main>", line 270, in pulsar2
  File "<frozen yamain.main>", line 158, in wrapper
  File "<frozen yamain.common.error>", line 23, in wrapper
  File "<frozen yamain.command.build>", line 111, in build_error
  File "<frozen yamain.common.error>", line 14, in wrapper
  File "<frozen yamain.command.build>", line 560, in build
  File "<frozen yamain.command.build>", line 1284, in compile_ptq_model
  File "/usr/local/lib/python3.9/contextlib.py", line 137, in __exit__
    self.gen.throw(typ, value, traceback)
  File "<frozen yamain.common.error>", line 61, in guard_context
  File "<frozen yamain.common.error>", line 73, in error_func
yamain.common.error.CodeException: (<ErrorCode.NPUBackendError: 8>, OpBuildException("op: AxLeakyRelu, 'dont support lut_float opr in AXOPS'\n\tattrs = {'const_inputs': {}, 'negative_slope': 0.009999999776482582, 'output_dtype': 'FP32', 'prefix': '/dec/LeakyRelu_5_1129_s0_gop_extra'}\n\tinput = {'x': Tensor(FP32, name=/dec/Div_4_output_0[:1,:16,:4096]1129_s0_gop, shape=(1, 16, 4096), offset=0, bit_strides=(2097152, 131072, 32)}\n\toutput = {'r': Tensor(FP32, name=/dec/LeakyRelu_5_output_0[:1,:16,:4096]1129_s0_gop, shape=(1, 16, 4096), offset=0, bit_strides=(2097152, 131072, 32)}"))

```
把混合量化的那几层注释掉后，可以正常导出，但是精度有问题

手动把末尾几层去掉：

转换时精度都变成了0.99

有什么办法可以把末尾几层加上并且以fp32导出？






让开发板也能跑图！**AX8850部署SD1.5全流程：从C站选模到板端推理**

---

### 正文内容：

大家好，今天给大家带来在 **AX8850** 平台上部署 **Stable Diffusion 1.5** 的全流程技术分享。

想要在边缘端跑出高质量的图，模型的转换和部署是关键。本项目代码已开源，欢迎Star！
👉 **代码仓库**：`https://github.com/yuyun2000/sd15-to-ax8850-deploy`

下面直接上干货，教大家如何从零开始把一个 Safetensor 权重转换成板端可用的 axmodel。

---

### 第一步：模型选型（C站）
首先去 Civitai 挑选底模。
**选模标准（敲黑板）：**
1.  **Base Model**：必须是 SD 1.5。
2.  **类型**：完整的 Checkpoint（Lora的教程之后再填坑）。
3.  **大小**：最好是标准的 **1.99GB** 左右（稍微大一点通常也行，可以自行测试）。

比如我选用了这个 `xxmix9realistic` 模型：
`https://civitai.com/models/47274/xxmix9realistic`
下载好对应的 `.safetensors` 权重文件备用。

---

### 第二步：PC端验证与导出 ONNX
在开始转换前，先在PC环境跑通流程，确保模型本身没问题。

**1. 安装依赖并验证 Safetensor**
运行 `safetensor_infer.py`，输入你的 prompt 和 negative prompt。
这一步是为了确认下载的权重能正常出图（虽然PC端效果和最终板端可能因分辨率、采样器不同有细微差异，但风格应一致）。

**2. 导出 ONNX 模型**
运行核心脚本：
```bash
python export_onnx.py --input_path ./s/xxmix9realistic_v40.safetensors --output_path ./out_onnx --isize 480x320 --prompt "你的提示词..."
```
**⚠️ 注意事项：**
*   **分辨率设置**：`--isize` 可以是 `480x320`（竖图）、`320x480`（横图）或 `512x512`。
*   **尺寸限制**：建议不要超过 512，且尺寸必须能被8整除（因为后面VAE缩放需要）。
*   **关于 Text Encoder**：脚本**不会**导出文本编码器。原因有两个：一是环境配置有点麻烦；二是Text Encoder通常都是标准的CLIP，直接用通用的即可（后面会给下载链接）。

**3. (可选) ONNX 推理验证**
运行 `python dpm_infer.py` 验证导出的 ONNX 是否能跑通，这一步可以跳过，仅作排查用。

---

### 第三步：准备量化校准集
模型转换需要量化，量化需要校准数据。
直接运行：
```bash
python prepare_data.py
```
脚本会在目录下生成 `calib_data_vae` 和 `calib_data_unet`。
*Tip：不同的模型风格不同，建议根据你选的模型，在代码里微调一下生成校准数据的 Prompt，效果会更好。*

---

### 第四步：Pulsar 模型转换（耗时预警 ☕）
接下来需要切换到 **Pulsar 工具链环境**（推荐版本 4.2）。

**1. 转换 UNet（重头戏）**
```bash
pulsar2 build --input out_onnx/unet_sim_cut.onnx --config config_unet_u16.json --output_dir output_unet_ax --output_name unet.axmodel
```
🛑 **高能预警**：这一步非常慢！实测耗时约 **68分钟**，建议跑起来去喝杯咖啡或者打局游戏。

**2. 转换 VAE**
```bash
pulsar2 build --input out_onnx/vae_decoder_sim.onnx --config config_vae_decoder_u16.json --output_dir output_vae_ax --output_name vae.axmodel
```
VAE 转换相对较快。

---

### 第五步：板端部署
转换完成后，你现在手头有了 `unet.axmodel` 和 `vae.axmodel`。
还缺一个 Text Encoder 对吧？
👉 **Text Encoder 下载**：`https://huggingface.co/yunyu1258/SD1.5-AX650-Dark_Sushi_Mix`
直接去 HuggingFace 下载这个现成的 axmodel 即可。

最后，使用仓库中提供的板端推理 Python 脚本，把三个模型加载进去，就可以在 AX8850 上愉快地跑图啦！

---

### 📝 总结
整个流程概括为：**C站选模 -> PC验证 -> 导出ONNX -> 生成校准集 -> Pulsar转换 -> 板端运行**。

如果你在部署过程中遇到问题，或者有成功的返图，欢迎在评论区交流！觉得有用的话，别忘了**点赞、收藏、投币**三连支持一下！

#AI #嵌入式 #AX8850 #StableDiffusion #模型部署 #深度学习
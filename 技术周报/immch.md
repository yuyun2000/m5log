# 在树莓派5上跑通Immich智能相册：搭配M5Stack AXera-Pi实现本地AI搜图

最近在树莓派5上部署了Immich这个开源相册系统，配合M5Stack的AXera-Pi加速卡，实现了**完全本地化的AI图片搜索**。实测下来，中文搜索"戴眼镜的人"、"蓝天白云"这类描述都能准确找到对应照片，响应速度也挺快。

整个部署过程其实不复杂，主要是docker容器 + Python虚拟环境的组合。下面记录下完整流程，给想搭建私有相册的朋友一个参考。

---

## 为什么选择Immich + AXera-Pi

**Immich**本身是个很成熟的开源相册方案，支持自动备份、人脸识别、地理位置等功能。但它的AI能力通常需要跑在服务器上，对硬件要求不低。

**M5Stack的AXera-Pi加速卡**正好解决了这个问题——它内置的AX8850芯片专门做AI推理加速，插在树莓派5上后，可以直接跑CLIP这类视觉模型，实现**语义级别的图片搜索**（而不是简单的标签匹配）。

这意味着：
- 不需要把照片上传到云端
- 不需要昂贵的GPU服务器
- 搜索延迟低，隐私完全可控

---

## 部署流程

### 1️⃣ 下载模型和配置文件

有两种方式获取所需文件：

**方式一：手动下载**
从 [HuggingFace仓库](https://huggingface.co/AXERA-TECH/immich) 下载打包好的文件，上传到树莓派。

**方式二：Git拉取**（推荐）
```bash
# 确保已安装git lfs（处理大文件必需）
git clone https://huggingface.co/AXERA-TECH/immich
```

下载完成后可以看到这些文件：
```bash
m5stack@raspberrypi:~/rsp/immich $ ls -lh
total 421M
drwxrwxr-x 2 m5stack m5stack 4.0K Oct 10 09:12 asset
-rw-rw-r-- 1 m5stack m5stack 421M Oct 10 09:20 ax-immich-server-aarch64.tar.gz  # docker镜像
-rw-rw-r-- 1 m5stack m5stack 104K Oct 10 09:12 immich_ml-1.129.0-py3-none-any.whl  # ML服务预编译包
-rw-rw-r-- 1 m5stack m5stack 7.6K Oct 10 09:12 docker-deploy.zip  # 部署配置
-rw-rw-r-- 1 m5stack m5stack  177 Oct 10 09:12 requirements.txt
```

> ⚠️ 如果之前没装过**docker**和**git lfs**，需要先完成这两个前置工作。

---

### 2️⃣ 启动Immich主服务

**导入docker镜像**
```bash
cd immich
docker load -i ax-immich-server-aarch64.tar.gz
```

**准备配置文件**
```bash
unzip docker-deploy.zip
cp example.env .env  # 复制环境配置模板
```

**启动容器组**
```bash
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

成功后会看到三个容器启动：
```bash
[+] Running 3/3
 ✔ Container immich_postgres  Started   # 数据库
 ✔ Container immich_redis     Started   # 缓存
 ✔ Container immich_server    Started   # 主服务
```

这一步完成后，Immich的基础服务就跑起来了，但AI搜索功能还需要单独配置机器学习模块。

---

### 3️⃣ 部署机器学习服务

为了隔离Python依赖，用虚拟环境来管理：

**创建并激活虚拟环境**
```bash
python -m venv mich
source mich/bin/activate
```

**安装依赖包**
```bash
# 安装AXera-Pi的推理引擎
pip install https://github.com/AXERA-TECH/pyaxengine/releases/download/0.1.3.rc2/axengine-0.1.3-py3-none-any.whl

# 安装Immich ML的依赖
pip install -r requirements.txt

# 安装预编译的ML服务包（文件名以实际为准）
pip install immich_ml-1.129.0-py3-none-any.whl
```

**启动ML服务**
```bash
python -m immich_ml
```

正常运行后会看到：
```bash
[INFO] Available providers:  ['AXCLRTExecutionProvider']  # 确认AXera加速器被识别
[10/10/25 09:50:16] INFO     Started server process [8699]
[10/10/25 09:50:16] INFO     Application startup complete.
```

这里的关键是看到**AXCLRTExecutionProvider**这个提示，说明**M5Stack的AXera-Pi加速卡已经被正确调用**，AI推理会走硬件加速路径，而不是纯CPU运算。

---

## 配置和使用

### 初次访问设置

在浏览器输入 `http://树莓派IP:3003`（例如`http://192.168.20.27:3003`）

**第一步：创建管理员账户**
![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich1.png)
⚠️ 这个账号信息只存在本地，不会上传到任何地方，务必记住密码。

**第二步：上传照片**
![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich2.png)

---

### 配置AI搜索功能

进入 **管理员设置 → Machine Learning**：
![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich3.png)

关键配置项：
- **ML服务地址**：填写`http://树莓派IP:3003`
- **CLIP模型选择**：
  - **中文搜索** → `ViT-L-14-336-CN__axera`
  - **英文搜索** → `ViT-L-14-336__axera`

![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich4.png)

保存后，进入 **Jobs → SMART SEARCH**，手动触发一次索引构建：
![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich6.png)

这个过程会用**AXera-Pi加速卡**对已上传的照片进行特征提取，第一次会花点时间，后续新增照片会自动处理。

---

### 实际体验

索引完成后，直接在搜索框输入描述性文字：
![](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/linux/ax8850_card/images/immich7.png)

实测下来，像"海边"、"笑脸"、"建筑物"这类词汇识别率都很高，比传统的文件名匹配实用太多。

---

## 技术细节补充

**为什么要单独跑ML服务？**
Immich的架构是主服务 + ML服务分离的，主服务负责照片管理、用户认证等，ML服务专门处理AI推理。这样的好处是：
- ML服务可以部署在有加速卡的设备上
- 可以独立扩展或更新模型
- 主服务故障不影响AI功能

**AXera-Pi加速的实际效果**
在树莓派5 + AXera-Pi的组合下，处理一张照片的特征提取大概在100-200ms左右（具体看分辨率）。如果纯CPU跑，这个时间至少要翻3-5倍。对于相册场景来说，批量处理几百张照片时差异会很明显。

**模型选择建议**
中文模型（ViT-L-14-336-CN）是在中文语料上微调的，对中文描述的理解更准确。如果你主要用中文搜索，建议直接选这个。英文模型在英文语义上表现更好，可以根据实际使用场景切换。

---

## 写在最后

整个部署流程走下来，最大的感受是**M5Stack的AXera-Pi确实降低了本地AI应用的门槛**。以前要在树莓派上跑视觉模型，要么速度慢得难以实用,要么就得外挂一块英伟达的Jetson，成本直接翻倍。

现在有了专用的AI加速卡，在保持树莓派生态便利性的同时，性能也能满足日常使用。对于想搭建智能家居、本地AI服务的玩家来说，这套方案性价比还是挺高的。

有问题欢迎评论区讨论 💬
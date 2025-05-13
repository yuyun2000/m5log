## 1、准备支持 SSE 传输的客户端
**推荐客户端：**
- [Cherry Studio](https://www.cherry-ai.com/)（首选）
- Claude Desktop
- 其他支持 SSE 传输的 Claude 客户端

## 2、配置客户端与模型设置
以 Cherry Studio 为例：
1. 打开设置
2. 选择任意模型供应商
3. 输入相应的 API Key
4. 添加模型（支持 Claude、Deepseek、GPT 等）
![](../file/Pasted%20image%2020250513145452.png)

## 3、加载mcp服务
服务地址：`http://47.113.125.164:5056/sse`
还是到设置-MCP服务器
随便写一个名称，类型选sse，地址如上
![](../file/Pasted%20image%2020250513150315.png)
如果加载正常，会在工具栏显示可用的工具：
![](../file/Pasted%20image%2020250513150405.png)


## 4、聊天界面开启MCP 开始对话
开启后模型就会自动选择是否调用mcp工具，速度还是比较快的
![](../file/Pasted%20image%2020250513150602.png)
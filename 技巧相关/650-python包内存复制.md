

```python
        # import mmap

        # if isinstance(path_or_bytes, (str, os.PathLike)):
        #     self._model_name = os.path.splitext(os.path.basename(path_or_bytes))[0]
        #     with open(path_or_bytes, "rb") as f:
        #         # 使用内存映射，不实际读入内存
        #         mmapped_file = mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ)
        #         self._model_buffer = engine_cffi.from_buffer("char[]", mmapped_file)
        #         self._model_buffer_size = len(mmapped_file)
        #         self._mmapped_file = mmapped_file  # 保持引用

  
        # model buffer, almost copied from onnx runtime
        if isinstance(path_or_bytes, (str, os.PathLike)):
            self._model_name = os.path.splitext(os.path.basename(path_or_bytes))[0]
            with open(path_or_bytes, "rb") as f:
                data = f.read()
            self._model_buffer = engine_cffi.new("char[]", data)
            self._model_buffer_size = len(data)
        elif isinstance(path_or_bytes, bytes):
            self._model_buffer = engine_cffi.new("char[]", path_or_bytes)
            self._model_buffer_size = len(path_or_bytes)
        else:
            raise TypeError(f"Unable to load model from type '{type(path_or_bytes)}'")
```
系统原本内存占用为1.35G
![](../file/Pasted%20image%2020251030161719.png)

图中是 "`_axe.py`" 中加载模型的片段，加载完模型后内存占用
![](../file/Pasted%20image%2020251030161448.png)

如果使用注释段的代码替换第一个if，重新加载占用
![](../file/Pasted%20image%2020251030161355.png)
节省了0.23g，模型文件大小250MB，差不多对的上


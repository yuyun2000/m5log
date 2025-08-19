对比项目：
端侧效果 和 sherpa-onnx推理结果
对比内容：
commonvoice cv-corpus-22.0-delta-2025-06-20 en_other.tsv

具体对比方法：
1、sherpa-onnx使用全语种模式，所以识别的结果可能包含其他语种：https://k2-fsa.github.io/sherpa/onnx/sense-voice/python-api.html#decode-a-file
2、ax指定英语模式，只会输出英语的结果：https://huggingface.co/AXERA-TECH/SenseVoice
3、均转为16khz的wav后进行识别
4、指标计算也许和其他仓库不一致，但横向对比结果已经非常明显


sherpa的结果：
处理的音频文件数: 4076
整体词错误率 (WER): 0.3076 (30.76%)
  替换错误率: 0.2086 (20.86%)
  删除错误率: 0.0719 (7.19%)
  插入错误率: 0.0270 (2.70%)

ax的结果：
处理的音频文件数: 5815
整体词错误率 (WER): 0.5113 (51.13%)
  替换错误率: 0.3291 (32.91%)
  删除错误率: 0.1157 (11.57%)
  插入错误率: 0.0665 (6.65%)


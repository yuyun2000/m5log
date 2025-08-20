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
处理的音频文件数: 25859
整体词错误率 (WER): 0.2400 (24.00%)
  替换错误率: 0.1660 (16.60%)
  删除错误率: 0.0507 (5.07%)
  插入错误率: 0.0233 (2.33%)

ax的结果：
处理的音频文件数: 25859
整体词错误率 (WER): 0.4489 (44.89%)
  替换错误率: 0.2882 (28.82%)
  删除错误率: 0.0905 (9.05%)
  插入错误率: 0.0702 (7.02%)


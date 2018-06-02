

## Behavioral Cloning

1. Run simulator
2. Collect Data

自己构建的模型，算不出来，内存不够
从 keras 提供的 applications 中拿到 InceptionV3，分别导入模型和 weights （命令行无法从 github 直接把模型下载，因此分别导入）。
使用自己的数据，总是偏向一边，然后跑着出了轨道。

在前面加入 Lambda 层正则处理，以及使用了 Crop 裁剪掉无用的信息，效果依旧有限。

pip install flask-socketio==1.0b1

InceptionV3

0 Input

278 activation_85
279 mixed9
280 conv2d_90
281 batch_normalization_90
282 activation_90
283 conv2d_87
284 conv2d_91
285 batch_normalization_87
286 batch_normalization_91
287 activation_87
288 activation_91
289 conv2d_88
290 conv2d_89
291 conv2d_92
292 conv2d_93
293 average_pooling2d_9
294 conv2d_86
295 batch_normalization_88
296 batch_normalization_89
297 batch_normalization_92
298 batch_normalization_93
299 conv2d_94
300 batch_normalization_86
301 activation_88
302 activation_89
303 activation_92
304 activation_93
305 batch_normalization_94
306 activation_86
307 mixed9_1
308 concatenate_2
309 activation_94
310 mixed10
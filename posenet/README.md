# 浏览器中的姿势检测：PoseNet模型

该包包含一个名为PoseNet的独立模型，以及一些演示，用于使用TensorFlow.js在浏览器中运行实时姿态估计。

[在这里试试吧！](https://storage.googleapis.com/tfjs-models/demos/posenet/camera.html)

<img src="https://raw.githubusercontent.com/irealva/tfjs-models/master/posenet/demos/camera.gif" alt="cameraDemo" style="width: 600px;"/>

PoseNet可用于估计单个姿势或多个姿势，这意味着存在可以仅检测图像/视频中的一个人的算法版本以及可以检测图像/视频中的多个人的一个版本。

[请参阅此博客文章](https://medium.com/tensorflow/real-time-human-pose-estimation-in-the-browser-with-tensorflow-js-7dd0bc881cd5)有关在Tensorflow.js上运行的PoseNet的高级描述。

我们使用[tensorflow/tfjs](https://github.com/tensorflow/tfjs) Github仓库跟踪问题

## 安装

您可以将此作为独立的es5包使用，如下所示：

```html
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/posenet"></script>
```

或者您可以通过npm安装它以在TypeScript / ES6项目中使用。

```sh
npm install @tensorflow-models/posenet
```

## 用法

可以从图像预测单个姿势或多个姿势。
每种方法都有自己的算法和参数集。

### 关键点

所有关键点都按部件ID索引。 组件及其ID是：

| Id | Part |
| -- | -- |
| 0 | nose |
| 1 | leftEye |
| 2 | rightEye |
| 3 | leftEar |
| 4 | rightEar |
| 5 | leftShoulder |
| 6 | rightShoulder |
| 7 | leftElbow |
| 8 | rightElbow |
| 9 | leftWrist |
| 10 | rightWrist |
| 11 | leftHip |
| 12 | rightHip |
| 13 | leftKnee |
| 14 | rightKnee |
| 15 | leftAnkle |
| 16 | rightAnkle |

### 加载预先训练的PoseNet模型

在姿势预测的第一步中，图像是通过预先训练的模型输入的。 PoseNet **附带了几个不同版本的模型，**每个版本都对应一个具有特定乘数的MobileNet v1架构。 首先，必须从检查点加载模型，并使用乘数指定的MobileNet体系结构：

```javascript
const net = await posenet.load(multiplier);
```

#### 输入

* **乘数**  - 一个可选数字，其值为：`1.01`，`1.0`，`0.75`或`0.50`。 默认为`1.01`。 它是所有卷积运算的深度（通道数）的浮点乘数。 该值对应于MobileNet体系结构和检查点。 值越大，层的尺寸越大，并且以速度为代价的模型越精确。 将其设置为较小的值，以提高准确性为代价。

**默认情况下，**PoseNet加载一个带有乘数为 **`0.75`** 的模型。 对于具有**中档/低档GPUS**的计算机，建议使用此选项。对于具有功能**强大的GPUS**的计算机，建议使用带有乘数为`1.00`的型号。**带**的型号 建议**移动设备**使用乘数为`0.50` 架构。

### 单人姿态预测

单个姿态预测是两种算法中更简单和更快速的。 它的理想用例是图像中只有一个人。 缺点是如果图像中有多个人，那么来自两个人的关键点可能被估计为同一个单一姿势的一部分 - 意味着，例如，＃1的左臂和第2个人的右膝可能会混淆 通过算法属于相同的姿势。

```javascript
const net = await posenet.load();

const pose = await net.estimateSinglePose(image, imageScaleFactor, flipHorizontal, outputStride);
```

#### 输入

* **image**  -  ImageData | HTMLImageElement | HTMLCanvasElement | HTMLVideoElement
    输入图像通过网络提供。

* **imageScaleFactor** - 数字介于0.2和1.0之间。 默认为0.50。 在通过网络馈送图像之前缩放图像的内容。 将此数字设置得较低以缩小图像，并以牺牲精度为代价提高通过网络的速度。

* **flipHorizontal** - 默认为false。 如果姿势应该水平翻转/镜像。 对于视频默认水平翻转的视频（即网络摄像头），应将此设置为true，并且您希望以正确的方向返回姿势。

* **outputStride** - 通过模型输送图像时所需的输出步幅。 必须为32,16,8。默认为16.数字越大，性能越快但准确度越低，反之亦然。

#### 结果

它返回一个带有置信度得分的`pose`和一个由组件id索引的关键点数组，每个关键点都有一个得分和位置。

#### 用法示例

##### 浏览器环境直接引用脚本：

```html
<html>
  <head>
    <!-- Load TensorFlow.js -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <!-- Load Posenet -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/posenet"></script>
 </head>

  <body>
    <img id='cat' src='/images/cat.jpg '/>
  </body>
  <!-- Place your code in the script tag below. You can also use an external .js file -->
  <script>
    var imageScaleFactor = 0.5;
    var outputStride = 16;
    var flipHorizontal = false;

    var imageElement = document.getElementById('cat');

    posenet.load().then(function(net){
      return net.estimateSinglePose(imageElement, imageScaleFactor, flipHorizontal, outputStride)
    }).then(function(pose){
      console.log(pose);
    })
  </script>
</html>
```

###### 通过NPM引用：
```sh
npm install --save @tensorflow/tfjs
npm install --save @tensorflow-models/posenet
```

```javascript
import * as posenet from '@tensorflow-models/posenet';
const imageScaleFactor = 0.5;
const outputStride = 16;
const flipHorizontal = false;

async function estimatePoseOnImage(imageElement) {
  // load the posenet model from a checkpoint
  const net = await posenet.load();

  const pose = await net.estimateSinglePose(imageElement, imageScaleFactor, flipHorizontal, outputStride);

  return pose;
}

const imageElement = document.getElementById('cat');

const pose = estimatePoseOnImage(imageElement);

console.log(pose);

```

这将产生输出：

```json
{
  "score": 0.32371445304906,
  "keypoints": [
    {
      "position": {
        "y": 76.291801452637,
        "x": 253.36747741699
      },
      "part": "nose",
      "score": 0.99539834260941
    },
    {
      "position": {
        "y": 71.10383605957,
        "x": 253.54365539551
      },
      "part": "leftEye",
      "score": 0.98781454563141
    },
    {
      "position": {
        "y": 71.839515686035,
        "x": 246.00454711914
      },
      "part": "rightEye",
      "score": 0.99528175592422
    },
    {
      "position": {
        "y": 72.848854064941,
        "x": 263.08151245117
      },
      "part": "leftEar",
      "score": 0.84029853343964
    },
    {
      "position": {
        "y": 79.956565856934,
        "x": 234.26812744141
      },
      "part": "rightEar",
      "score": 0.92544466257095
    },
    {
      "position": {
        "y": 98.34538269043,
        "x": 399.64068603516
      },
      "part": "leftShoulder",
      "score": 0.99559044837952
    },
    {
      "position": {
        "y": 95.082359313965,
        "x": 458.21868896484
      },
      "part": "rightShoulder",
      "score": 0.99583911895752
    },
    {
      "position": {
        "y": 94.626205444336,
        "x": 163.94561767578
      },
      "part": "leftElbow",
      "score": 0.9518963098526
    },
    {
      "position": {
        "y": 150.2349395752,
        "x": 245.06030273438
      },
      "part": "rightElbow",
      "score": 0.98052614927292
    },
    {
      "position": {
        "y": 113.9603729248,
        "x": 393.19735717773
      },
      "part": "leftWrist",
      "score": 0.94009721279144
    },
    {
      "position": {
        "y": 186.47859191895,
        "x": 257.98034667969
      },
      "part": "rightWrist",
      "score": 0.98029226064682
    },
    {
      "position": {
        "y": 208.5266418457,
        "x": 284.46710205078
      },
      "part": "leftHip",
      "score": 0.97870296239853
    },
    {
      "position": {
        "y": 209.9910736084,
        "x": 243.31219482422
      },
      "part": "rightHip",
      "score": 0.97424703836441
    },
    {
      "position": {
        "y": 281.61965942383,
        "x": 310.93188476562
      },
      "part": "leftKnee",
      "score": 0.98368924856186
    },
    {
      "position": {
        "y": 282.80120849609,
        "x": 203.81164550781
      },
      "part": "rightKnee",
      "score": 0.96947449445724
    },
    {
      "position": {
        "y": 360.62716674805,
        "x": 292.21047973633
      },
      "part": "leftAnkle",
      "score": 0.8883239030838
    },
    {
      "position": {
        "y": 347.41177368164,
        "x": 203.88229370117
      },
      "part": "rightAnkle",
      "score": 0.8255187869072
    }
  ]
}
```

### 多人姿态预测

多姿态预测可以解码图像中的多个姿势。 它比单个姿势算法更复杂且稍慢，但具有如下优点：如果图像中出现多个人，则他们检测到的关键点不太可能与错误姿势相关联。 即使用例是检测单个人的姿势，该算法也可能是更理想的，因为当多个人出现在图像中时，两个姿势连接在一起的意外效果将不会发生。 它使用了研究论文中的`Fast greedy decoding`(快速贪婪解码)算法[PersonLab: Person Pose Estimation and Instance Segmentation with a Bottom-Up, Part-Based, Geometric Embedding Model](https://arxiv.org/pdf/1803.08225.pdf).

```javascript
const net = await posenet.load();

const poses = await net.estimateMultiplePoses(image, imageScaleFactor, flipHorizontal, outputStride, maxPoseDetections, scoreThreshold, nmsRadius);
```

#### 输入

* **image** - ImageData|HTMLImageElement|HTMLCanvasElement|HTMLVideoElement
  输入图像通过网络提供。
* **imageScaleFactor** - 数字介于0.2和1.0之间。 默认为0.50。 在通过网络馈送图像之前缩放图像的内容。 将此数字设置得较低以缩小图像，并以牺牲精度为代价提高通过网络的速度。

* **flipHorizontal** - 默认为false。 如果姿势应该水平翻转/镜像。 对于视频默认水平翻转的视频（即网络摄像头），应将此设置为true，并且您希望以正确的方向返回姿势。

* **outputStride** - 通过模型输送图像时所需的输出步幅。 必须为32,16,8。默认为16.数字越大，性能越快但准确度越低，反之亦然。

* **maxPoseDetections** (可选) - 要检测的最大姿势数。 默认为5。

* **scoreThreshold** (可选) - 仅返回根部分得分大于或等于此值的实例检测。 默认为0.5。

* **nmsRadius** (可选) - 非最大抑制部分距离。 它必须是严格积极的。 如果它们小于`nmsRadius`像素，则两个部分相互抑制。 默认为20。

#### 结果

它返回一个`promise`，用一系列`pose`解析，每个都有一个置信度分数和一个由组件id索引的`keypoints`数组，每个都有一个得分和位置。

##### 浏览器环境直接引用脚本：

```html
<html>
  <head>
    <!-- Load TensorFlow.js -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <!-- Load Posenet -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/posenet"></script>
 </head>

  <body>
    <img id='cat' src='/images/cat.jpg '/>
  </body>
  <!-- Place your code in the script tag below. You can also use an external .js file -->
  <script>
    var imageScaleFactor = 0.5;
    var flipHorizontal = false;
    var outputStride = 16;
    var maxPoseDetections = 2;

    var imageElement = document.getElementById('cat');

    posenet.load().then(function(net){
      return net.estimateMultiplePoses(imageElement, 0.5, flipHorizontal, outputStride, maxPoseDetections)
    }).then(function(poses){
      console.log(poses);
    })
  </script>
</html>
```

###### 通过NPM引用：
```sh
npm install --save @tensorflow/tfjs
npm install --save @tensorflow-models/posenet
```

```javascript
import * as posenet from '@tensorflow-models/posenet';

const imageScaleFactor = 0.5;
const outputStride = 16;
const flipHorizontal = false;
const maxPoseDetections = 2;

async function estimateMultiplePosesOnImage(imageElement) {
  const net = await posenet.load();

  // estimate poses
  const poses = await net.estimateMultiplePoses(imageElement,
    imageScaleFactor, flipHorizontal, outputStride, maxPoseDetections);

  return poses;
}

const imageElement = document.getElementById('people');

const poses = estimateMultiplePosesOnImage(imageElement);

console.log(poses);
```

这会产生输出：
```json
[
  // pose 1
  {
    // pose score
    "score": 0.42985695206067,
    "keypoints": [
      {
        "position": {
          "x": 126.09371757507,
          "y": 97.861720561981
        },
        "part": "nose",
        "score": 0.99710708856583
      },
      {
        "position": {
          "x": 132.53466176987,
          "y": 86.429876804352
        },
        "part": "leftEye",
        "score": 0.99919074773788
      },
      {
        "position": {
          "x": 100.85626316071,
          "y": 84.421931743622
        },
        "part": "rightEye",
        "score": 0.99851280450821
      },

      ...

      {
        "position": {
          "x": 72.665352582932,
          "y": 493.34189963341
        },
        "part": "rightAnkle",
        "score": 0.0028593824245036
      }
    ],
  },
  // pose 2
  {

    // pose score
    "score": 0.13461434583673,
    "keypoints": [
      {
        "position": {
          "x": 116.58444058895,
          "y": 99.772533416748
        },
        "part": "nose",
        "score": 0.0028593824245036
      }
      {
        "position": {
          "x": 133.49897611141,
          "y": 79.644590377808
        },
        "part": "leftEye",
        "score": 0.99919074773788
      },
      {
        "position": {
          "x": 100.85626316071,
          "y": 84.421931743622
        },
        "part": "rightEye",
        "score": 0.99851280450821
      },

      ...

      {
        "position": {
          "x": 72.665352582932,
          "y": 493.34189963341
        },
        "part": "rightAnkle",
        "score": 0.0028593824245036
      }
    ],
  },
  // pose 3
  {
    // pose score
    "score": 0.13461434583673,
    "keypoints": [
      {
        "position": {
          "x": 116.58444058895,
          "y": 99.772533416748
        },
        "part": "nose",
        "score": 0.0028593824245036
      }
      {
        "position": {
          "x": 133.49897611141,
          "y": 79.644590377808
        },
        "part": "leftEye",
        "score": 0.99919074773788
      },

      ...

      {
        "position": {
          "x": 59.334579706192,
          "y": 485.5936152935
        },
        "part": "rightAnkle",
        "score": 0.004110524430871
      }
    ]
  }
]
```

## 开发 Demo

有关如何运行演示的详细信息包含在`demos /`文件夹中。


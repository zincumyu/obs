# OpenCV 机器视觉招新突击复习笔记（Python · 4.x）

> 面向：明天就要笔试/面试/实操的初学者
> 目标：1～2 小时把「常用调用 + 参数含义 + 适用场景 + 易错点」过一遍
> 标记说明：**【必背】**= 明天大概率直接考、必须能默写 ｜ **【理解】**= 能说清原理和场景 ｜ **【了解】**= 听过就行，被问到能聊两句

---

## 目录

- [0. 1～2 小时速通路线](#0-12-小时速通路线)
- [1. 环境安装与验证](#1-环境安装与验证-必背)
- [2. 图像 IO 与像素操作](#2-图像-io-与像素操作-必背)
- [3. 绘图与几何变换](#3-绘图与几何变换-必背)
- [4. 颜色与预处理](#4-颜色与预处理-必背)
- [5. 边缘检测与形态学](#5-边缘检测与形态学-必背)
- [6. 轮廓分析](#6-轮廓分析-必背)
- [7. 视频与摄像头](#7-视频与摄像头-必背)
- [8. 特征与匹配](#8-特征与匹配-理解)
- [9. 目标检测入门](#9-目标检测入门-理解)
- [10. 机器视觉典型流程](#10-机器视觉典型流程-理解)
- [11. 招新高频问题直答](#11-招新高频问题直答-必背)
- [12. 常见坑合集](#12-常见坑合集-必背)
- [13. 综合小项目](#13-综合小项目完整代码)
- [14. 重点 API 速查表](#14-重点-api-速查表)
- [15. 10 个必会代码片段](#15-10-个必会代码片段)
- [16. 面试自测 20 问](#16-面试自测-20-问)
- [17. 明天考前 10 分钟复习清单](#17-明天考前-10-分钟复习清单)

---

## 0. 1～2 小时速通路线

| 时间 | 干什么 | 重点 |
|---|---|---|
| 0～10 min | 第 1、2 章 | 会装、会读图、会看 shape |
| 10～25 min | 第 3 章 | resize / 透视变换的**宽高顺序** |
| 25～45 min | 第 4、5 章 | 阈值、滤波、Canny、形态学（**最高频**） |
| 45～65 min | 第 6、7 章 | findContours 返回值、面积/外接矩形、摄像头循环模板 |
| 65～80 min | 第 8、9 章 | ORB/SIFT 区别、matchTemplate 缺点、Haar 人脸 |
| 80～95 min | 第 13 章综合项目 | 至少能手敲一遍颜色追踪 |
| 95～120 min | 第 14～17 章 | 速查表 + 自测 20 问 + 考前清单 |

**一句话心态**：招新考的不是数学，是「给你一个需求，你知道调哪个函数、参数怎么写、跑不通时先看哪里」。

---

## 1. 环境安装与验证 【必背】

**一句话概念**：OpenCV 的 Python 包叫 `opencv-python`，装完 `import cv2` 能打印版本号就算成功。

**核心 API / 命令**

```bash
# 基础版（够用）
pip install opencv-python numpy -i https://pypi.tuna.tsinghua.edu.cn/simple

# 需要 SIFT/SURF/额外算法（4.4 之前 SIFT 在这里）
pip install opencv-contrib-python

# 千万不要两个都装！会互相覆盖
```

**关键点**

| 项 | 说明 |
|---|---|
| 包名 vs 导入名 | 装 `opencv-python`，导入 `cv2`（不是 opencv） |
| 版本要求 | 4.x，`cv2.__version__` 打印 `4.x.x` |
| GUI 支持 | `opencv-python` 带 GUI；`opencv-python-headless` 无窗口，服务器上用 |
| 图像存哪 | 代码文件和图片放**同一目录**，或写**绝对路径** |

**最小可运行代码**

```python
import cv2
import numpy as np

print("OpenCV:", cv2.__version__)
print("NumPy :", np.__version__)

# 生成一张纯色图并显示，验证 GUI 正常
img = np.zeros((300, 400, 3), dtype=np.uint8)   # 高300 宽400 3通道 黑图
img[:] = (0, 0, 255)                            # BGR 红色
cv2.putText(img, "OpenCV OK", (60, 160),
            cv2.FONT_HERSHEY_SIMPLEX, 1.2, (255, 255, 255), 2)
cv2.imshow("test", img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

**常见坑**

- `ModuleNotFoundError: No module named 'cv2'` → 装错环境（PyCharm 的 venv 和命令行 python 不是一个）。
- 装了 `opencv-python` 又装 `opencv-contrib-python` → 两个包共用同一目录，会坏。卸载重装一个。
- 无显示器/远程 SSH → `imshow` 报错，此时用 `cv2.imwrite` 存图验证。
- 中文路径下 `imread` 静默失败（见第 12 章）。

---

## 2. 图像 IO 与像素操作 【必背】

### 2.1 imread / imshow / imwrite

**一句话概念**：`imread` 把图片读成 NumPy 数组，`imshow` 显示，`imwrite` 存盘。

**核心 API**

```python
img = cv2.imread(path, flags)      # 失败返回 None，不报错！
cv2.imshow(winname, img)
key = cv2.waitKey(delay)           # 毫秒，0=无限等待
cv2.destroyAllWindows()
ok = cv2.imwrite(path, img)        # 返回 True/False
```

**关键参数**

| 参数 | 取值 | 含义 |
|---|---|---|
| `flags` | `cv2.IMREAD_COLOR`(1，默认) | 读成 3 通道 **BGR**，忽略透明通道 |
| | `cv2.IMREAD_GRAYSCALE`(0) | 读成单通道灰度，`shape=(h,w)` |
| | `cv2.IMREAD_UNCHANGED`(-1) | 原样读，含 alpha，4 通道 |
| `waitKey` | `0` | 无限等待按键（看图用） |
| | `1` / `30` | 等 1ms / 30ms（视频循环用） |

**最小可运行代码**

```python
import cv2

img = cv2.imread("test.jpg")            # 默认彩色
if img is None:                          # ★ 必须判空
    raise SystemExit("读图失败：路径错 / 中文路径 / 文件不存在")

gray = cv2.imread("test.jpg", cv2.IMREAD_GRAYSCALE)

print(img.shape, img.dtype, img.size)   # (H, W, 3) uint8 H*W*3
print(gray.shape)                        # (H, W)

cv2.imshow("color", img)
cv2.imshow("gray", gray)
cv2.waitKey(0)
cv2.destroyAllWindows()

cv2.imwrite("out.jpg", gray)
cv2.imwrite("out_q95.jpg", img, [cv2.IMWRITE_JPEG_QUALITY, 95])
```

**常见坑**

- **`imread` 读不到不会崩，只会返回 `None`**，然后你在下一行 `img.shape` 才崩 → 永远先 `if img is None`。
- Windows 下 **中文/空格路径** 会失败（见 12 章解决方案）。
- `imwrite` 的**扩展名决定格式**，写 `.jpg` 就是 JPEG；忘记扩展名会失败。
- JPEG 有损，反复读写会糊；中间结果存 `.png`。

---

### 2.2 BGR 与 RGB 【必背·最高频】

**一句话概念**：OpenCV 默认 **BGR**，Matplotlib / PIL / 大部分标注工具是 **RGB**，混用会「红蓝互换」。

**核心 API**

```python
rgb  = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)   # OpenCV -> 显示/其他库
bgr  = cv2.cvtColor(rgb, cv2.COLOR_RGB2BGR)   # 反向
rgb2 = img[:, :, ::-1]                        # 更快的切片写法，等价
```

**最小可运行代码**

```python
import cv2, matplotlib.pyplot as plt

img = cv2.imread("test.jpg")
plt.subplot(121); plt.imshow(img); plt.title("BGR -> wrong color")
plt.subplot(122); plt.imshow(img[:, :, ::-1]); plt.title("RGB -> correct")
plt.show()
```

**常见坑**

- `cv2.imshow` 要 **BGR**，`plt.imshow` 要 **RGB**。给 plt 传 BGR 会出现「人脸变蓝」。
- 通道顺序是 `img[y, x]` 得到 `(B, G, R)`，写代码时 `img[y,x] = (255,0,0)` 是**蓝色**。
- 灰度图可以直接给 `plt.imshow(..., cmap='gray')`，不用转。

---

### 2.3 shape / 像素 / ROI / 通道 【必背】

**一句话概念**：图像就是 NumPy 三维数组，索引顺序是 **[行 y, 列 x, 通道 c]**，切片规则和 NumPy 完全一样。

**核心 API**

```python
h, w = img.shape[:2]
px   = img[y, x]              # 单个像素 (B,G,R)
blue = img[y, x, 0]
img[y, x] = (0, 0, 255)       # 写像素：红
roi  = img[y1:y2, x1:x2]      # ROI，注意 y2/x2 不包含
b, g, r = cv2.split(img)
merged  = cv2.merge([b, g, r])
masked  = cv2.bitwise_and(img, img, mask=mask)
```

**关键点**

| 写法 | 含义 |
|---|---|
| `img.shape` | `(h, w, 3)` 彩色；`(h, w)` 灰度 |
| `img[h//2, w//2]` | 正中心像素 |
| `img[100:200, 50:150]` | 第 100~199 行、第 50~149 列 |
| `img[:, :, 0] = 0` | 整幅图蓝色通道清零 |

**最小可运行代码**

```python
import cv2

img = cv2.imread("test.jpg")
h, w = img.shape[:2]
print(f"高={h} 宽={w} 通道={img.shape[2]}")

# 取中间一块 ROI 并涂成绿色
roi = img[h//4:h//2, w//4:w//2]
roi[:] = (0, 255, 0)          # ★ 用 [:] 才能写回原图

cv2.rectangle(img, (w//4, h//4), (w//2, h//2), (255, 0, 0), 2)
cv2.imshow("roi", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **`roi = img[a:b, c:d]` 是视图不是拷贝！** 改 `roi` 会改原图；想独立操作用 `roi = img[a:b, c:d].copy()`。
- 切片是 `[y, x]`，不是 `[x, y]`。**招新最爱考的坑**。
- 灰度图没有通道维，写 `img[y, x, 0]` 会 IndexError。
- `cv2.split` 较慢，能用切片 `img[:, :, 0]` 就用切片。

---

## 3. 绘图与几何变换 【必背】

### 3.1 画线 / 矩形 / 圆 / 文字

**一句话概念**：OpenCV 的绘图函数**直接在原数组上修改**，颜色是 BGR 元组，`thickness=-1` 表示填充。

**核心 API**

```python
cv2.line(img, pt1, pt2, color, thickness=1, lineType=cv2.LINE_AA)
cv2.rectangle(img, pt1, pt2, color, thickness=1)     # pt 是左上、右下
cv2.circle(img, center, radius, color, thickness=1)
cv2.putText(img, text, org, fontFace, fontScale, color, thickness, lineType)
cv2.polylines(img, [pts], isClosed, color, thickness)
```

**关键参数**

| 参数          | 说明                                                 |
| ----------- | -------------------------------------------------- |
| `color`     | BGR 元组，如 `(0,255,0)` 绿、`(0,0,255)` 红、`(255,0,0)` 蓝 |
| `thickness` | 线宽；`-1` 或 `cv2.FILLED` = 实心填充                      |
| `lineType`  | `cv2.LINE_AA` 抗锯齿，画圆/斜线更顺滑                         |
| `fontFace`  | 常用 `cv2.FONT_HERSHEY_SIMPLEX`                      |
| `fontScale` | 字号缩放，1.0 左右合适                                      |
| `org`       | 文字**左下角**坐标（不是左上角！）                                |

**最小可运行代码**

```python
import cv2, numpy as np

img = np.zeros((400, 600, 3), np.uint8) + 30

cv2.line(img, (20, 20), (580, 20), (255, 255, 255), 2)
cv2.rectangle(img, (50, 60), (250, 200), (0, 255, 0), 2)        # 空心
cv2.rectangle(img, (300, 60), (500, 200), (255, 0, 0), -1)      # 实心
cv2.circle(img, (150, 300), 60, (0, 0, 255), 3, cv2.LINE_AA)
cv2.putText(img, "Hello MV", (300, 300), cv2.FONT_HERSHEY_SIMPLEX,
            1.2, (0, 255, 255), 2, cv2.LINE_AA)

cv2.imshow("draw", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **绘图会污染原图** → `canvas = img.copy()`。
- `putText` **不支持中文**，会显示成 `????`。中文用 PIL：
  ```python
  from PIL import Image, ImageDraw, ImageFont
  pil = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
  ImageDraw.Draw(pil).text((10, 10), "中文", font=ImageFont.truetype("simhei.ttf", 24),
                           fill=(255, 0, 0))
  img = cv2.cvtColor(np.array(pil), cv2.COLOR_RGB2BGR)
  ```
- 坐标是 `(x, y)`，和 numpy 的 `[y, x]` **刚好相反**，别搞混。
- 画到边界外不会报错，但看不到。

---

### 3.2 resize 【必背】

**一句话概念**：改图像尺寸，`dsize` 是 **(宽, 高)**，不是 (高, 宽)。

**核心 API**

```python
dst = cv2.resize(src, dsize, fx=0, fy=0, interpolation=cv2.INTER_LINEAR)
dst = cv2.resize(src, None, fx=0.5, fy=0.5)     # 按比例缩放
```

**关键参数**

| `interpolation` | 用途 |
|---|---|
| `cv2.INTER_LINEAR` | 默认，**放大**用（双线性，快且平滑） |
| `cv2.INTER_AREA` | **缩小**首选（区域重采样，不会产生摩尔纹） |
| `cv2.INTER_NEAREST` | 最近邻，最快，做标签图/分割掩膜时保持类别值 |
| `cv2.INTER_CUBIC` | 放大质量更好但更慢 |

**最小可运行代码**

```python
import cv2
img = cv2.imread("test.jpg")
small = cv2.resize(img, (320, 240))              # 宽320 高240 ★
half  = cv2.resize(img, None, fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)
big   = cv2.resize(img, None, fx=2.0, fy=2.0, interpolation=cv2.INTER_CUBIC)
print(small.shape)                                # (240, 320, 3) ← 高在前
cv2.imshow("small", small); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- `dsize=(w, h)`，但 `img.shape` 是 `(h, w)`。**用 `img.shape[:2][::-1]` 最保险**。
- 缩小用 `INTER_LINEAR` 会糊/有摩尔纹，用 `INTER_AREA`。
- `dsize=None` 必须配合 `fx/fy`；`dsize` 和 `fx/fy` 同时给时以 `dsize` 为准。
- 改尺寸后原图与缩放图坐标要按比例换算：`x_orig = x_small / scale`。

---

### 3.3 旋转与仿射变换 【理解】

**一句话概念**：仿射变换用 **3 个点对**决定一个 2×3 矩阵，能做旋转、平移、缩放、错切。

**核心 API**

```python
M = cv2.getRotationMatrix2D(center, angle, scale)   # 2x3，绕点旋转
dst = cv2.warpAffine(src, M, (w, h))

M2 = cv2.getAffineTransform(src_pts, dst_pts)       # 3点 -> 2x3
dst = cv2.warpAffine(src, M2, (w, h))
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `center` | 旋转中心 `(x, y)`，一般 `(w/2, h/2)` |
| `angle` | **正数 = 逆时针**，单位度 |
| `scale` | 缩放系数，1.0 不变 |
| `dsize` | 输出尺寸 **(宽, 高)** |
| `src_pts` | `np.float32` 类型，形状 `(3,2)` |

**最小可运行代码**

```python
import cv2, numpy as np

img = cv2.imread("test.jpg")
h, w = img.shape[:2]

# 绕中心逆时针转 45°，不缩放
M = cv2.getRotationMatrix2D((w / 2, h / 2), 45, 1.0)
rot = cv2.warpAffine(img, M, (w, h))

# 仿射：把三角形映射到另一个三角形
src = np.float32([[50, 50], [200, 50], [50, 200]])
dst_pts = np.float32([[10, 100], [200, 20], [100, 250]])
M2 = cv2.getAffineTransform(src, dst_pts)
aff = cv2.warpAffine(img, M2, (w, h))

cv2.imshow("rot", rot); cv2.imshow("affine", aff)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- 点数组必须 `np.float32`，写 `np.float64` 会报错。
- 旋转后**四角被裁掉** → 想完整保留要手动扩大输出画布并调整 M 的平移分量。
- `warpAffine` 的 `dsize` 是 **(宽, 高)**。
- 仿射保持「平行线仍平行」；透视变换不保持。

---

### 3.4 透视变换（文档矫正）【必背·实操高频】

**一句话概念**：透视变换用 **4 个点对**把任意四边形拉成正视图，招新实操「把歪的 A4 纸/工牌摆正」就是这个。

**核心 API**

```python
M = cv2.getPerspectiveTransform(src_pts, dst_pts)     # 4点 -> 3x3
dst = cv2.warpPerspective(src, M, (w, h))
```

**标准套路代码**

```python
import cv2, numpy as np

img = cv2.imread("card.jpg")
h, w = img.shape[:2]

# 1) 原图中卡片的四个角：左上、右上、右下、左下（顺序必须一致）
src = np.float32([[120, 80], [520, 110], [540, 430], [100, 400]])

# 2) 目标正视图尺寸（宽 x 高，自己按卡片宽高比定）
W, H = 400, 260
dst_pts = np.float32([[0, 0], [W, 0], [W, H], [0, H]])

M = cv2.getPerspectiveTransform(src, dst_pts)
warped = cv2.warpPerspective(img, M, (W, H))

cv2.imshow("src", img); cv2.imshow("warped", warped)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- 4 个点**顺序必须对应**（都顺时针或都逆时针），顺序错 → 图像扭曲/翻转。
- 点必须是 `np.float32`，形状 `(4,2)`。
- 结果尺寸 `(W, H)` 自己定，要按真实宽高比，否则会被拉扁。
- 更自动的做法：`cv2.findContours` → 取面积最大的四边形 → `approxPolyDP` 得 4 点 → 透视变换。

---

## 4. 颜色与预处理 【必背】

### 4.1 cvtColor

**一句话概念**：颜色空间转换，最常用的是 BGR→GRAY 和 BGR→HSV。

**核心 API / 常用转换码**

| 转换码 | 用途 |
|---|---|
| `cv2.COLOR_BGR2GRAY` | 灰度化（边缘、阈值、检测的前置） |
| `cv2.COLOR_BGR2HSV` | 颜色分割（颜色追踪） |
| `cv2.COLOR_BGR2RGB` | 给 matplotlib / PIL 显示 |
| `cv2.COLOR_GRAY2BGR` | 灰度图叠到彩色图时补齐 3 通道 |
| `cv2.COLOR_HSV2BGR` | 反向 |

```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
hsv  = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
```

**常见坑**：灰度图再 `COLOR_BGR2GRAY` 会报错；HSV 在 OpenCV 里 **H∈[0,179]、S∈[0,255]、V∈[0,255]**（不是 0~360）。

---

### 4.2 threshold 与 Otsu 【必背】

**一句话概念**：固定阈值把灰度图二值化；图像直方图呈**双峰**时用 Otsu 自动找最佳阈值。

**核心 API**

```python
ret, dst = cv2.threshold(src, thresh, maxval, type)
ret, dst = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```

**关键参数**

| 参数/类型 | 说明 |
|---|---|
| `thresh` | 阈值；用 Otsu 时随便填 0（会被覆盖） |
| `maxval` | 超过阈值的像素赋的值，一般 255 |
| `THRESH_BINARY` | >thresh → maxval，否则 0 |
| `THRESH_BINARY_INV` | 反过来（**黑底白前景**时常用） |
| `THRESH_TRUNC` | 超过则截断为 thresh |
| `THRESH_TOZERO` / `_INV` | 低于阈值置 0 / 高于置 0 |
| `THRESH_OTSU` | 与 BINARY 或 BINARY_INV 相加使用 |
| `THRESH_TRIANGLE` | 单峰直方图（如细胞图）时可用 |

**最小可运行代码**

```python
import cv2

gray = cv2.imread("test.jpg", cv2.IMREAD_GRAYSCALE)
gray = cv2.GaussianBlur(gray, (5, 5), 0)        # ★ 先滤波再阈值，效果更稳

ret1, th1 = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)      # 手动
ret2, th2 = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
print("Otsu 自动阈值 =", ret2)                   # ret 只在 Otsu 时有用

cv2.imshow("manual", th1); cv2.imshow("otsu", th2)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- 返回的是**两个值** `(ret, dst)`，忘记接 `ret` 会报错。非 Otsu 时 `ret == thresh`。
- 输入必须是**单通道**（8 位或 16 位）；彩色图先 `cvtColor`。
- 光照不均（一半亮一半暗）别用全局阈值 → 用 4.3 自适应。
- Otsu 只适合**双峰**直方图，光照不均时照样翻车。

---

### 4.3 自适应阈值 【必背】

**一句话概念**：每个像素用它**邻域**的均值/高斯加权决定阈值，专门治光照不均。

**核心 API**

```python
dst = cv2.adaptiveThreshold(src, maxValue, adaptiveMethod,
                            thresholdType, blockSize, C)
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `adaptiveMethod` | `cv2.ADAPTIVE_THRESH_MEAN_C`（邻域均值）/ `ADAPTIVE_THRESH_GAUSSIAN_C`（高斯加权，更平滑） |
| `thresholdType` | 只能是 `THRESH_BINARY` 或 `THRESH_BINARY_INV` |
| `blockSize` | 邻域尺寸，**必须是大于 1 的奇数**（3、11、31…） |
| `C` | 常数，从阈值里减掉：`T = 邻域均值 - C`，**越大越保守（前景越少）** |

**最小可运行代码**

```python
import cv2

gray = cv2.imread("doc.jpg", cv2.IMREAD_GRAYSCALE)
blur = cv2.GaussianBlur(gray, (5, 5), 0)

adap = cv2.adaptiveThreshold(blur, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                             cv2.THRESH_BINARY, 31, 10)
_, otsu = cv2.threshold(blur, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

cv2.imshow("otsu", otsu); cv2.imshow("adaptive", adap)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**：`blockSize` 填偶数直接抛异常；`blockSize` 太小 → 满屏噪点；太大 → 退化成全局阈值。文档二值化常用 `blockSize=31, C=10` 起步。

---

### 4.4 四种滤波 【必背】

**一句话概念**：去噪 = 牺牲一点清晰度换稳定性；**高斯**去高斯噪声，**中值**去椒盐噪声，**双边**保边去噪。

**核心 API**

```python
cv2.blur(src, ksize)                                        # 均值，最快
cv2.GaussianBlur(src, ksize, sigmaX, sigmaY=0)              # 最常用
cv2.medianBlur(src, ksize)                                  # 椒盐噪声克星
cv2.bilateralFilter(src, d, sigmaColor, sigmaSpace)         # 保边，慢
```

**对比表（面试常问）**

| 滤波 | 核要求 | 去什么噪声 | 边缘 | 速度 | 典型场景 |
|---|---|---|---|---|---|
| `blur` | 任意 | 一般噪声 | 糊 | 最快 | 临时降噪 |
| `GaussianBlur` | **奇数** | 高斯噪声 | 糊 | 快 | **Canny/阈值前的标配** |
| `medianBlur` | **奇数 >1** | **椒盐噪声** | 保留较好 | 中 | 去除孤立黑白点 |
| `bilateralFilter` | — | 一般噪声 | **保边** | **最慢** | 美颜、保边预处理 |

**最小可运行代码**

```python
import cv2

img = cv2.imread("noisy.jpg")
g = cv2.GaussianBlur(img, (5, 5), 0)          # ksize=(5,5)，sigmaX=0 自动算
m = cv2.medianBlur(img, 5)                     # ★ 只传一个整数
b = cv2.bilateralFilter(img, 9, 75, 75)        # d=9, sigmaColor=75, sigmaSpace=75

cv2.imshow("gauss", g); cv2.imshow("median", m); cv2.imshow("bilat", b)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- `GaussianBlur` 的 `ksize` 必须**正奇数**，`(4,4)` 直接报错。
- `medianBlur` 的 `ksize` 是**整数**不是元组；且 >5 在 8 位图之外可能有限制。
- `sigmaX=0` 时 OpenCV 按 `0.3*((ksize-1)*0.5 - 1) + 0.8` 自动算，一般够用。
- 滤波 kernel 越大越糊，轮廓/边缘任务里 **3 或 5 就够**。

---

### 4.5 直方图均衡化 【了解】

```python
eq = cv2.equalizeHist(gray)                        # 全局，可能过曝

clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
eq2 = clahe.apply(gray)                            # 分块自适应，更自然
```

**用途**：人脸检测、低对比度图增强。**坑**：只能对单通道 8 位图用；彩色图要转到 YCrCb 只对 Y 通道做。

---

## 5. 边缘检测与形态学 【必背】

### 5.1 Sobel / Scharr / Laplacian

**一句话概念**：梯度 = 亮度变化快慢；变化大的地方就是边缘。

**核心 API**

```python
cv2.Sobel(src, ddepth, dx, dy, ksize=3)
cv2.Scharr(src, ddepth, dx, dy)                     # ksize 固定为 3，更精确
cv2.Laplacian(src, ddepth, ksize=3)
cv2.convertScaleAbs(src)                            # ★ 取绝对值并转回 uint8
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `ddepth` | 用 `cv2.CV_64F` 保留负梯度，算完再 `convertScaleAbs`；用 `-1` 会**截断负值丢一半边缘** |
| `dx, dy` | `dx=1,dy=0` → x 方向梯度，**突出竖直边缘**；`dx=0,dy=1` → 突出水平边缘 |
| `ksize` | 1/3/5/7，越大越平滑（`ksize=-1` 在 Sobel 中等价于 Scharr） |

**最小可运行代码**

```python
import cv2, numpy as np

gray = cv2.imread("test.jpg", cv2.IMREAD_GRAYSCALE)
gray = cv2.GaussianBlur(gray, (3, 3), 0)

sx = cv2.convertScaleAbs(cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3))  # 竖直边缘
sy = cv2.convertScaleAbs(cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3))  # 水平边缘
sobel = cv2.addWeighted(sx, 0.5, sy, 0.5, 0)                          # 合并

lap = cv2.convertScaleAbs(cv2.Laplacian(gray, cv2.CV_64F))

cv2.imshow("sobel", sobel); cv2.imshow("laplacian", lap)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- 忘 `convertScaleAbs` → 图像几乎全黑。
- 一阶（Sobel）对梯度方向敏感；二阶（Laplacian）**对噪声极敏感**，用前一定模糊。
- 先高斯再求导是标配（Sobel 内部有轻微平滑，Laplacian 没有）。

---

### 5.2 Canny 【必背·面试必考】

**一句话概念**：工业级边缘检测，效果最好也最常用，内部是「高斯 → 梯度 → 非极大值抑制 → 双阈值滞后」四步。

**四步流程（面试背下来）**

1. **高斯滤波**降噪（`GaussianBlur`）
2. **计算梯度**幅值和方向（Sobel）
3. **非极大值抑制 NMS**：沿梯度方向只保留最强的点 → 边缘变细成 1 像素
4. **双阈值 + 滞后连接**：
   - 梯度 > `threshold2`(`maxVal`) → 强边缘，保留
   - < `threshold1`(`minVal`) → 丢弃
   - 介于两者 → 只有**连在强边缘上**才保留

**核心 API**

```python
edges = cv2.Canny(image, threshold1, threshold2, apertureSize=3, L2gradient=False)
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `threshold1` | 低阈值 `minVal`，越小边缘越多、噪点越多 |
| `threshold2` | 高阈值 `maxVal`，**经验取 `threshold2 ≈ 2~3 × threshold1`** |
| `apertureSize` | Sobel 核大小，默认 3 |
| `L2gradient` | `True` 用平方根算梯度（更准更慢），默认 `False` |

**最小可运行代码**

```python
import cv2

img = cv2.imread("test.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
blur = cv2.GaussianBlur(gray, (5, 5), 0)          # ★ 先降噪

edges = cv2.Canny(blur, 50, 150)                   # 1:3 黄金比例
cv2.imshow("edges", edges); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **输入必须单通道**：传 3 通道彩色图会报错。
- 不加高斯 → 满屏噪点边缘。
- 阈值凭感觉调不出来时：先用 `cv2.createTrackbar` 现场拖两个滑块。
- Canny 输出是**二值边缘图**，可以直接喂给 `findContours`（但边缘不闭合时轮廓会乱，做轮廓通常用**阈值+形态学**更稳）。

---

### 5.3 腐蚀 / 膨胀 / 开闭运算 【必背·面试必考】

**一句话概念**（记死这两句）：

- **腐蚀 erode**：白色前景**变小**，去掉小白点，断开粘连 → 「瘦身」
- **膨胀 dilate**：白色前景**变大**，填掉小黑洞，连接断裂 → 「增肥」
- **开运算** = 先腐蚀后膨胀 = **去小白点/毛刺**，大物体尺寸基本不变
- **闭运算** = 先膨胀后腐蚀 = **填小黑洞/连接断口**

**核心 API**

```python
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
dst = cv2.erode(src, kernel, iterations=1)
dst = cv2.dilate(src, kernel, iterations=1)
dst = cv2.morphologyEx(src, cv2.MORPH_OPEN, kernel, iterations=1)
```

**关键参数**

| 项 | 说明 |
|---|---|
| kernel 形状 | `MORPH_RECT` 矩形 / `MORPH_ELLIPSE` 椭圆（圆形物体） / `MORPH_CROSS` 十字 |
| kernel 大小 | `(5,5)`、`(3,3)` 常用；越大效果越猛 |
| `iterations` | 重复次数，等效于放大 kernel |
| `MORPH_OPEN` | 开：去小白点、去毛刺 |
| `MORPH_CLOSE` | 闭：填小黑洞、连断裂 |
| `MORPH_GRADIENT` | 膨胀-腐蚀 = 得到轮廓 |
| `MORPH_TOPHAT` | 原图-开 = 提取亮的小目标 |
| `MORPH_BLACKHAT` | 闭-原图 = 提取暗的小目标 |

**最小可运行代码**

```python
import cv2, numpy as np

# 造一个带噪点的二值图
mask = np.zeros((300, 300), np.uint8)
cv2.rectangle(mask, (80, 80), (220, 220), 255, -1)
for _ in range(200):                              # 随机小白点
    x, y = np.random.randint(0, 300, 2)
    mask[y, x] = 255
mask[140:160, 80:220] = 0                          # 一条裂缝

k = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
opened = cv2.morphologyEx(mask, cv2.MORPH_OPEN, k)     # 去白点
closed = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, k)    # 补裂缝

cv2.imshow("mask", mask)
cv2.imshow("open", opened)
cv2.imshow("close", closed)
cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **只对二值图/掩膜做形态学**，对彩色图直接做语义很奇怪。
- 核越大越慢、物体变形越多；**轮廓任务里核不要超过 (7,7)**。
- 顺序是死的：开 = 先腐蚀；闭 = 先膨胀。记法：**开**运算把小的**开**掉，**闭**运算把洞**闭**上。
- 腐蚀后物体可能消失 → 面积阈值筛选前先想清楚。

---

## 6. 轮廓分析 【必背·实操核心】

### 6.1 findContours / drawContours

**一句话概念**：在**二值图**上找出所有连通区域的边界点集。

**核心 API**

```python
contours, hierarchy = cv2.findContours(image, mode, method)   # ★ OpenCV 4.x 只返回 2 个值
cv2.drawContours(img, contours, contourIdx, color, thickness)
```

**关键参数**

| 参数 | 取值 | 含义 |
|---|---|---|
| `image` | 二值图 | **必须是单通道二值图**（8 位） |
| `mode` | `cv2.RETR_EXTERNAL` | 只取**最外层**轮廓（最常用） |
| | `cv2.RETR_LIST` | 全部轮廓，不建层级 |
| | `cv2.RETR_CCOMP` | 两层：外轮廓 + 内孔 |
| | `cv2.RETR_TREE` | 完整层级树 |
| `method` | `cv2.CHAIN_APPROX_SIMPLE` | **压缩**水平/垂直/对角段，只留端点（省内存，常用） |
| | `cv2.CHAIN_APPROX_NONE` | 存所有边界点 |
| `contourIdx` | `-1` | 画全部轮廓 |
| `thickness` | `-1` | 填充轮廓内部 |

**最小可运行代码**

```python
import cv2

img = cv2.imread("shapes.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
_, th = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
th = cv2.morphologyEx(th, cv2.MORPH_OPEN,
                      cv2.getStructuringElement(cv2.MORPH_RECT, (3, 3)))

contours, hierarchy = cv2.findContours(th, cv2.RETR_EXTERNAL,
                                       cv2.CHAIN_APPROX_SIMPLE)
print("找到轮廓数：", len(contours))

vis = img.copy()
cv2.drawContours(vis, contours, -1, (0, 255, 0), 2)
cv2.imshow("contours", vis); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑（超高频）**

- **OpenCV 4.x 返回 2 个值**，3.x 返回 3 个。写 3 个会 `ValueError: not enough values to unpack`。
- 输入**必须是二值图**：彩色图不报错但结果乱；要么 `cvtColor`+`threshold`，要么 `Canny`。
- **前景必须是白色，背景黑色**。反了就 `THRESH_BINARY_INV` 或 `cv2.bitwise_not`。
- `findContours` 在 OpenCV 4 里**不再修改输入图**（3.x 会改）。
- 轮廓点形状是 `(N, 1, 2)`，转成 `(N, 2)` 用 `cnt.reshape(-1, 2)`。
- 噪点会产生大量小轮廓 → 用**面积阈值**过滤：
  ```python
  contours = [c for c in contours if cv2.contourArea(c) > 100]
  ```

---

### 6.2 轮廓信息全家桶【必背】

**核心 API 一览**

| 目的 | API | 返回值 |
|---|---|---|
| 面积 | `cv2.contourArea(cnt)` | float（像素²） |
| 周长 | `cv2.arcLength(cnt, True)` | float，`True`=闭合 |
| 正外接矩形 | `cv2.boundingRect(cnt)` | `(x, y, w, h)` |
| 最小外接矩形（可旋转） | `cv2.minAreaRect(cnt)` | `((cx,cy), (w,h), angle)` |
| 最小外接矩形 4 顶点 | `cv2.boxPoints(rect)` | `(4,2)` float，用前 `np.intp()` |
| 最小外接圆 | `cv2.minEnclosingCircle(cnt)` | `((cx,cy), radius)` 都是 float |
| 多边形近似 | `cv2.approxPolyDP(cnt, eps, True)` | 近似后的点集 |
| 凸包 | `cv2.convexHull(cnt)` | 凸包点集 |
| 是否凸 | `cv2.isContourConvex(cnt)` | `True/False` |
| 形状匹配 | `cv2.matchShapes(c1, c2, cv2.CONTOURS_MATCH_I1, 0)` | 越小越像 |

**最小可运行代码（画全套）**

```python
import cv2, numpy as np

img = cv2.imread("shapes.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
_, th = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
contours, _ = cv2.findContours(th, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

vis = img.copy()
for cnt in contours:
    area = cv2.contourArea(cnt)
    if area < 200:                                # ★ 过滤噪点
        continue

    peri = cv2.arcLength(cnt, True)

    x, y, w, h = cv2.boundingRect(cnt)            # 正矩形
    cv2.rectangle(vis, (x, y), (x + w, y + h), (0, 255, 0), 2)

    rect = cv2.minAreaRect(cnt)                   # 最小外接矩形
    box = np.intp(cv2.boxPoints(rect))
    cv2.drawContours(vis, [box], 0, (255, 0, 0), 2)

    (cx, cy), r = cv2.minEnclosingCircle(cnt)     # 最小外接圆
    cv2.circle(vis, (int(cx), int(cy)), int(r), (0, 0, 255), 2)

    approx = cv2.approxPolyDP(cnt, 0.02 * peri, True)   # ★ eps 用周长的百分比
    hull = cv2.convexHull(cnt)
    cv2.drawContours(vis, [hull], 0, (0, 255, 255), 1)

    # 用顶点数猜形状
    v = len(approx)
    name = {3: "Triangle", 4: "Quad"}.get(v, "Circle" if v > 8 else "Poly")
    cv2.putText(vis, f"{name} A={int(area)}", (x, y - 6),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)

cv2.imshow("analysis", vis); cv2.waitKey(0); cv2.destroyAllWindows()
```

**用顶点数识别形状（招新实操常考）**

| `len(approx)` | 形状 |
|---|---|
| 3 | 三角形 |
| 4 | 四边形（用 `w/h` 比例进一步判正方形 / 长方形） |
| 5 | 五边形 |
| >8~10 且接近圆 | 圆形（可配合 `4πA/P²` 圆度判断） |

**常见坑**

- `minAreaRect` 的角度在 OpenCV 4.5+ 里范围是 **(0, 90]**，老版本是 `[-90, 0)`，跨版本对比会踩坑。
- `boxPoints` 返回 float，画图前要 `np.intp()` / `.astype(int)`。
- `minEnclosingCircle` 返回 float 圆心和半径，`cv2.circle` 需要 int。
- `approxPolyDP` 的 `epsilon` 太小 → 点太多，太大 → 形状丢失；常用 `0.02 * 周长`。
- `contourArea` 对**自相交**或**未闭合**轮廓不准确。
- **`boundingRect` 与 `minAreaRect` 区别**：前者是水平正矩形，后者是带角度、面积最小的旋转矩形（测倾斜零件用后者）。

---

## 7. 视频与摄像头 【必背】

**一句话概念**：视频就是「一帧帧的图 + 一个循环」，`waitKey` 同时负责**刷新窗口**和**收键盘**。

**核心 API**

```python
cap = cv2.VideoCapture(0)              # 0=默认摄像头；也可传视频文件路径
ret, frame = cap.read()                # ret=False 表示读完/读失败
cap.isOpened()                         # ★ 先判断是否打开成功
cap.release()

out = cv2.VideoWriter(path, fourcc, fps, (w, h))    # ★ (宽, 高)
out.write(frame)
out.release()
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `fourcc` | `cv2.VideoWriter_fourcc(*'mp4v')` + `.mp4`；`*'XVID'` + `.avi`；`*'MJPG'` |
| `fps` | 帧率，摄像头可 `cap.get(cv2.CAP_PROP_FPS)` 取（常返回 0，直接填 20~30） |
| `frameSize` | **(宽, 高)**，必须和 `write` 的帧尺寸**完全一致**，否则写入失败且不报错 |
| `cap.get/set` | `CAP_PROP_FRAME_WIDTH/HEIGHT/FPS/POS_FRAMES/FRAME_COUNT` |

**最小可运行代码（摄像头标准模板，背下来）**

```python
import cv2

cap = cv2.VideoCapture(0)
if not cap.isOpened():
    raise SystemExit("摄像头打开失败：被占用 / 索引不是 0 / 权限问题")

cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

while True:
    ret, frame = cap.read()
    if not ret:                       # ★ 必须判断
        break

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 50, 150)

    cv2.imshow("frame", frame)
    cv2.imshow("edges", edges)

    key = cv2.waitKey(1) & 0xFF       # ★ & 0xFF 别省
    if key == ord('q'):               # 按 q 退出
        break
    if key == ord('s'):               # 按 s 截图
        cv2.imwrite("shot.png", frame)

cap.release()
cv2.destroyAllWindows()
```

**最小可运行代码（录像 + 播放）**

```python
import cv2

cap = cv2.VideoCapture(0)
w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fourcc = cv2.VideoWriter_fourcc(*'mp4v')
out = cv2.VideoWriter("record.mp4", fourcc, 20.0, (w, h))   # ★ (宽, 高)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    out.write(frame)
    cv2.imshow("rec", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release(); out.release(); cv2.destroyAllWindows()

# 回放
cap = cv2.VideoCapture("record.mp4")
while True:
    ret, frame = cap.read()
    if not ret:
        break
    cv2.imshow("play", frame)
    if cv2.waitKey(30) & 0xFF == 27:      # 30 ≈ 33fps；27 = ESC
        break
cap.release(); cv2.destroyAllWindows()
```

**常见坑**

- **`waitKey` 的三个坑**：① 必须在 `imshow` 之后调用，否则窗口不刷新；② 返回值要 `& 0xFF` 才能和 `ord('q')` 比；③ 循环里 `waitKey(0)` 会卡死。
- `cap.read()` 不判断 `ret` → 视频结尾拿到 `None`，下一行崩。
- `VideoWriter` 尺寸不匹配 → 生成 0 字节文件，**不报错**。
- 摄像头被其他软件（腾讯会议/相机 App）占用 → `isOpened()` 为 False，先关掉。
- `VideoWriter` 写 `mp4` 有时受 ffmpeg 编码器限制，**存 `.avi` + XVID 最稳**。
- 没有装 GUI 版 opencv（headless）→ `imshow` 直接报错。

---

## 8. 特征与匹配 【理解】

### 8.1 ORB / SIFT 关键点与描述子

**一句话概念**：特征点 = 图像里「有辨识度」的角点，描述子 = 该点的**指纹向量**，用来跨图匹配。

**核心 API**

```python
orb  = cv2.ORB_create(nfeatures=500)          # 快，二进制描述子
sift = cv2.SIFT_create()                      # 准，128 维 float 描述子（4.4+ 在主库）

kp, des = orb.detectAndCompute(gray, None)    # 一步到位
img_kp = cv2.drawKeypoints(img, kp, None, color=(0, 255, 0),
                           flags=cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS)
```

**SIFT vs ORB（面试必考对比）**

| 维度 | SIFT | ORB |
|---|---|---|
| 原理 | DoG 尺度空间极值 + 128 维梯度直方图 | FAST 角点 + BRIEF 描述子（加方向） |
| 尺度不变 | ✅ 强 | ⚠️ 弱（金字塔有限） |
| 旋转不变 | ✅ | ✅ |
| 抗光照/模糊 | ✅ 较好 | ⚠️ 一般 |
| 描述子 | 128 维 **float** | 32 字节 **二进制** |
| 匹配距离 | `NORM_L2` | `NORM_HAMMING` |
| 速度 | 慢 | **快 1~2 个数量级** |
| 是否免费 | 专利 2020 已过期 | ✅ 一直免费 |
| 场景 | 精确匹配、拼接、三维重建 | **实时 SLAM、AR、无人机** |

**最小可运行代码**

```python
import cv2

img = cv2.imread("test.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

orb = cv2.ORB_create(nfeatures=500)
kp, des = orb.detectAndCompute(gray, None)
print("特征点数：", len(kp), " 描述子 shape：", None if des is None else des.shape)

vis = cv2.drawKeypoints(img, kp, None, (0, 255, 0),
                        cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS)
cv2.imshow("ORB", vis); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- `des` 可能是 `None`（图太素、找不到特征）→ 用前判空。
- `detectAndCompute` 建议传**灰度图**；彩色也行但更慢。
- 老代码写 `cv2.SIFT()` / `cv2.xfeatures2d.SIFT_create()` → 新版报错，改成 `cv2.SIFT_create()`。
- 一张图上 `kp` 和 `des` 行数必须一一对应。

---

### 8.2 BFMatcher / FLANN

**一句话概念**：BFMatcher 暴力两两比（准、慢）；FLANN 用近似最近邻索引（快，适合特征多）。

**核心 API**

```python
bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)      # ORB 用 HAMMING
matches = bf.match(des1, des2)
matches = sorted(matches, key=lambda m: m.distance)[:50]   # 取最好的 50 个

# KNN + Lowe's ratio test（更稳）
bf2 = cv2.BFMatcher(cv2.NORM_L2)                            # SIFT 用 L2
knn = bf2.knnMatch(des1, des2, k=2)
good = [m for m, n in knn if m.distance < 0.75 * n.distance]   # ★ 0.75 经验值

# FLANN（SIFT 用 KD-Tree）
flann = cv2.FlannBasedMatcher(dict(algorithm=1, trees=5), dict(checks=50))
```

**匹配距离选型表**

| 描述子类型 | 距离 | Matcher |
|---|---|---|
| ORB / BRIEF / BRISK（二进制） | `cv2.NORM_HAMMING` | BFMatcher / FLANN+LSH |
| SIFT / SURF（float） | `cv2.NORM_L2` | BFMatcher / FLANN+KDTree |

**最小可运行代码**

```python
import cv2

img1 = cv2.imread("box.png", 0)
img2 = cv2.imread("box_in_scene.png", 0)

orb = cv2.ORB_create(1000)
kp1, des1 = orb.detectAndCompute(img1, None)
kp2, des2 = orb.detectAndCompute(img2, None)

bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
matches = sorted(bf.match(des1, des2), key=lambda m: m.distance)[:30]

vis = cv2.drawMatches(img1, kp1, img2, kp2, matches, None,
                      flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS)
cv2.imshow("matches", vis); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **ORB 用 `NORM_L2` 匹配 → 全是垃圾**；二进制描述子必须 HAMMING。
- 不做 ratio test 或不过滤距离 → 大量错误匹配。
- FLANN 用 ORB 时必须配 LSH 参数，否则报错；新手直接用 BFMatcher 更省事。
- `drawMatches` 两张图必须**同类型**（都灰度或都彩色），尺寸可以不同。

---

### 8.3 matchTemplate 模板匹配 【必背·缺点必考】

**一句话概念**：拿小模板在大图上**逐像素滑窗**算相似度，找最像的位置。只能找「大小、方向都一样」的目标。

**核心 API**

```python
res = cv2.matchTemplate(img, templ, method)
min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(res)
```

**关键参数 / 方法**

| method | 说明 |
|---|---|
| `cv2.TM_CCOEFF_NORMED` | **最常用**，归一化相关，越大越匹配（接近 1） |
| `TM_CCORR_NORMED` | 归一化互相关，亮的地方易误匹配 |
| `TM_SQDIFF_NORMED` | 平方差归一化，**越小越匹配**，取 `min_loc` |

**结果尺寸**：`res.shape == (H-h+1, W-w+1)`。

**最小可运行代码**

```python
import cv2
import numpy as np

img = cv2.imread("board.jpg")
templ = cv2.imread("chip.jpg")
h, w = templ.shape[:2]

res = cv2.matchTemplate(img, templ, cv2.TM_CCOEFF_NORMED)
min_v, max_v, min_l, max_l = cv2.minMaxLoc(res)
print("最高相似度 =", max_v)

if max_v > 0.8:                                   # ★ 一定要设阈值
    cv2.rectangle(img, max_l, (max_l[0] + w, max_l[1] + h), (0, 255, 0), 2)

# 多目标：阈值化 + 找所有峰
loc = np.where(res >= 0.8)
for pt in zip(*loc[::-1]):
    cv2.rectangle(img, pt, (pt[0] + w, pt[1] + h), (0, 0, 255), 1)

cv2.imshow("result", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**缺点（面试原题，背下来）**

1. **不支持旋转**：模板转了 5° 就匹配不上
2. **不支持缩放**：大小变了就废（要配图像金字塔/多尺度滑窗）
3. **不支持遮挡/形变**：目标被挡住一部分就失败
4. **对光照、亮度、对比度变化敏感**（`TM_CCOEFF_NORMED` 稍好）
5. **计算量大**：大图 + 大模板时 O(W·H·w·h) 很慢
6. **只能给位置，不能给类别**，本质是「找相同的图」不是「识别物体」

**适用场合**：工业上**位置固定、光照可控**的定位/有无检测（PCB 焊点、字符位置），一用就是又快又准。

---

### 8.4 霍夫变换 【理解·面试常考】

**一句话概念**：把「找直线/圆」变成在**参数空间投票**，票数超过阈值的参数就是检测到的形状。

**核心 API**

```python
# 直线（概率霍夫，返回线段端点，最实用）
lines = cv2.HoughLinesP(edges, rho, theta, threshold, minLineLength, maxLineGap)

# 标准霍夫（返回 (rho, theta)，需要自己算端点，少用）
lines = cv2.HoughLines(edges, 1, np.pi / 180, 150)

# 圆
circles = cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, dp, minDist,
                           param1=100, param2=30, minRadius=0, maxRadius=0)
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `rho` | 距离分辨率，像素，一般 1 |
| `theta` | 角度分辨率，一般 `np.pi/180`（1 度） |
| `threshold` | 累加器阈值，**越小检出越多、误检越多** |
| `minLineLength` | 最短线段长度，短于此丢弃 |
| `maxLineGap` | 允许断开的缝隙，同一直线两段间隔小于它就合并 |
| `dp` | 圆检测的分辨率倒数，`dp=1` 原分辨率 |
| `minDist` | 圆心之间最小距离，防止一堆同心圆 |
| `param1` | Canny 高阈值（内部先做 Canny） |
| `param2` | 圆心累加器阈值，**越小检出圆越多** |

**最小可运行代码**

```python
import cv2, numpy as np

img = cv2.imread("road.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
edges = cv2.Canny(cv2.GaussianBlur(gray, (5, 5), 0), 50, 150)

# --- 直线 ---
lines = cv2.HoughLinesP(edges, 1, np.pi / 180, threshold=80,
                        minLineLength=100, maxLineGap=10)
if lines is not None:
    for x1, y1, x2, y2 in lines[:, 0]:
        cv2.line(img, (x1, y1), (x2, y2), (0, 255, 0), 2)

# --- 圆 ---
gray2 = cv2.medianBlur(gray, 5)                  # ★ 圆检测前必须去噪
circles = cv2.HoughCircles(gray2, cv2.HOUGH_GRADIENT, dp=1, minDist=30,
                           param1=100, param2=30, minRadius=10, maxRadius=100)
if circles is not None:
    for cx, cy, r in np.uint16(np.around(circles))[0]:
        cv2.circle(img, (cx, cy), r, (0, 0, 255), 2)

cv2.imshow("hough", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- **必须输入边缘图（Canny 结果）**，直接喂灰度图结果很差；`HoughCircles` 例外，它自己内部做 Canny，喂**灰度图**。
- 返回值为 **`None`** 的情况要判空。
- `HoughLinesP` 返回形状 `(N, 1, 4)`，要 `lines[:, 0]` 解包。
- `HoughCircles` 返回 **float**，要 `np.uint16/np.around` 转整数才能画。
- `param2` 调小 → 圆变多（含误检）；调大 → 漏检。
- 直线检测在纹理/噪声多的图上爆炸 → 先调 Canny 阈值和 `minLineLength`。

---

## 9. 目标检测入门 【理解】

### 9.1 Haar Cascade 人脸检测

**一句话概念**：OpenCV 自带的**传统**级联分类器，用 Haar 特征滑窗 + AdaBoost，CPU 上就能实时，正脸检测很稳。

**核心 API**

```python
xml = cv2.data.haarcascades + "haarcascade_frontalface_default.xml"
face_cascade = cv2.CascadeClassifier(xml)
faces = face_cascade.detectMultiScale(gray, scaleFactor, minNeighbors, minSize)
```

**关键参数**

| 参数 | 说明 |
|---|---|
| `image` | **灰度图**（彩色能跑但慢且差） |
| `scaleFactor` | 金字塔缩放步长，`1.1` 常用；越接近 1 越慢越全 |
| `minNeighbors` | 一个候选框要被多少个邻居确认，**越大越严（漏检多）**，3~6 常用 |
| `minSize` | 最小人脸尺寸，`(30, 30)` 起步，能显著提速去误检 |
| `flags` | 老版本用 `CASCADE_SCALE_IMAGE`，新版可省 |

**常用 xml**：`haarcascade_frontalface_default.xml`（正脸）、`haarcascade_eye.xml`（眼睛）、`haarcascade_smile.xml`（微笑）、`haarcascade_fullbody.xml`（全身）、`haarcascade_licence_plate_rus_16stages.xml`（车牌）。

**最小可运行代码**

```python
import cv2

img = cv2.imread("people.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
gray = cv2.equalizeHist(gray)                  # ★ 提对比度，检出率明显提升

cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + "haarcascade_frontalface_default.xml")

faces = cascade.detectMultiScale(gray, scaleFactor=1.1,
                                 minNeighbors=5, minSize=(30, 30))
print("检测到人脸：", len(faces))

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)

cv2.imshow("faces", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**常见坑**

- 返回是 `()` 空元组（不是 None），`for` 遍历安全但 `faces.shape` 会崩。
- 传彩色图 → 速度慢，效果差；**必须灰度 + 均衡化**。
- 侧脸、遮挡、暗光**基本检不到**——Haar 只认正脸。
- 新版 OpenCV 已逐步移除部分 Haar XML，找不到文件时用 `cv2.data.haarcascades` 路径确认真实存在。
- 想要更好效果 → 换 **DNN 人脸检测器**（`cv2.dnn` + Caffe/ONNX 模型）或 **YuNet**（`cv2.FaceDetectorYN`，4.5.4+ 自带，快且准）。

---

### 9.2 颜色追踪（HSV 阈值法）【必背·实操高频】

**一句话概念**：把图转 HSV → 用 `inRange` 卡出目标颜色区域 → 形态学去噪 → 找最大轮廓 → 输出位置。

**核心 API**

```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
mask = cv2.inRange(hsv, lower, upper)          # 在 [lower, upper] 内 → 255
mask = cv2.bitwise_and(frame, frame, mask=mask) # 只保留目标区域
```

**HSV 参考范围（OpenCV：H 0~179, S/V 0~255）**

| 颜色 | lower | upper |
|---|---|---|
| 红（低段） | `(0, 100, 100)` | `(10, 255, 255)` |
| 红（高段） | `(170, 100, 100)` | `(180, 255, 255)` |
| 橙 | `(11, 100, 100)` | `(22, 255, 255)` |
| 黄 | `(23, 100, 100)` | `(34, 255, 255)` |
| 绿 | `(35, 80, 80)` | `(85, 255, 255)` |
| 青 | `(86, 80, 80)` | `(100, 255, 255)` |
| 蓝 | `(101, 100, 100)` | `(130, 255, 255)` |
| 紫 | `(131, 80, 80)` | `(160, 255, 255)` |

**常见坑**

- **红色跨越 H=0 边界，必须两段 mask 相加**：
  ```python
  mask = cv2.inRange(hsv, (0,100,100), (10,255,255)) | \
         cv2.inRange(hsv, (170,100,100), (180,255,255))
  ```
- 光照变化会改变 S/V → 卡太紧就丢目标，**先用滑条现场调**（见 13 章调参工具）。
- 背景有相似颜色 → 加面积/长宽比筛选，别只靠颜色。
- 记住 `inRange` 的上下界是 `(H, S, V)` 顺序，别和 BGR 混。

---

## 10. 机器视觉典型流程 【理解·面试爱问】

**六步法（背下来，被问「做一个视觉项目怎么做」就按这个答）**

```
采集 → 预处理 → 分割 → 特征提取 → 检测/测量/识别 → 输出
```

| 阶段 | 干什么 | 常用 API |
|---|---|---|
| **1 采集** | 相机/视频/文件读入，定分辨率、曝光、光照 | `VideoCapture`, `imread`, `cap.set` |
| **2 预处理** | 去噪、灰度化、增强、几何校正 | `cvtColor`, `GaussianBlur`, `medianBlur`, `equalizeHist`, `warpPerspective` |
| **3 分割** | 把目标从背景里分出来 | `threshold`, `adaptiveThreshold`, `inRange`, `morphologyEx`, `Canny` |
| **4 特征** | 描述目标：面积、周长、位置、形状、特征点 | `findContours`, `contourArea`, `boundingRect`, `minAreaRect`, `approxPolyDP`, `ORB`, `moments` |
| **5 检测/测量/识别** | 判定有无、分类、测尺寸、定位 | `matchTemplate`, `detectMultiScale`, `HoughLinesP`, `BFMatcher`, `dnn` |
| **6 输出** | 画框/标尺寸/存图/发结果 | `rectangle`, `putText`, `imwrite`, `VideoWriter`, 串口/PLC/网络 |

**一句话总结**：**90% 的机器视觉项目难点在第 2、3 步**——把光照和分割搞定，后面都是写代码的事。

**常见工程顺序（可直接背）**

```python
frame = 采集()
gray = 预处理(frame)          # 灰度 + 滤波
mask = 分割(gray)             # 阈值 / inRange / Canny
mask = 形态学清理(mask)       # 开闭运算去噪
contours = 找轮廓(mask)
target = 筛选最大/最符合的轮廓(contours)
结果 = 测量或判定(target)
画出结果(frame)
输出(frame)
```

---

## 11. 招新高频问题直答 【必背】

**Q：为什么 OpenCV 用 BGR？**
A：历史原因（早期 Windows 位图/DirectShow 的像素布局就是 BGR），OpenCV 沿用至今。给 `imshow` 用 BGR，给 matplotlib/PIL 前要 `cvtColor` 转 RGB。

**Q：Canny 的步骤？**
A：① 高斯滤波降噪 ② Sobel 计算梯度幅值和方向 ③ 非极大值抑制，边缘细化到 1 像素 ④ 双阈值分类 + 滞后连接（强边缘保留，弱边缘只有连着强边缘才保留）。

**Q：`findContours` 的返回值？**
A：OpenCV 4.x 返回两个：`contours`（轮廓列表，每个是 `(N,1,2)` 的点集）和 `hierarchy`（层级关系 `[Next, Previous, First_Child, Parent]`）。**3.x 返回三个**（多一个被修改的原图）。

**Q：形态学开运算和闭运算的作用？**
A：开 = 先腐蚀后膨胀 → 去掉小的白色噪点/毛刺，物体主体尺寸基本不变；闭 = 先膨胀后腐蚀 → 填补小的黑色空洞、连接断裂区域。腐蚀让白区变小，膨胀让白区变大。

**Q：霍夫变换原理？**
A：把图像空间的点映射到参数空间（直线用极坐标 (ρ,θ)，圆用 (a,b,r)），每个边缘点在参数空间给可能的参数投票，累加器峰值对应检测到的形状。`HoughLinesP` 是概率版本，只随机采样部分点，输出线段端点，更快更实用。

**Q：模板匹配的缺点？**
A：不支持旋转、缩放、遮挡、形变，对光照敏感，计算量随模板增大急剧上升，且只能定位不能识别类别。工业上用它的前提是**位置固定、光照可控**。

**Q：SIFT 和 ORB 的区别？**
A：SIFT 用 DoG 找尺度空间极值，128 维 float 描述子，尺度/旋转不变性强、精度高但慢，用 L2 距离匹配；ORB 用 FAST 角点 + 带方向的 BRIEF 二进制描述子（32 字节），速度快一到两个数量级，适合实时，用汉明距离匹配，但尺度不变性较弱。SIFT 专利 2020 年到期，OpenCV 4.4+ 已进主库。

**Q：`threshold` 和 `adaptiveThreshold` 区别？**
A：前者全图用一个阈值，要求光照均匀、直方图双峰；后者每个像素用邻域均值/高斯加权算局部阈值，适合光照不均（如手机拍文档、有阴影的工件）。

**Q：为什么阈值/轮廓前要滤波？**
A：噪声会在二值化后变成孤立小白点，在 Canny 后变成假边缘，污染后续的轮廓面积和形状判断。滤波是「用一点清晰度换稳定性」。

**Q：`cv2.waitKey(0)` 和 `waitKey(1)` 区别？**
A：`0` 无限阻塞等到按键（看图用）；`1` 等 1 毫秒，返回 -1 表示超时（视频循环用，同时负责刷新 HighGUI 窗口）。比较按键要 `waitKey(n) & 0xFF == ord('q')`。

**Q：`boundingRect` 和 `minAreaRect` 区别？**
A：`boundingRect` 返回**水平正**外接矩形 `(x,y,w,h)`；`minAreaRect` 返回**面积最小的可旋转**矩形 `((cx,cy),(w,h),angle)`。物体倾斜时要用后者才能得到真实的长宽。

**Q：轮廓面积和外接矩形面积为什么不一样？**
A：`contourArea` 是轮廓多边形围成的面积（用格林公式算），`boundingRect` 的 `w*h` 是外框面积，对非矩形/倾斜物体必然更大；填充度 = `contourArea / (w*h)`，可用来判形状。

**Q：图像坐标系原点在哪？**
A：**左上角**，x 向右增大，y 向下增大。像素访问是 `img[y, x]`。

**Q：OpenCV 内存里的图是什么？**
A：NumPy `ndarray`，`dtype` 通常 `uint8`（0~255）。所以所有 NumPy 操作（切片、掩膜、广播、`np.where`）都能直接用。

---

## 12. 常见坑合集 【必背】

| # | 坑 | 现象 | 解决 |
|---|---|---|---|
| 1 | `imread` 路径错 | 返回 `None`，下一行 `AttributeError` | 每次 `if img is None`；路径用绝对路径 |
| 2 | **中文路径** | Windows 下 `imread`/`imwrite` 静默失败 | 见下方中文路径方案 |
| 3 | **坐标顺序** | `img[y, x]` vs `(x, y)` vs `dsize=(w,h)` | 记住三条：numpy 是 `[y,x]`、绘图是 `(x,y)`、size 是 `(w,h)` |
| 4 | `waitKey` 缺失/位置错 | 窗口白屏不刷新、按 q 无效 | 必须在 `imshow` 后调用；比较加 `& 0xFF` |
| 5 | **版本差异** | `findContours` 解包失败 | 4.x 返回 **2** 个值，3.x 返回 3 个 |
| 6 | `cv2.SIFT()` 不存在 | `AttributeError` | 用 `cv2.SIFT_create()`；确保 opencv-contrib 或 4.4+ |
| 7 | 忘 `convertScaleAbs` | Sobel/Laplacian 结果全黑 | `cv2.convertScaleAbs()` 转回 uint8 |
| 8 | 椭圆 `GaussianBlur` ksize | `(4,4)` 报错 | 必须**正奇数** |
| 9 | `findContours` 输入彩色 | 不报错但结果混乱 | 先 `cvtColor` + `threshold`，且前景白背景黑 |
| 10 | 前景背景反了 | 轮廓是整张图边框 | 用 `THRESH_BINARY_INV` 或 `cv2.bitwise_not` |
| 11 | `VideoWriter` 尺寸不匹配 | 生成 0 字节视频，无报错 | `(w, h)` 与帧一致；优先 `.avi` + XVID |
| 12 | ROI 是视图 | 改 ROI 原图也变 | `roi = img[a:b, c:d].copy()` |
| 13 | 绘图污染原图 | 原图被画花了 | `canvas = img.copy()` 再画 |
| 14 | `putText` 中文 | 显示 `????` | 用 PIL `ImageFont` 绘制 |
| 15 | HSV 的 H 范围 | 用 0~360 卡不出颜色 | OpenCV H∈[0,179]，红色要分两段 |
| 16 | `np.float` 报错 | `AttributeError: np.float` | numpy≥1.24 移除，改 `float` / `np.float32` |
| 17 | 归一化忘做 | 两图相加全白 | `cv2.add` 会饱和，`cv2.addWeighted` 或先转 float |
| 18 | `imshow` 在 headless 报错 | `cv2.error: ... not implemented` | 装 `opencv-python`（非 headless），或用 `imwrite` |
| 19 | 摄像头打不开 | `isOpened()` False | 关掉占用摄像头的程序；换索引 0/1/2 |
| 20 | 轮廓点形状 | `cnt[:, 0]` 才对 | 用 `cnt.reshape(-1, 2)` 统一成 `(N,2)` |

### 中文路径终极方案（背这两行）

```python
import cv2
import numpy as np

def imread_zh(path, flags=cv2.IMREAD_COLOR):
    """支持中文路径的读图"""
    data = np.fromfile(path, dtype=np.uint8)
    return cv2.imdecode(data, flags)

def imwrite_zh(path, img):
    """支持中文路径的写图"""
    ext = "." + path.rsplit(".", 1)[-1]
    ok, buf = cv2.imencode(ext, img)
    if ok:
        buf.tofile(path)
    return ok

img = imread_zh(r"D:\项目\样本\检测图.jpg")
imwrite_zh("结果_输出.png", img)
```

### 版本差异速查

| 项目 | OpenCV 3.x | OpenCV 4.x |
|---|---|---|
| `findContours` | 返回 3 个值 | **返回 2 个值** |
| 常量名 | `cv2.CV_LOAD_IMAGE_COLOR` | `cv2.IMREAD_COLOR` |
| SIFT | 只在 contrib | 4.4+ 进主库 `cv2.SIFT_create()` |
| 轮廓修改输入图 | 会修改 | 不修改 |
| 圆形检测返回角度 | 有变化 | `minAreaRect` 角度 (0,90] |

---

## 13. 综合小项目（完整代码）

### 项目 A：颜色追踪（推荐，改一改就能当面试作品）

**流程**：摄像头采集 → 高斯滤波 → BGR→HSV → `inRange` 分割 → 开闭运算去噪 → 找最大轮廓 → 画外接矩形/最小外接圆 → 输出中心坐标。

**先跑这个调参工具**，拖动滑条找到你的目标颜色范围：

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)

def nothing(x):
    pass

cv2.namedWindow("track")
for name, val, mx in [("Hmin", 0, 179), ("Hmax", 179, 179),
                      ("Smin", 0, 255), ("Smax", 255, 255),
                      ("Vmin", 0, 255), ("Vmax", 255, 255)]:
    cv2.createTrackbar(name, "track", val, mx, nothing)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    g = cv2.getTrackbarPos("Hmin", "track"); h_ = cv2.getTrackbarPos("Hmax", "track")
    s = cv2.getTrackbarPos("Smin", "track"); s_ = cv2.getTrackbarPos("Smax", "track")
    v = cv2.getTrackbarPos("Vmin", "track"); v_ = cv2.getTrackbarPos("Vmax", "track")

    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, np.array([g, s, v]), np.array([h_, s_, v_]))
    result = cv2.bitwise_and(frame, frame, mask=mask)

    cv2.imshow("frame", frame)
    cv2.imshow("mask", mask)
    cv2.imshow("result", result)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release(); cv2.destroyAllWindows()
```

**正式版（颜色追踪 + 中心点输出）**：

```python
import cv2
import numpy as np

# ============ 参数区（按你的目标颜色改这里）============
LOWER = np.array([35, 80, 80])       # 绿色下界 (H, S, V)
UPPER = np.array([85, 255, 255])     # 绿色上界
MIN_AREA = 500                        # 最小面积，过滤噪点
# =====================================================

cap = cv2.VideoCapture(0)
if not cap.isOpened():
    raise SystemExit("摄像头打不开")

cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # 1) 预处理：轻微高斯模糊，抑制噪点
    blurred = cv2.GaussianBlur(frame, (5, 5), 0)

    # 2) 分割：BGR -> HSV，用 inRange 卡出目标颜色
    hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, LOWER, UPPER)

    # 3) 形态学：先开后闭，去小白点 + 补小洞
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel, iterations=2)
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=2)

    # 4) 找轮廓（OpenCV 4.x 返回 2 个值）
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL,
                                   cv2.CHAIN_APPROX_SIMPLE)

    if contours:
        # 5) 只处理面积最大的那个轮廓
        cnt = max(contours, key=cv2.contourArea)
        if cv2.contourArea(cnt) > MIN_AREA:
            # 外接正矩形
            x, y, w, h = cv2.boundingRect(cnt)
            cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)

            # 最小外接圆 + 圆心
            (cx, cy), r = cv2.minEnclosingCircle(cnt)
            cx, cy, r = int(cx), int(cy), int(r)
            cv2.circle(frame, (cx, cy), r, (0, 0, 255), 2)
            cv2.circle(frame, (cx, cy), 4, (255, 0, 0), -1)

            # 6) 输出：画面中心误差（可直接给云台/舵机做闭环）
            fh, fw = frame.shape[:2]
            dx, dy = cx - fw // 2, cy - fh // 2
            cv2.line(frame, (fw // 2, fh // 2), (cx, cy), (0, 255, 255), 1)
            cv2.putText(frame, f"center=({cx},{cy}) err=({dx},{dy})",
                        (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.6,
                        (255, 255, 255), 2)

    cv2.imshow("frame", frame)
    cv2.imshow("mask", mask)

    key = cv2.waitKey(1) & 0xFF
    if key == ord('q'):
        break
    if key == ord('s'):
        cv2.imwrite("track_result.png", frame)

cap.release()
cv2.destroyAllWindows()
```

**解释 / 面试可以这么讲**

- **为什么用 HSV 不用 BGR？** BGR 三通道都随光照剧烈变化，HSV 把「颜色种类(H)」和「明暗/饱和度(S,V)」解耦，H 通道对光照鲁棒得多。
- **为什么先开后闭？** 开运算去掉阈值残留的小白噪点，闭运算补上目标内部因反光产生的小黑洞，得到干净的连通区域。
- **为什么取最大轮廓？** 背景里可能有零星的同色小块，取面积最大的等价于「假设画面里只有一个人目标」。
- **输出中心坐标有什么用？** 这是**视觉伺服**的输入：把 `(dx, dy)` 交给 PID 控制舵机/小车，让它把目标拉到画面中心。
- **局限性**：光照剧变、同色背景、目标被遮挡时失效。进阶方案：加 `cv2.CamShift`/`meanShift` 做跟踪，或用 YOLO 做检测。

---

### 项目 B：人脸打码（Haar + 模糊/马赛克）

```python
import cv2

# 加载级联分类器
CASCADE_PATH = cv2.data.haarcascades + "haarcascade_frontalface_default.xml"
face_cascade = cv2.CascadeClassifier(CASCADE_PATH)
if face_cascade.empty():
    raise SystemExit("分类器加载失败，检查路径")

cap = cv2.VideoCapture(0)
mode = 0          # 0=高斯模糊  1=马赛克

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # 1) 灰度 + 直方图均衡化，显著提升检出率
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.equalizeHist(gray)

    # 2) 检测人脸（返回 (x, y, w, h) 列表；没检测到返回空元组）
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1,
                                          minNeighbors=5, minSize=(40, 40))

    # 3) 对每个人脸区域做打码
    for (x, y, w, h) in faces:
        # 防止越界
        x2, y2 = min(x + w, frame.shape[1]), min(y + h, frame.shape[0])
        x, y = max(x, 0), max(y, 0)
        roi = frame[y:y2, x:x2]
        if roi.size == 0:
            continue

        if mode == 0:
            # 高斯模糊：核必须奇数，且随人脸大小自适应
            k = max(3, (w // 3) | 1)              # 保证是奇数
            frame[y:y2, x:x2] = cv2.GaussianBlur(roi, (k, k), 0)
        else:
            # 马赛克：缩小再放大（最近邻）
            small = cv2.resize(roi, (max(1, w // 15), max(1, h // 15)),
                               interpolation=cv2.INTER_LINEAR)
            frame[y:y2, x:x2] = cv2.resize(small, (x2 - x, y2 - y),
                                           interpolation=cv2.INTER_NEAREST)

        cv2.rectangle(frame, (x, y), (x2, y2), (0, 255, 0), 2)

    cv2.putText(frame, f"mode={'Blur' if mode == 0 else 'Mosaic'} (m to switch)",
                (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 255), 2)
    cv2.imshow("face privacy", frame)

    key = cv2.waitKey(1) & 0xFF
    if key == ord('q'):
        break
    if key == ord('m'):
        mode = 1 - mode
    if key == ord('s'):
        cv2.imwrite("face_masked.png", frame)

cap.release()
cv2.destroyAllWindows()
```

**解释 / 面试可以这么讲**

- Haar 是**传统方法**：Haar-like 特征 + 积分图加速 + AdaBoost 选特征 + 级联结构（前几级就能否掉大部分非人脸窗口），所以 CPU 上很快。
- **必须灰度 + 均衡化**：Haar 只看灰度纹理对比度，彩色图对它是负担。
- **打码写在原图 ROI 上**：`frame[y:y2, x:x2] = ...`，注意这是**原地写**（ROI 是视图），正好省一次赋值回写。
- **`minNeighbors` 越大越严格**：误检少了但容易漏检；光线差时把它调小到 3。
- **升级路线**：Haar → DNN（`cv2.dnn` 加载 Caffe/ONNX）→ YuNet（`cv2.FaceDetectorYN_create`，自带且快）→ YOLOv8-face。

---

## 14. 重点 API 速查表

### 图像 IO 与基础

| 功能 | API | 关键点 |
|---|---|---|
| 读图 | `cv2.imread(path, flags)` | 失败返回 `None` |
| 显示 | `cv2.imshow(name, img)` | 需配 `waitKey` |
| 等待按键 | `cv2.waitKey(0/1) & 0xFF` | 0=无限等 |
| 关闭窗口 | `cv2.destroyAllWindows()` | — |
| 存图 | `cv2.imwrite(path, img)` | 返回 bool；扩展名定格式 |
| 尺寸 | `img.shape` → `(h, w, c)` | 灰度是 `(h, w)` |
| 类型 | `img.dtype` → `uint8` | — |
| 复制 | `img.copy()` | ROI 要拷贝就用它 |
| 通道分离 | `cv2.split(img)` / `img[:,:,0]` | 切片更快 |
| 掩膜运算 | `cv2.bitwise_and(a, b, mask=m)` | 抠图 |
| 中文读写 | `cv2.imdecode` / `cv2.imencode` + `tofile` | 见 12 章 |

### 绘图

| 功能 | API | 参数顺序 |
|---|---|---|
| 线 | `cv2.line(img, p1, p2, color, thickness)` | 点是 `(x,y)` |
| 矩形 | `cv2.rectangle(img, p1, p2, color, thickness)` | `-1` 填充 |
| 圆 | `cv2.circle(img, center, r, color, thickness)` | — |
| 文字 | `cv2.putText(img, s, org, font, scale, color, th)` | org 是左下角 |
| 多边形 | `cv2.polylines(img, [pts], True, color, th)` | 点要 `(N,1,2)` |

### 几何变换

| 功能 | API | 关键点 |
|---|---|---|
| 缩放 | `cv2.resize(src, (w,h), interpolation=...)` | 缩小用 `INTER_AREA` |
| 旋转矩阵 | `cv2.getRotationMatrix2D(center, angle, scale)` | 正角=逆时针 |
| 仿射 | `cv2.getAffineTransform(3点, 3点)` + `warpAffine` | `float32` |
| 透视 | `cv2.getPerspectiveTransform(4点, 4点)` + `warpPerspective` | 顺序要一致 |
| 翻转 | `cv2.flip(src, 1/0/-1)` | 1 水平 / 0 垂直 / -1 双向 |
| 边界填充 | `cv2.copyMakeBorder(src, t,b,l,r, type)` | — |

### 颜色 / 阈值 / 滤波

| 功能 | API | 关键参数 |
|---|---|---|
| 灰度化 | `cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)` | — |
| HSV | `cv2.cvtColor(img, cv2.COLOR_BGR2HSV)` | H∈[0,179] |
| 固定阈值 | `cv2.threshold(g, t, 255, cv2.THRESH_BINARY)` | 返回 `(ret, dst)` |
| Otsu | `cv2.threshold(g, 0, 255, cv2.THRESH_BINARY+cv2.THRESH_OTSU)` | 双峰图 |
| 自适应 | `cv2.adaptiveThreshold(g,255,METHOD,TYPE,blockSize,C)` | blockSize 奇数 |
| 颜色分割 | `cv2.inRange(hsv, lower, upper)` | 输出 0/255 掩膜 |
| 高斯滤波 | `cv2.GaussianBlur(src, (k,k), sigmaX)` | k 正奇数 |
| 中值滤波 | `cv2.medianBlur(src, k)` | 去椒盐 |
| 双边滤波 | `cv2.bilateralFilter(src, d, sc, ss)` | 保边，慢 |
| 直方图均衡 | `cv2.equalizeHist(gray)` | 单通道 8 位 |

### 边缘 / 形态学

| 功能 | API | 关键参数 |
|---|---|---|
| Sobel | `cv2.Sobel(g, cv2.CV_64F, dx, dy, ksize=3)` | 配 `convertScaleAbs` |
| Scharr | `cv2.Scharr(g, cv2.CV_64F, dx, dy)` | 比 Sobel 精确 |
| Laplacian | `cv2.Laplacian(g, cv2.CV_64F)` | 对噪声敏感 |
| Canny | `cv2.Canny(g, t1, t2)` | `t2≈2~3*t1` |
| 结构元素 | `cv2.getStructuringElement(shape, (w,h))` | RECT/ELLIPSE/CROSS |
| 腐蚀 | `cv2.erode(src, k, iterations)` | 白区变小 |
| 膨胀 | `cv2.dilate(src, k, iterations)` | 白区变大 |
| 开运算 | `cv2.morphologyEx(src, cv2.MORPH_OPEN, k)` | 去小白点 |
| 闭运算 | `cv2.morphologyEx(src, cv2.MORPH_CLOSE, k)` | 填小黑洞 |

### 轮廓

| 功能 | API | 返回值 |
|---|---|---|
| 找轮廓 | `cv2.findContours(bin, mode, method)` | **2 个值**（4.x） |
| 画轮廓 | `cv2.drawContours(img, cnts, -1, color, th)` | `-1`=全部 |
| 面积 | `cv2.contourArea(cnt)` | float |
| 周长 | `cv2.arcLength(cnt, True)` | float |
| 正外接矩形 | `cv2.boundingRect(cnt)` | `(x,y,w,h)` |
| 旋转外接矩形 | `cv2.minAreaRect(cnt)` / `cv2.boxPoints(r)` | `((cx,cy),(w,h),ang)` |
| 最小外接圆 | `cv2.minEnclosingCircle(cnt)` | `((cx,cy), r)` float |
| 多边形近似 | `cv2.approxPolyDP(cnt, eps, True)` | `eps≈0.02*周长` |
| 凸包 | `cv2.convexHull(cnt)` | 点集 |
| 矩/质心 | `cv2.moments(cnt)` | `M['m10']/M['m00']` |

### 视频

| 功能 | API | 关键点 |
|---|---|---|
| 打开 | `cv2.VideoCapture(0或路径)` | 先 `isOpened()` |
| 读帧 | `cap.read()` → `(ret, frame)` | 判 `ret` |
| 属性 | `cap.get/set(cv2.CAP_PROP_*)` | WIDTH/HEIGHT/FPS |
| 释放 | `cap.release()` | 别忘 |
| 录像 | `cv2.VideoWriter(path, fourcc, fps, (w,h))` | size 是 `(宽,高)` |
| 编码 | `cv2.VideoWriter_fourcc(*'mp4v'/'XVID')` | avi 最稳 |
| 写帧 | `out.write(frame)` | 尺寸必须一致 |

### 特征 / 检测

| 功能 | API | 关键点 |
|---|---|---|
| ORB | `cv2.ORB_create(nfeatures)` → `detectAndCompute` | HAMMING 匹配 |
| SIFT | `cv2.SIFT_create()` → `detectAndCompute` | L2 匹配 |
| 画特征 | `cv2.drawKeypoints(img, kp, None, color, flags)` | — |
| 暴力匹配 | `cv2.BFMatcher(NORM_HAMMING/L2, crossCheck=True)` | `.match` |
| KNN 匹配 | `bf.knnMatch(d1, d2, k=2)` + ratio 0.75 | 过滤误匹配 |
| 画匹配 | `cv2.drawMatches(...)` | 两图类型要一致 |
| 模板匹配 | `cv2.matchTemplate(img, tpl, cv2.TM_CCOEFF_NORMED)` + `minMaxLoc` | 不支持旋转缩放 |
| 霍夫直线 | `cv2.HoughLinesP(edges, 1, np.pi/180, thr, minLen, maxGap)` | 输入边缘图 |
| 霍夫圆 | `cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, dp, minDist, param1, param2)` | 输入灰度图 |
| 人脸 | `cv2.CascadeClassifier(xml).detectMultiScale(gray, 1.1, 5)` | 灰度+均衡化 |

---

## 15. 10 个必会代码片段

> 全是能独立跑的最小片段。考前把每个手敲一遍，比看十遍有用。

### ① 安全读图 + 显示 + 存图

```python
import cv2

img = cv2.imread("test.jpg")
if img is None:
    raise SystemExit("读图失败，检查路径")
print(img.shape, img.dtype)          # (H, W, 3) uint8
cv2.imshow("img", img)
if cv2.waitKey(0) & 0xFF == ord('q'):
    pass
cv2.destroyAllWindows()
cv2.imwrite("out.png", img)
```

### ② BGR ↔ RGB + 像素 + ROI

```python
import cv2

img = cv2.imread("test.jpg")
rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)      # 给 matplotlib 用
px = img[100, 200]                              # [y, x] -> (B,G,R)
blue = img[100, 200, 0]
roi = img[50:150, 100:250].copy()               # 拷贝，别污染原图
print(px, blue, roi.shape)
```

### ③ 缩放 / 旋转 / 透视变换

```python
import cv2, numpy as np

img = cv2.imread("card.jpg")
h, w = img.shape[:2]

small = cv2.resize(img, (320, 240), interpolation=cv2.INTER_AREA)   # (w,h)
M = cv2.getRotationMatrix2D((w / 2, h / 2), 15, 1.0)
rot = cv2.warpAffine(img, M, (w, h))

src = np.float32([[120, 80], [520, 110], [540, 430], [100, 400]])
dst = np.float32([[0, 0], [400, 0], [400, 260], [0, 260]])
warp = cv2.warpPerspective(img, cv2.getPerspectiveTransform(src, dst), (400, 260))

cv2.imshow("a", small); cv2.imshow("b", rot); cv2.imshow("c", warp)
cv2.waitKey(0); cv2.destroyAllWindows()
```

### ④ 灰度 + 高斯 + Otsu 二值化

```python
import cv2

gray = cv2.imread("test.jpg", cv2.IMREAD_GRAYSCALE)
blur = cv2.GaussianBlur(gray, (5, 5), 0)
ret, th = cv2.threshold(blur, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
print("Otsu 阈值 =", ret)
cv2.imshow("th", th); cv2.waitKey(0); cv2.destroyAllWindows()
```

### ⑤ 自适应阈值（光照不均）

```python
import cv2

gray = cv2.imread("doc.jpg", cv2.IMREAD_GRAYSCALE)
blur = cv2.GaussianBlur(gray, (5, 5), 0)
adap = cv2.adaptiveThreshold(blur, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                             cv2.THRESH_BINARY, 31, 10)   # blockSize 必须奇数
cv2.imshow("adap", adap); cv2.waitKey(0); cv2.destroyAllWindows()
```

### ⑥ Canny 边缘检测

```python
import cv2

gray = cv2.imread("test.jpg", cv2.IMREAD_GRAYSCALE)
blur = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blur, 50, 150)          # t2 ≈ 3*t1
cv2.imshow("edges", edges); cv2.waitKey(0); cv2.destroyAllWindows()
```

### ⑦ 形态学开闭运算

```python
import cv2

mask = cv2.imread("mask.png", cv2.IMREAD_GRAYSCALE)
k = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, k, iterations=2)    # 去小白点
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, k, iterations=2)   # 填小黑洞
cv2.imshow("mask", mask); cv2.waitKey(0); cv2.destroyAllWindows()
```

### ⑧ 轮廓：筛选 + 测量 + 形状判断

```python
import cv2, numpy as np

img = cv2.imread("shapes.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
_, th = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

contours, _ = cv2.findContours(th, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

for cnt in contours:
    area = cv2.contourArea(cnt)
    if area < 200:
        continue
    peri = cv2.arcLength(cnt, True)
    x, y, w, h = cv2.boundingRect(cnt)
    approx = cv2.approxPolyDP(cnt, 0.02 * peri, True)

    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)
    cv2.putText(img, f"v={len(approx)} A={int(area)}", (x, y - 5),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 255), 1)

cv2.imshow("c", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

### ⑨ 摄像头循环（万能模板，改中间的处理就行）

```python
import cv2

cap = cv2.VideoCapture(0)
if not cap.isOpened():
    raise SystemExit("摄像头打不开")

while True:
    ret, frame = cap.read()
    if not ret:
        break
    # ---- 在这里写你的处理 ----
    process = cv2.Canny(cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY), 50, 150)
    # -------------------------
    cv2.imshow("frame", frame)
    cv2.imshow("process", process)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

### ⑩ ORB 特征 + 匹配

```python
import cv2

img1 = cv2.imread("box.png", 0)
img2 = cv2.imread("scene.png", 0)

orb = cv2.ORB_create(1000)
kp1, des1 = orb.detectAndCompute(img1, None)
kp2, des2 = orb.detectAndCompute(img2, None)

bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)     # ORB 用 HAMMING
matches = sorted(bf.match(des1, des2), key=lambda m: m.distance)[:30]

vis = cv2.drawMatches(img1, kp1, img2, kp2, matches, None,
                      flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS)
cv2.imshow("matches", vis); cv2.waitKey(0); cv2.destroyAllWindows()
```

---

## 16. 面试自测 20 问

> 先遮住答案自己说一遍，说不出来的就是明天要补的。

**1. `imread` 读图失败会怎样？怎么排查？**
> 返回 `None`，**不抛异常**。排查顺序：文件是否存在 → 路径是否含中文 → 扩展名写对没 → 工作目录是不是代码所在目录。永远先 `if img is None`。

**2. OpenCV 为什么是 BGR？哪些场景会出问题？**
> 历史沿袭 Windows 位图/DirectShow 的 BGR 布局。给 `cv2.imshow`、`cv2.imwrite` 用 BGR；给 matplotlib、PIL、部分 DNN 模型（如 Caffe）前要转 RGB。

**3. `img.shape` 返回值顺序？`resize` 的 `dsize` 顺序？绘图坐标顺序？**
> `shape=(高, 宽, 通道)`；`dsize=(宽, 高)`；画图坐标 `(x, y)`。三个都不一样，最经典的坑。

**4. `cv2.threshold` 返回什么？Otsu 时 `ret` 是什么？**
> 返回 `(retval, dst)`。普通模式下 `retval == thresh`；Otsu 模式下 `retval` 是自动算出的最优阈值。

**5. 什么情况下用自适应阈值？**
> 光照不均匀（阴影、手机拍文档、环形光源打光不匀）。它逐像素用邻域均值/高斯加权算局部阈值，`blockSize` 取奇数、`C` 越大前景越少。

**6. Canny 的四个步骤？阈值怎么配？**
> 高斯滤波 → Sobel 求梯度 → 非极大值抑制 → 双阈值滞后连接。低:高 ≈ 1:2~1:3，常用 `50/150`、`100/200`。

**7. 腐蚀和膨胀分别让白色区域变大还是变小？开闭运算呢？**
> 腐蚀让白区变小（去小白点、断开粘连）；膨胀让白区变大（填小黑洞、连接断裂）。开=先腐蚀后膨胀（去小白点/毛刺）；闭=先膨胀后腐蚀（填小黑洞/连断口）。

**8. 形态学里 kernel 怎么选？**
> 形状按物体轮廓选（圆物体用 `MORPH_ELLIPSE`），大小取 `(3,3)`~`(7,7)`，太大把目标形状改掉；要更强效果优先加 `iterations`。

**9. `findContours` 在 OpenCV 4 返回几个值？分别是什么？**
> 两个：`contours`（每个元素是 `(N,1,2)` 的点集）和 `hierarchy`（层级数组，每行 `[Next, Prev, FirstChild, Parent]`）。**3.x 返回三个**。

**10. `RETR_EXTERNAL` 和 `RETR_TREE` 区别？`CHAIN_APPROX_SIMPLE` 呢？**
> `RETR_EXTERNAL` 只取最外层轮廓（速度最快，做物体计数常用）；`RETR_TREE` 建立完整层级树（要分析内孔时用）。`CHAIN_APPROX_SIMPLE` 压缩水平/垂直/对角方向的中间点，只保留端点，省内存。

**11. `boundingRect` 和 `minAreaRect` 区别？**
> 前者是水平正外接矩形 `(x,y,w,h)`；后者是面积最小的**可旋转**矩形 `((cx,cy),(w,h),angle)`，测倾斜物体的真实长宽要用后者。

**12. 怎么用轮廓判断形状是三角形/矩形/圆？**
> `approxPolyDP(cnt, 0.02*arcLength, True)` 后看顶点数：3 三角、4 四边形（再看 `w/h` 分正方形/长方形）、>8 且圆度高则为圆。也可用 `4πA/P²` 圆度或 `matchShapes`。

**13. `waitKey(0)` 和 `waitKey(1)` 区别？为什么要 `& 0xFF`？**
> `0` 无限阻塞等按键；`1` 等 1ms 返回 -1（视频循环用，同时刷新窗口）。`& 0xFF` 是为了在不同平台上把返回值截断成 ASCII 码，避免高位干扰导致和 `ord('q')` 比较失败。

**14. `VideoWriter` 需要注意什么？**
> `frameSize` 必须是 `(宽, 高)` 且与写入帧尺寸**完全一致**，否则生成 0 字节文件且不报错；`fourcc` 与容器匹配（`mp4v`+`.mp4`、`XVID`+`.avi`）；用完 `release()`。

**15. 霍夫变换原理？`HoughLinesP` 的输入应该是什么？**
> 参数空间投票，累加器峰值 = 检测到的形状。`HoughLinesP` 输入应为 **Canny 边缘图**，输出线段端点 `(N,1,4)`；`HoughCircles` 输入是**灰度图**（内部自己做 Canny），返回 float，画图前要转 int。

**16. 模板匹配的缺点？什么时候还用它？**
> 不支持旋转/缩放/遮挡/形变，光照敏感，计算量随模板增大而暴涨，只给位置不给类别。**工业定位、有无检测**（位置固定、光照可控）时它又快又稳，仍然首选。

**17. SIFT 和 ORB 的区别？匹配距离分别是什么？**
> SIFT：DoG 尺度空间 + 128 维 float 描述子，尺度/旋转不变性强、精度高、慢、用 `NORM_L2`；ORB：FAST 角点 + 带方向 BRIEF 二进制描述子（32 字节），快 1~2 个数量级、适合实时、用 `NORM_HAMMING`，尺度不变性较弱。SIFT 专利 2020 到期，OpenCV 4.4+ 已入主库。

**18. Haar 人脸检测为什么快？怎么提高检出率？**
> Haar-like 特征 + 积分图秒算特征 + AdaBoost 挑特征 + 级联结构，前几级就剔掉大部分非人脸窗口。提高检出率：灰度化 + `equalizeHist`、`scaleFactor` 调小（1.05~1.1）、`minNeighbors` 调小，或改用 DNN/YuNet。

**19. 颜色追踪为什么在 HSV 里做？红色的坑是什么？**
> HSV 把色相 H 与明暗/饱和度解耦，对光照变化比 BGR 鲁棒得多，且 `inRange` 一次就能框出颜色区域。红色 H 跨越 0/180 边界，需要两段 `inRange` 结果取**按位或**。

**20. 描述一个完整的机器视觉处理流程。**
> 采集（相机/视频/文件）→ 预处理（灰度、滤波、增强、几何校正）→ 分割（阈值/`inRange`/Canny/形态学）→ 特征提取（轮廓、面积、外接矩形、特征点）→ 检测/测量/识别（模板匹配、分类器、霍夫、DNN）→ 输出（画框标注、存图/录像、串口/网络发送结果）。**难点在预处理和分割，这两步做好后面都是写代码。**

---

## 17. 明天考前 10 分钟复习清单

**① 三个顺序（最容易失分，先背这个）**

- `img.shape` = **(高, 宽, 通道)**
- `cv2.resize` / `warpAffine` / `VideoWriter` 的 size = **(宽, 高)**
- numpy 取像素 `img[y, x]`；绘图 `cv2.rectangle((x, y), ...)`

**② 五个返回值（写错就报错）**

- `cv2.threshold` → `ret, dst`
- `cv2.findContours` → **`contours, hierarchy`（只有 2 个！）**
- `cap.read()` → `ret, frame`
- `cv2.minMaxLoc` → `minVal, maxVal, minLoc, maxLoc`
- `cv2.minEnclosingCircle` → `((cx,cy), r)`（都是 float）

**③ 五句话（面试直接说）**

1. **Canny**：高斯去噪 → Sobel 梯度 → 非极大值抑制 → 双阈值滞后连接；`t2 ≈ 2~3 × t1`
2. **形态学**：腐蚀白变小去白点，膨胀白变大填黑洞；开=先腐蚀（去小白点），闭=先膨胀（填小黑洞）
3. **HSV**：H∈[0,179]，S/V∈[0,255]；红色要分 `[0,10]` 和 `[170,180]` 两段
4. **模板匹配**：不支持旋转/缩放/遮挡，光照敏感，只适合位置固定的工业定位
5. **SIFT vs ORB**：SIFT 准而慢用 L2，ORB 快而实用汉明距离；ROI 是视图不是拷贝

**④ 一个万能代码骨架（闭眼能默写）**

```python
import cv2
img = cv2.imread("x.jpg")
if img is None: raise SystemExit("读图失败")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
blur = cv2.GaussianBlur(gray, (5, 5), 0)
_, th  = cv2.threshold(blur, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
th = cv2.morphologyEx(th, cv2.MORPH_OPEN,
                      cv2.getStructuringElement(cv2.MORPH_RECT, (3, 3)))
cnts, _ = cv2.findContours(th, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
for c in cnts:
    if cv2.contourArea(c) < 100: continue
    x, y, w, h = cv2.boundingRect(c)
    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)
cv2.imshow("r", img); cv2.waitKey(0); cv2.destroyAllWindows()
```

**⑤ 三条救场话术（不会也别慌）**

1. **参数记不清** → 「我记不准具体数值，我的习惯是先用 `createTrackbar` 现场拖滑条调，或者用 Otsu 这类自动方法，调好再写死。」
2. **函数忘了** → 「思路是先把目标分割成二值图，再去 `findContours` 拿轮廓，用 `boundingRect` 定位；具体 API 我会查 `cv2.` 的补全或文档。」
3. **被问不会的算法** → 「这个方法我了解它的适用场景和局限（说出场景+缺点），底层推导我还没深入，但我能说清它和 XX 的区别。」

**⑥ 心态**

> 招新考的是**解决问题的路径**，不是背诵量。能说清「我遇到这张图会先做什么、为什么、失败了再怎么调」，比记住 100 个函数更值钱。

---

**祝明天顺利。跑通代码的手感，比看过多少教程都管用。**

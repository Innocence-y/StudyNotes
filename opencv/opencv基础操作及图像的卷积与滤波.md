# 机器视觉第一讲  

## opencv基础操作
1. namedWindow(const String& winname, int flags = WINDOW_AUTOSIZE)  
创建一个指定名称的窗口
2. imread(const cv::String &filename ,int flags = IMREAD_COLO)  
 读取图像返回Mat
3. imshow(const String& winname, InputArray mat)  
在指定窗口显示Mat图像
4. VideoCapture  
获取视频图像的类  
VideoCapture(int index):  
0打开主摄像头,n打开其他摄像头  
VideoCapture(const String& filename):  
通过路径打开视频文件

---


## 图像处理
1. cvtColor(mat , mat , 参数) 图像处理  
CV_RGB2GREY 将彩色图像转化为灰度图
2. GaussianBlur(frame , frame , cvSize(5 , 5) , 10 , 10 ,0 );高斯滤镜
3. Canny(frame, frame, 100, 100);边缘检测
4. Sobel(frame, frame, 0, 1, 1);边缘检测

---

## Mat
计算机中的图像本质上是一个矩阵  
![pic](C://Users/lenovo.Lenovo-PC/Desktop/ComputerVision/nature of image.png "nature of imag")  
Mat是用于保存图像矩阵的数据结构
1. 单通道  
二值图像 0 255  
灰度图 0-255
2. 3通道  
RGB
3. 4通道  
RGBA


## 原理
### 浮雕
```c++
cvtColor(mat, mat, CV_RGB2GRAY);

Mat img = Mat(mat.rows, mat.cols-2, CV_8UC1);
for (int i = 0; i < mat.rows; i++) {
	for (int j = 1; j < mat.cols - 1; j++) {
		img.at<uchar>(i, j-1) = mat.at<uchar>(i, j - 1) - mat.at<uchar>(i, j + 1);
	}
}
```

### 卷积之浮雕

```c++
cvtColor(mat, mat, CV_RGB2GRAY);

Mat model = Mat(1, 3, CV_8UC1);
model.at<uchar>(0, 0) = 1;
model.at<uchar>(0, 1) = 0;
model.at<uchar>(0, 2) = -1;

Mat img = Mat(mat.rows, mat.cols-2, CV_8UC1);
for (int i = 0; i < mat.rows; i++) {
	for (int j = 1; j < mat.cols - 1; j++) {
		int sum = 0;
		for (int m = 0; m < model.rows; m++) {
			for (int n = 0; n < model.cols; n++) {
				sum += mat.at<uchar>(i, j + n -1) * model.at<uchar>(m, n);
			}
		}

		img.at<uchar>(i, j -1) = sum;
	}
}
```

### 高斯模糊
高斯模糊的原理即将正态分布应用于图像处理

### 升采样
```c++
cvtColor(mat, mat, CV_RGB2GRAY);

Mat img = Mat(mat2.rows * 2 - 1, mat2.cols * 2 - 1, CV_8UC1);
Mat imgt = Mat(mat2.rows, mat2.cols * 2 - 1, CV_8UC1);

//横向拉伸
for (int i = 0; i < mat2.rows; i++) {
	for (int j = 0; j < mat2.cols -1; j++) {
		imgt.at<uchar>(i, 2*j) = mat2.at<char>(i, j);
		imgt.at<uchar>(i, 2*j + 1) = (mat2.at<char>(i, j) + mat2.at<char>(i, j + 1)) / 2;
	}
}

//竖向拉伸
for (int i = 0; i < imgt.cols; i++) {
	for (int j = 0; j < imgt.rows - 1; j++) {
		img.at<uchar>(2 * j, i) = imgt.at<char>(j, i);
		img.at<uchar>(2 * j + 1, i) = (imgt.at<char>(j, i) + imgt.at<char>(j + 1, i)) / 2;
	}
}

```

### 降采样
```c++
cvtColor(mat, mat, CV_RGB2GRAY);

int nrows = mat.rows%2==0?mat.rows:mat.rows-1;
int ncols = mat.cols%2==0?mat.cols:mat.cols-1;

Mat img = Mat((nrows-1)/2, (ncols-1)/2, CV_8UC1);

for (int i = 1; i < nrows - 1; i += 2) {
	for (int j = 1; j < ncols - 1;  j += 2) {
		img.at<uchar>(i / 2, j / 2) = mat.at<uchar>(i, j);
	}
}
```

# opencv基础操作及图像的卷积与滤波

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
计算机中的图像本质上是一个矩阵, 下图为单位矩阵*255所形成的图像

![nature of image][nature of image]  

Mat是用于保存图像矩阵的数据结构
1. 单通道  
二值图像 0 255  
灰度图 0-255
2. 3通道  
RGB
3. 4通道  
RGBA


## 图像处理数学原理  

[图像处理数学原理][yuanli]   

### 简易浮雕效果  

对矩阵横向求差分
```c++
cvtColor(mat, mat, CV_RGB2GRAY);

Mat img = Mat(mat.rows, mat.cols-2, CV_8UC1);
for (int i = 0; i < mat.rows; i++) {
	for (int j = 1; j < mat.cols - 1; j++) {
		img.at<uchar>(i, j-1) = mat.at<uchar>(i, j - 1) - mat.at<uchar>(i, j + 1);
	}
}
```

使用卷积对矩阵求差分

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

效果  

![difference][difference]  

可见光线变化越强烈的地方越明显,这是由于这些地方对应矩阵数值差分数值差异较大

### 高斯模糊

高斯模糊的原理是将正态分布应用于图像处理 

一维正态分布曲线:  
![normal distribution curve][normal distribution curve]  

由于图像为二维,所以需要二维正态分布,即高斯分布:  

![Gaussian distribution][Gaussian distribution]  

高斯分布公式:  

![Gaussian function][Gaussian function]

创建高斯核
```c++
cvtColor(frame, frame, CV_RGB2GRAY);

double sigma = 5000;
Mat gauss = Mat(5, 5, CV_64FC1);
for (int i = -2; i < 3; i++) {
	for (int j = -2; j < 3; j++) {
		gauss.at<double>(i + 2 , j + 2) =  exp(-(i * i + j * j) / (2 * sigma * sigma));
	}
}
```
归一化
```c++
double gssum = sum(gauss).val[0];//求总和·
cout << gssum << endl;
for (int i = -2; i < 3; i++) {
	for (int j = -2; j < 3; j++) {
		gauss.at<double>(i + 2, j + 2)/= gssum;
	}
}
```
卷积
```c++
Mat dimg = Mat(frame.rows - 4, frame.cols - 4, CV_8UC1);
for (int i = 2; i < frame.rows - 2; i++) {
	for (int j = 2; j < frame.cols - 2; j++) {
		int half = gauss.cols / 2;
		double sum = 0;
		for (int m = 0; m < gauss.rows; m++) {
			for (int n = 0; n < gauss.cols; n++) {
				sum += (double)(frame.at<uchar>(i + m - 2, j + n - 2))*gauss.at<double>(m, n);
			}
		}
		dimg.at<uchar>(i - 2, j - 2) = (uchar)sum;
	}
}
```



### 升采样

升采样就是图像的拉伸,最简单的方法是计算相邻像素值的平均并插进中间  

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
降采样即图像的缩小,只需按缩小的比例选取对应比例的像素即可
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

[nature of image]:http://oh1zr9i3e.bkt.clouddn.com/public/16-11-26/13922823.jpg  
[normal distribution curve]:http://oh1zr9i3e.bkt.clouddn.com/public/16-11-26/87303803.jpg  
[Gaussian distribution]:http://oh1zr9i3e.bkt.clouddn.com/public/16-11-26/87347346.jpg
[Gaussian function]:http://oh1zr9i3e.bkt.clouddn.com/public/16-11-26/83043598.jpg
[yuanli]:http://www.cnblogs.com/cnzhao/p/5954475.html
[difference]:http://oh1zr9i3e.bkt.clouddn.com/public/16-11-27/41959627.jpg

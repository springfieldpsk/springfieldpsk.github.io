---
title: opencv灯光捕捉
date: 2022-04-10 21:46:35
tags:
---
## 思路：

对图像进行二值化处理，滤波、腐蚀，生成可用二值图，对该二值图进行连通域计算，获取屏幕中心看亮度区域大小，以该亮度区域大小作为参考，统计亮度。

## 流程

1. 获取导入图像
2. 对导入图像进行后处理
3. 后处理图像进行连通域统计
4. 获取屏幕中心统计信息
5. 将统计结果写入json文件

<!-- more -->

## 依赖

1. opencv
2. nlohmann_json
3. cmake

## 代码

```cpp
// main.cpp
#include <iostream>
#include <fstream>
#include <queue>
#include "opencv2/opencv.hpp"
#include "GetBrightness.h"
#include "nlohmann/json.hpp"
#include <unordered_map>

using namespace cv;
using json = nlohmann::json;

// 外部参数解析器
void ParseArgv(int argc, char *argv[], std::unordered_map<std::string, std::string> &map)
{
    for (int i = 0; i < argc; i++)
    {
        int len = strlen(argv[i]);
        std::string ParamName;
        std::string ParamValue;
        bool ChangeMode = false;
        for (int j = 0; j < argv[i][j]; j++)
        {
            if (argv[i][j] == '=')
            {
                ChangeMode = true;
                continue;
            }
            if (ChangeMode)
                ParamValue.push_back(argv[i][j]);
            else
                ParamName.push_back(argv[i][j]);
        }
        map[ParamName] = ParamValue;
    }
}

int main(int argc, char *argv[])
{
    // 解析外部参数
    std::unordered_map<std::string, std::string> argvMap;
    ParseArgv(argc, argv, argvMap);
    // std::cout<< argvMap["ImagePath"] << std::endl;
    if (!argvMap.count("ImagePath") || !argvMap.count("JsonPath"))
    {
        return 0;
    }

    std::string ImagePath = argvMap["ImagePath"];
    std::string JsonPath = argvMap["JsonPath"];
    int MaxCnt = 5;
    if (argvMap.count("MaxCnt"))
        MaxCnt = std::stoi(argvMap["MaxCnt"]);

    GetBrightness *Brightness_Ptr = GetBrightness::GetBrightnessFactory();

    // 处理图像
    Mat frame;
    std::priority_queue<node> BrightNessQue;

    frame = imread(ImagePath);
    Brightness_Ptr->mainFrame = frame;
    Brightness_Ptr->PostProcess();
    int Brightness = Brightness_Ptr->GetCenterBrightness();

    // 写入Json中
    json itemJson{
        {"BrightNess", Brightness},
        {"rows",Brightness_Ptr->mainFrame.rows},
        {"cols",Brightness_Ptr->mainFrame.cols}
    };

    std::ofstream(JsonPath) << itemJson;
    return 0;
}

// GetBrightness.h
#pragma once
#include "string"
#include "queue"
#include "opencv2/opencv.hpp"

struct node
{
	int x, y, Brightness;

	node() {}
	node(int _x, int _y, int _Brightness) :x(_x), y(_y), Brightness(_Brightness) {}

	bool operator < (node a) const {
		if (Brightness != a.Brightness) return Brightness > a.Brightness;
		if (x != a.x) return x < a.x;
		return y < a.y;
	}
};

class GetBrightness
{
public:

	GetBrightness(){}
	
	cv::Mat mainFrame;
	cv::Mat RawFrame;

	static GetBrightness* GetBrightnessFactory();

	void PostProcess();
	void GetTopBrightness(std::priority_queue<node>& BrightnessQueue,int MaxSize);
	int GetCenterBrightness();
};

// GetBrightness.cpp
#include "GetBrightness.h"

GetBrightness* GetBrightness::GetBrightnessFactory() {
	GetBrightness* BrightnessPtr = new GetBrightness();
	return BrightnessPtr;
}

void GetBrightness::PostProcess()
{
	RawFrame = mainFrame;
	// 转为灰度图
	cv::cvtColor(mainFrame, mainFrame, cv::COLOR_BGR2GRAY);

	// 利用高斯滤波器滤波, 减少高频噪声
	GaussianBlur(mainFrame, mainFrame, cv::Size(11, 11), 0);

	// 将阈值设为 200 , 最大阈值结果设置为 255 (白色) 以二值化方式进行阈值化处理
	double ThresValue = 220;
	double MaxThres = 255;

	threshold(mainFrame, mainFrame, ThresValue, MaxThres, cv::ThresholdTypes::THRESH_BINARY);

	// 通过膨胀与腐蚀清除图像噪声
	// 生成一个矩形大小为3*3的核
	cv::Mat Element = cv::getStructuringElement(cv::MORPH_ELLIPSE, cv::Size(3, 3));
	morphologyEx(mainFrame, mainFrame, cv::MORPH_OPEN, Element);
	morphologyEx(mainFrame, mainFrame, cv::MORPH_CLOSE, Element);
}

void GetBrightness::GetTopBrightness(std::priority_queue<node>& BrightnessQueue,int MaxSize)
{
	// 输出标记图像
	cv::Mat labels;
	// stats 统计信息，包括每个组件的位置、宽、高与面积
	// centroid 组件中心位置
	cv::Mat stats, centroid;

	// 连通域计算 设置格式为CV_32S 连通域为8
	int num_labels = connectedComponentsWithStats(mainFrame, labels, stats, centroid, 8, CV_32S);
	
	for (int i = 1; i < num_labels; i++) {
		// 获取连通域中点位置 区域大小 
		cv::Vec2d pt = centroid.at<cv::Vec2d>(i, 0);
		int Area = stats.at<int>(i, cv::CC_STAT_AREA);
		// 压入堆中
		BrightnessQueue.push(node((int)pt[0], (int)pt[1], Area));
		if (BrightnessQueue.size() > MaxSize) BrightnessQueue.pop();
	}
}

int GetBrightness::GetCenterBrightness(){
	// 输出标记图像
	cv::Mat labels;
	// stats 统计信息，包括每个组件的位置、宽、高与面积
	// centroid 组件中心位置
	cv::Mat stats, centroid;

	// 连通域计算 设置格式为CV_32S 连通域为8
	int num_labels = connectedComponentsWithStats(mainFrame, labels, stats, centroid, 8, CV_32S);
	
	double x = mainFrame.cols/2.0;
	double y = mainFrame.rows/2.0;

	int label = labels.at<int>(y,x);
	return stats.at<int>(label, cv::CC_STAT_AREA);
}
```
---
title: ORB-SLAM2源码分析一
date: 2019-05-28 09:51:42
tags: 
- SLAM
- Tracking
categories: ORB-SLAM2
---

## **主函数说明 mono_kitty.cc**
###1. *main* 入口函数，读取3个文件参数，初始化系统 
- strVocFile: 字典词包的路径
- strSettingFile: 系统中装有一些如相机参数、view窗口的配置文件，格式为YAML
- strSequence: 数据集路径

###2. *LoadImages* 函数
 LoadImages(const string &strPathToSequence, vector<string> &vstrImageFilenames, vector<double> &vTimestamps)
 ```
 void LoadImages(const string &strPathToSequence, vector<string> &vstrImageFilenames, vector<double> &vTimestamps)
{
    ifstream fTimes;
    string strPathTimeFile = strPathToSequence + "/times.txt";
    fTimes.open(strPathTimeFile.c_str());
    while(!fTimes.eof())
    {
        string s;
        getline(fTimes,s);
        if(!s.empty())
        {
            stringstream ss;
            ss << s;
            double t;
            ss >> t;
            vTimestamps.push_back(t);
        }
    }

    string strPrefixLeft = strPathToSequence + "/image_0/";

    const int nTimes = vTimestamps.size();
    vstrImageFilenames.resize(nTimes);

    for(int i=0; i<nTimes; i++)
    {
        stringstream ss;
        ss << setfill('0') << setw(6) << i;
        vstrImageFilenames[i] = strPrefixLeft + ss.str() + ".png";
    }
}
```

加载数据集函数，函数执行完vstrImageFileNames是一个存有图片具体位置的vector，位置形式如xxx/xxx/000xxx.png，vTimestamps是存有图片时间戳的vector

---------------------------------
### 3. 实例化 SLAM 系统
加载图片路径完成后，需要实例化一个SLAM系统对象
ORB_SLAM2::System SLAM(argv[1],argv[2],ORB_SLAM2::System::MONOCULAR,true);

System.h 包含了7个类，分别是Viewer， FrameDrawer, Map, Tracking, LocalMapping, LoopClosing 的声明，和System 类的定义， 就像描述的那样，这些类组成了一个系统。

```
class Viewer;
class FrameDrawer;
class Map;
class Tracking;
class LocalMapping;
class LoopClosing;
class System;
```

## 成员变量说明

### 3.1 sensor 枚举
```
enum eSensor{
        MONOCULAR=0,
        STEREO=1,
        RGBD=2
};  
```
0,1,2 分别代表传感器的类型

### 3.2 System 构造函数
System(const string &strVocFile, const string &strSettingsFile, const eSensor sensor, const bool bUseViewer = true);

Monocular System 构造时，读入词包路径，YAML配置文件，设置eSensor类型为Monocular，并启用Viewer线程

### 3.3 Tracking 函数
```
// Proccess the given stereo frame. Images must be synchronized and rectified.
// Input images: RGB (CV_8UC3) or grayscale (CV_8U). RGB is converted to grayscale.
// Returns the camera pose (empty if tracking fails).
cv::Mat TrackStereo(const cv::Mat &imLeft, const cv::Mat &imRight, const double &timestamp);

// Process the given rgbd frame. Depthmap must be registered to the RGB frame.
// Input image: RGB (CV_8UC3) or grayscale (CV_8U). RGB is converted to grayscale.
// Input depthmap: Float (CV_32F).
// Returns the camera pose (empty if tracking fails).
cv::Mat TrackRGBD(const cv::Mat &im, const cv::Mat &depthmap, const double &timestamp);

// Proccess the given monocular frame
// Input images: RGB (CV_8UC3) or grayscale (CV_8U). RGB is converted to grayscale.
// Returns the camera pose (empty if tracking fails).
cv::Mat TrackMonocular(const cv::Mat &im, const double &timestamp);
```
针对不同传感器不同的Tracking。输入图像可以使rgb的也可以是grayscale的（最终读进去都会转化为grayscale的），函数返回值为camera的位姿pose。Tracking 过程是对针对每一幅图像，通过先初始化然后track和优化过程来估计相机误差。

### 3.4 定位模式函数
```
// This stops local mapping thread (map building) and performs only camera tracking.
void ActivateLocalizationMode();
// This resumes local mapping thread and performs SLAM again.
void DeactivateLocalizationMode();
```
调用ActivateLocalizationMode()将终止mapping线程，开启定位模式，调用后者重启mapping线程。

### 3.5 重启与终止函数
```
// Reset the system (clear map)
void Reset();

// All threads will be requested to finish.
// It waits until all threads have finished.
// This function must be called before saving the trajectory.
void Shutdown();
```
Reset()函数将清空map，Shutdown()函数可以终止所有线程，在保存相机轨迹之前需要调用此函数。

### 3.6 SaveTrajectory 函数
```
    // Save camera trajectory in the TUM RGB-D dataset format.
    // Only for stereo and RGB-D. This method does not work for monocular.
    // Call first Shutdown()
    // See format details at: http://vision.in.tum.de/data/datasets/rgbd-dataset
    void SaveTrajectoryTUM(const string &filename);

    // Save keyframe poses in the TUM RGB-D dataset format.
    // This method works for all sensor input.
    // Call first Shutdown()
    // See format details at: http://vision.in.tum.de/data/datasets/rgbd-dataset
    void SaveKeyFrameTrajectoryTUM(const string &filename);

    // Save camera trajectory in the KITTI dataset format.
    // Only for stereo and RGB-D. This method does not work for monocular.
    // Call first Shutdown()
    // See format details at: http://www.cvlibs.net/datasets/kitti/eval_odometry.php
    void SaveTrajectoryKITTI(const string &filename);

    // TODO: Save/Load functions
    // SaveMap(const string &filename);
    // LoadMap(const string &filename);

```
把相机轨迹保存成相应数据集的格式，系统调用此函数时先shutdown SLAM系统，mono_kittti中save函数用的是SaveKeyFrameTrajectoryTUM，这个函数看起来像是只能用于TUM数据集，但三种传感器均适合。

### 4 System private 成员变量说明
### 4.1 eSensor
```
// Input sensor
eSensor mSensor;
```
输入的传感器类型

### 4.2 mpVocabulary
```
// ORB vocabulary used for place recognition and feature matching.
ORBVocabulary* mpVocabulary;
```
用于位置识别和特征匹配的系统词包

### 4.3 mpKeyFrameDatabase
```
// KeyFrame database for place recognition (relocalization and loop detection).
KeyFrameDatabase* mpKeyFrameDatabase;
```
用于位置识别，重定位，回环检测的关键帧数据集

### 4.4 mpMap 
```
// Map structure that stores the pointers to all KeyFrames and MapPoints.
Map* mpMap;
```
存储系统关键帧的指针和地图点的指针

### 4.5 mpTracker
```
// Tracker. It receives a frame and computes the associated camera pose.
// It also decides when to insert a new keyframe, create some new MapPoints and
// performs relocalization if tracking fails.
Tracking* mpTracker;
```
Tracker 接受一帧图像并计算相机位姿，决定什么时候需要插入关键帧，创建地图点并且执行重定位如果跟踪失败。

### 4.6 mpLocalMapper
```
// Local Mapper. It manages the local map and performs local bundle adjustment.
LocalMapping* mpLocalMapper;
```
局部地图管理器，mpLocalMapper，管理局部地图并进行局部BA。

### 4.7 mpLoopCloser
```
// Loop Closer. It searches loops with every new keyframe. If there is a loop it performs
// a pose graph optimization and full bundle adjustment (in a new thread) afterwards.
LoopClosing* mpLoopCloser;
```
回环检测器，每次获取关键帧后都会进行回环检测，如果存在回环的话就执行位姿图的优化并且进行全局BA优化

### 4.8 mpViewer,mpFrameDrawer,mpMapDrawer
```
// The viewer draws the map and the current camera pose. It uses Pangolin.
Viewer* mpViewer;

FrameDrawer* mpFrameDrawer;
MapDrawer* mpMapDrawer;
```
视图显示

### 4.9 系统线程
```
// System threads: Local Mapping, Loop Closing, Viewer.
// The Tracking thread "lives" in the main execution thread that creates the System object.
std::thread* mptLocalMapping;
std::thread* mptLoopClosing;
std::thread* mptViewer;
```

### 4.10 Reset flag
```
// Reset flag
std::mutex mMutexReset;
bool mbReset;
```

### 4.11 Change mode flags
```
// Change mode flags
std::mutex mMutexMode;
bool mbActivateLocalizationMode;
bool mbDeactivateLocalizationMode;
```

### 5. 实例化SLAM-System构造函数
```
System::System(const string &strVocFile, const string &strSettingsFile, const eSensor sensor,
               const bool bUseViewer):mSensor(sensor),mbReset(false),mbActivateLocalizationMode(false),
               mbDeactivateLocalizationMode(false)
```
System 构造函数用于实例化一个SALM系统，开启相机跟踪(Tracking)，局部建图(Local Mapping)，回环检测(Loop Closing)，和可视化界面(Viewer)的线程。

### 5.1 初始形参传递
```
mSensor(sensor),
mbReset(false),
mbActivateLocalizationMode(false),
mbDeactivateLocalizationMode(false)
```
sensor是传进来的形参，是前面枚举体中三种传感器的一个，这里为MONOCULAR，它传递给了mSensor，这是一个System类的隐含成员变量，两种变量类型一样。
mpViewer是System类的隐含成员变量，Viewer类指针，这里赋空。
mbReset，mbActivateLocalizationMode，mbDeactivateLocalizationMode均为bool型，赋false。

### 5.2 初始化数据库
- 1 初始化词包 ***mpVocabulary***
```
//Load ORB Vocabulary
    cout << endl << "Loading ORB Vocabulary. This could take a while..." << endl;

    mpVocabulary = new ORBVocabulary();
    bool bVocLoad = false; // chose loading method based on file extension
    if (has_suffix(strVocFile, ".txt"))
	  bVocLoad = mpVocabulary->loadFromTextFile(strVocFile);
	else if(has_suffix(strVocFile, ".bin"))
	  bVocLoad = mpVocabulary->loadFromBinaryFile(strVocFile);
	else
	  bVocLoad = false;
    if(!bVocLoad)
    {
        cerr << "Wrong path to vocabulary. " << endl;
        cerr << "Failed to open at: " << strVocFile << endl;
        exit(-1);
    }
    cout << "Vocabulary loaded!" << endl << endl;
```
- 2 用**词包数据库**来初始化关键帧数据库（用于**重定位**和**回环检测**）***mpKeyFrameDatabase***
```
//Create KeyFrame Database
    mpKeyFrameDatabase = new KeyFrameDatabase(*mpVocabulary);
```
- 3 初始化一个Map类对象 ，该类用于存储指向所有**关键帧**和**地图点**的指针 ***mpMap***
```
//Create the Map
    mpMap = new Map();
```
- 4 初始化画图工具，用于可视化 ***mpFrameDrawer***、***mpMapDrawer***
```
//Create Drawers. These are used by the Viewer
    mpFrameDrawer = new FrameDrawer(mpMap);
    mpMapDrawer = new MapDrawer(mpMap, strSettingsFile);
```
- 5 初始化Tracking线程，**主线程**，使用this指针（只初始化不启动，启动在main函数里TrackMonocular()启动）
```
//Initialize the Tracking thread
//(it will live in the main thread of execution, the one that called this constructor)
    mpTracker = new Tracking(this, mpVocabulary, mpFrameDrawer, mpMapDrawer,
                             mpMap, mpKeyFrameDatabase, strSettingsFile, mSensor);
```
- 6 初始化Local Mapping线程并启动（这里mSensor传入MONOCULAR）***mpLocalMapper***
```
 //Initialize the Local Mapping thread and launch
    mpLocalMapper = new LocalMapping(mpMap, mSensor==MONOCULAR);
    mptLocalMapping = new thread(&ORB_SLAM2::LocalMapping::Run,mpLocalMapper);
```
- 7 初始化Loop Closing线程并启动（这里mSensor传入的不是MONOCULAR）***mptLoopClosing***
```
    //Initialize the Loop Closing thread and launch
    mpLoopCloser = new LoopClosing(mpMap, mpKeyFrameDatabase, mpVocabulary, mSensor!=MONOCULAR);
    mptLoopClosing = new thread(&ORB_SLAM2::LoopClosing::Run, mpLoopCloser);
```
- 8 初始化Viewer线程并启动，也使用了this指针；给Tracking线程设置Viewer
```
//Initialize the Viewer thread and launch
    mpViewer = new Viewer(this, mpFrameDrawer,mpMapDrawer,mpTracker,strSettingsFile);
    if(bUseViewer)
        mptViewer = new thread(&Viewer::Run, mpViewer);

    mpTracker->SetViewer(mpViewer);
```

- 9 ***mpTracker***，***mpLocalMapper***，***mptLoopClosing***三个线程每两个线程之间设置指针相互关联
```
 //Set pointers between threads
    mpTracker->SetLocalMapper(mpLocalMapper);
    mpTracker->SetLoopClosing(mpLoopCloser);

    mpLocalMapper->SetTracker(mpTracker);
    mpLocalMapper->SetLoopCloser(mpLoopCloser);

    mpLoopCloser->SetTracker(mpTracker);
    mpLoopCloser->SetLocalMapper(mpLocalMapper);
```

### 6. 循环Tracking

```
    // Main loop
    cv::Mat im;
    for(int ni=0; ni<nImages; ni++)
    {
        // Read image from file
        im = cv::imread(vstrImageFilenames[ni],CV_LOAD_IMAGE_UNCHANGED);
        double tframe = vTimestamps[ni];

        if(im.empty())
        {
            cerr << endl << "Failed to load image at: " << vstrImageFilenames[ni] << endl;
            return 1;
        }

#ifdef COMPILEDWITHC11
        std::chrono::steady_clock::time_point t1 = std::chrono::steady_clock::now();
#else
        std::chrono::monotonic_clock::time_point t1 = std::chrono::monotonic_clock::now();
#endif

        // Pass the image to the SLAM system
        SLAM.TrackMonocular(im,tframe);

#ifdef COMPILEDWITHC11
        std::chrono::steady_clock::time_point t2 = std::chrono::steady_clock::now();
#else
        std::chrono::monotonic_clock::time_point t2 = std::chrono::monotonic_clock::now();
#endif

        double ttrack= std::chrono::duration_cast<std::chrono::duration<double> >(t2 - t1).count();

        vTimesTrack[ni]=ttrack;

        // Wait to load the next frame
        double T=0;
        if(ni<nImages-1)
            T = vTimestamps[ni+1]-tframe;
        else if(ni>0)
            T = tframe-vTimestamps[ni-1];

        if(ttrack<T)
            this_thread::sleep_for(std::chrono::microseconds((int)((T-ttrack)*1e6)));
    }

    // Stop all threads
    SLAM.Shutdown();
```

上述分为两步：读图、Tracking，其中有一部分代码（注释 //Wait to load the next frame 后）目的是为了模拟真实时间状况，如果tracking过快，则下一帧可能还没来，所以要“睡” T-ttrack 秒等待装载下一帧图片。每次tracking只处理一帧图片。

 



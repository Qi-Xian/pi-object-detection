# Pi Object Detection
在樹莓派上執行即時物件偵測，採用webcam輸入影像，並於螢幕顯示結果。

# 架構
OpenCV負責讀取webcam的影像串流，將即時圖像交給TensorFlow Object Detection執行物件偵測，並藉其提供的工具把偵測結果繪製到原始圖像上，最後在OpenCV的視窗顯示。其中TensorFlow Object Detection包含了深度學習模型和已訓練好的權重，能輸入影像張量，輸出邊框和偵測結果。

- Raspberry Pi 3 (with 2017-11-29-raspbian-stretch)
- Camera (SJ4000)
- Python (3.5)
- OpenCV (3.1)
- TensorFlow (1.1.0)
- TensorFlow Object Detection

# 安裝

## 注意事項

- 用SSＨ來安裝會遇到麻煩，以下所有操作都在本機執行

## 準備 TensorFlow

### 更新環境
```shell
    sudo apt-get update
    sudo apt-get upgrade
```


### 安裝
```shell
    sudo apt-get install python3-pip python3-dev
    wget https://github.com/samjabrahams/tensorflow-on-raspberry-pi/releases/download/v1.1.0/tensorflow-1.1.0-cp34-cp34m-linux_armv7l.whl
    # 更名以避免版本錯誤 (Could not find a version that satisfies the requirement tensorflow ...)
    sudo mv tensorflow-1.1.0-cp34-cp34m-linux_armv7l.whl tensorflow-1.1.0-cp35-cp35m-linux_armv7l.whl
    sudo pip3 install tensorflow-1.1.0-cp35-cp35m-linux_armv7l.whl
    sudo pip3 uninstall mock
    sudo pip3 install mock
```

### 測試
```python
    # In python3 shell
    import tensorfolow as tf
    tf.__version__
```


## 準備 TensorFlow Object Detection

### 安裝環境
```shell
    # 安裝 protobuf-compiler  pillow  lxml  jupyter  matplotlib
    sudo apt-get install protobuf-compiler
    sudo pip3 install pillow

    # 解決未知問題（TypeError: unsupported operand type(s) for -= 'Retry' and 'int' ...）
    sudo aptitude install 

    sudo pip3 install lxml
    sudo pip3 install jupyter
    sudo pip3 install matplotlib

    # 避免numpy版本問題
    sudo pip3 install numpy==1.13
    sudo aptitude install
    sudo pip3 install cairocffi
```

### 安裝模組
```shell
    # 從Github下載
    cd ~
    git clone https://github.com/tensorflow/models.git

    # 進入research目錄
    cd ./models/research

    # 處理Google Protocol Buffer，將路徑加到PYTHONPATH
    sudo protoc object_detection/protos/*.proto --python_out=.
    export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim

    # 測試，如果顯示ＯＫ代表成功
    python3 object_detection/builders/model_builder_test.py

    # 安裝Python模組
    python3 setup.py build
    sudo python3 setup.py install
```

### 測試模組
```python
    # In python3 shell
    from object_detection.utils import label_map_util
    from object_detection.utils import visualization_utils as vis_util
    label_map_util
    vis_util
```


### 修改.bashrc 使得每次開機都可以生效
```shell
    sudo nano ~/.bashrc
    # 在最後方加入: export PYTHONPATH=$PYTHONPATH:/home/pi/models/research:/home/pi/models/research/slim
```

## 準備 OpenCV

### 更新環境
```shell
    sudo apt-get update
    sudo apt-get upgrade
    sudo rpi-update
    sudo reboot
```

### 安裝環境
```shell
    sudo apt-get install build-essential git cmake pkg-config
    sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
    sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
    sudo apt-get install libxvidcore-dev libx264-dev
    sudo apt-get install pkg-config
    sudo apt-get install libgtk2.0-dev
    sudo apt-get install libatlas-base-dev gfortran
```

### 下載原始碼
```shell
    # 下載opencv，並切換Branch到3.1版
    cd ~
    git clone https://github.com/Itseez/opencv.git
    cd opencv
    git checkout 3.1.0
    
    # 下載opencv_contrib，並切換Branch到3.1版
    cd ~
    git clone https://github.com/Itseez/opencv_contrib.git
    cd opencv_contrib
    git checkout 3.1.0  
```

### 編譯
```shell
    cd ~/opencv
    mkdir build
    cd build
    sudo cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D INSTALL_C_EXAMPLES=OFF \
        -D INSTALL_PYTHON_EXAMPLES=ON \
        -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
        -D BUILD_EXAMPLES=ON \
        -D ENABLE_PRECOMPILED_HEADERS=OFF ..

    # 筆者用四線程編譯當機很多次，如果遇到一樣的問題可以嘗試改用單線程就好
    sudo make -j4
```

### 安裝
``` shell
    sudo make install
    sudo ldconfig
```

### 測試
```python
    # In python3 shell
    import cv2
    cv2.__version__
```

## Intel Movidius Neural Compute Stick (NCS)
原本打算一步步完成TensorFlow、Opencv、NCS，但到最後發現NCS的用法和預期不同，所以會再另行研究。

# 實驗
1. [OpenCV 顯示Webcam即時影像](/lab/opencv-webcam.ipynb)
2. [TensorFlow Object Detection 單張影像物件偵測](/lab/tensorflow-object-detection-image.ipynb)
3. [OpevCV + Object Detection 持續串流物件偵測](/lab/tensorflow-object-detection-opencv-stream.ipynb)

# 心得
在持續串流物件偵測的實驗中，FPS非常的低，好幾秒才能處理一張照片，所以這個架構在PI上是不實用的，期待PI + NCS能夠達到即時的物件偵測。

# 參考
- CH.Tseng: [在樹莓派上運行object-detection-api](https://chtseng.wordpress.com/2017/09/15/%E5%9C%A8%E6%A8%B9%E8%8E%93%E6%B4%BE%E4%B8%8A%E9%81%8B%E8%A1%8Cobject-detection-api/)
- Theta: [Object Detection on a Raspberry Pi](https://www.theta.co.nz/news-blogs/tech-blog/object-detection-on-a-raspberry-pi/)
- Github: [samjabrahams/tensorflow-on-raspberry-pi](https://github.com/samjabrahams/tensorflow-on-raspberry-pi)
- Github: [lhelontra/tensorflow-on-arm](https://github.com/lhelontra/tensorflow-on-arm)
- Github: [tensorflow/models (Object Detection)](https://github.com/tensorflow/models/tree/master/research/object_detection)
- Github: [opencv/Build problems on the raspberry pi 3 : make: *** [all] Error 2](https://github.com/opencv/opencv/issues/8878)
- Movidius: [ncs-apps-on-rpi](https://movidius.github.io/blog/ncs-apps-on-rpi/)
# Note: this repository is not completed yet

<br>


# FAST-LIO-Localization-QN
+ This repository is a map-based localization implementation combining [FAST-LIO2](https://github.com/hku-mars/FAST_LIO) as an odometry with [Quatro](https://github.com/engcang/Quatro) and [Nano-GICP module](https://github.com/engcang/nano_gicp) as a map matching method
    + [Quatro](https://github.com/engcang/Quatro) - fast, accurate and robust global registration which provides great initial guess of transform
    + [Nano-GICP module](https://github.com/engcang/nano_gicp) - fast ICP combining [FastGICP](https://github.com/SMRT-AIST/fast_gicp) + [NanoFLANN](https://github.com/jlblancoc/nanoflann)
+ Note: similar repositories already exist
    + [FAST_LIO_LOCALIZATION](https://github.com/HViktorTsoi/FAST_LIO_LOCALIZATION): FAST-LIO2 + Open3D's ICP
+ Note2: main code is modularized and hence can be combined with any other LIO / LO
+ Note3: this repo is to apply `Quatro` in localization. `Quatro` can be worked in scan-to-scan matching or submap-to-submap matching but not scan-to-submap, i.e., ***the numbers of pointclouds to be matched should be similar.***
+ Note4: **saved map file** is needed. The map should be in `.bag` format. This `.bag` files can be built with [FAST-LIO-SAM-QN](https://github.com/engcang/FAST-LIO-SAM-QN) and [FAST-LIO-SAM](https://github.com/engcang/FAST-LIO-SAM)

<br>

## Dependencies
+ `C++` >= 17, `OpenMP` >= 4.5, `CMake` >= 3.10.0, `Eigen` >= 3.2, `Boost` >= 1.54
+ `ROS`
+ [`Teaser++`](https://github.com/MIT-SPARK/TEASER-plusplus)
    ```shell
    git clone https://github.com/MIT-SPARK/TEASER-plusplus.git
    cd TEASER-plusplus && mkdir build && cd build
    cmake .. -DENABLE_DIAGNOSTIC_PRINT=OFF
    sudo make install -j16
    sudo ldconfig
    ```

## How to build and use
+ Get the code, build `tbb` first, and then build the main code. `tbb` is used for faster `Quatro`
    ```shell
    cd ~/your_workspace/src
    git clone https://github.com/engcang/FAST-LIO-Localization-QN --recursive

    # Note the directory
    cd FAST-LIO-Localization-QN/third_party/oneTBB && mkdir build installed
    cd build
    # Note the directory
    cmake .. -DCMAKE_INSTALL_PREFIX=../installed -DTBB_TEST=OFF -DCMAKE_BUILD_TYPE=Release
    make -j16 && make install

    cd ~/your_workspace
    # nano_gicp, quatro first
    catkin build nano_gicp -DCMAKE_BUILD_TYPE=Release
    # Note the option!
    catkin build quatro -DCMAKE_BUILD_TYPE=Release -DQUATRO_TBB=ON
    catkin build -DCMAKE_BUILD_TYPE=Release
    . devel/setup.bash
    ```
+ Then run
    ```shell
    roslaunch fast_lio_localization_qn run.launch lidar:=ouster
    roslaunch fast_lio_localization_qn run.launch lidar:=velodyne
    roslaunch fast_lio_localization_qn run.launch lidar:=livox
    ```
+ Change config files in
    + third_party/`FAST_LIO`/config
    + fast_lio_localization_qn/config

<br>

## Structure
+ odom_pcd_cb
    + pub realtime pose in corrected frame
    + keyframe detection -> if keyframe, add to pose graph + save to keyframe queue
+ matching_timer_func
    + process a saved keyframe
        + detect map match -> if matched, correct TF
+ vis_timer_func
    + visualize all
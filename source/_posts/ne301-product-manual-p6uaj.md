---
title: NE301产品手册
date: '2025-10-17 10:48:16'
updated: '2025-10-17 11:31:59'
excerpt: ——NE301产品详情
tags:
  - NE301
categories:
  - 产品介绍
permalink: /post/ne301-product-manual-p6uaj.html
comments: true
toc: true
---





# NE301——边缘人工智能摄像头

——为工业边缘应用提供强大而灵活的解决方案  
NE301搭载STMS2N6（Cortex M55）处理器和“NeuralART”NPU，提供实时人工智能推理和专业图像处理，同时实现极低功耗。  
借助“Arm Helium”矢量处理技术，该设备具备灵活的硬件接口、强大的工业连接性以及开源生态系统，为边缘人工智能驱动的视觉应用提供可扩展且模块化的平台。

‍

- 边缘人工智能处理:STM32N6（Cortex M55）配备“NeuralART”NPU，提供0.6 TOPS的算力，搭配256MB RAM，支持实时视觉和音频人工智能，支持基于YOLO的视觉人工智能和YAMNet-1024语音人工智能。
- 优质高效的图像处理:内置ISP，支持H.264 1080p@30fps编码、JPEG压缩、MIPI CSI2/USB相机接口，实现高保真的实时成像。
- 丰富的硬件接口与模块化选择:16引脚GPIO、UART、RS485、SPI、I2C，以及可选的Cat1通信方式、多样化相机模组与规格，以及多种安装部署方式。
- 开源性：提供开源支持，支持STM32Cube AI、TensorFlow Lite、ONNX（PyTorch/MATLAB）、REST API，实现无缝的人工智能模型部署。
- 坚固可靠：IP67级防护，工作温度范围为-20°C至50°C，专为工业和户外人工智能部署设计。

‍

# 亮点——开源就绪，工业级性能

- 即时启动  
  冷启动时间小于1毫秒，实时AI推理速度可达25fps
- 高效性能  
  NPU效率达到3TOPS/W，配备优化的散热设计
- 友好交互

  提供WebUI进行交互，零基础上手，配备完整的WIKI教程，快速实现AI功能调试与即时验证
- 即插即用本地AI模型  
  支持预训练的STM32模型库，支持TensorFlow Lite、Keras、ONNX（PyTorch、MATLAB），内置YOLO模型实例以及完善的资料指引
- 灵活部署  
  支持电池、USB-C和PoE供电，适用于多种应用场景
- 模块化扩展  
  工业级接口，包括16pin引脚、UART、RS485、SPI、Wafer、GND等，提供扩展连接性
- 丰富的固件功能  
  内置MQTT、HTTP、各类触发控制、硬件监控、模型管理、日志记录等

# NE301硬件规格

‍

|中文版|||
| ------------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------| -------------------------------------------------------------------------------------------------------------------------------------------------|
|MCU<br />|Core|Cortex-M55，主频800 MHz，支持Arm Helium矢量处理技术|
||NPU|集成Neural-ART™加速器，运行频率1 GHz，AI算力高达600 GOPS（0.6 TOPS），支持实时神经网络推理|
||SRAM|4.2 MB|
||ISP Image Processor|内置专用ISP，具备去马赛克、自动白平衡等预处理功能|
||Video Codec|H.264 硬件编码器和 JPEG 编解码器，支持 1080p@30fps 视频处理<br />拍到的原始视频流压缩成 H.264 格式，进行存储或网络传输拍到的图像压缩成 JPEG 存储<br />|
||能效指标|NPU能效达3 TOPS/W，全速运行无需散热|
||启动/唤醒速度|微秒级启动（\<1 ms），毫秒级唤醒（\<10 ms）|
|主板<br />|HyperFlash|128 MB|
||PSRAM|64 MB|
||按键|复位按键、Boot按键、抓拍/录像按键|
||指示灯|电源指示灯、系统指示灯|
||通讯|WiFi6/BLE|
||镜头模组接口|4Pin USBx1,MIPI CSI-2x1|
||16 Pin IO|UARTx1<br />RS485x1<br />I2Cx1<br />SPIx1<br />GPIOX2<br />3.3Vx1/5Vx1 （供电可控）<br />GNDx2|
||调试与供电|USB Type-c×1，UART 4pin Wafer×1|
||音频输入输出|Audio Input\*1(Wafer) and Audio Output\*1(Wafer)|
||通讯模块扩展接口|12Pin+16Pin IO座子（通讯拓展模块使用）|
||电源接口|2Pin电源座子，可连接电池仓或Type-c供电|
|传感器|镜头模组|4MP OS04C10 Camera Module or USB Camera Module|
|传感器扩展|可通过主板IO或功能扩展模块扩展对接各种PIR、雷达、温湿度等传感器||
|功能模块扩展|通讯模块|Cat-1|
|供电模块|功能扩展模块Module，支持主板16Pin IO扩展功能<br />1、支持POE供电<br />2、支持网口RJ45，带指示灯<br />3、兼容太阳能供电组Type-c对接<br />4、Alarm/GND/RS485/Wafer接口扩展<br />5、16 Pin IO||
|电源输入|5V||
|工作温度|-20°C～50°C||
|工作湿度|0%～95% RH 无冷凝||
|尺寸|77 x 77 x 48 mm||
|认证|CE/FCC/RoHS||

‍

‍

‍

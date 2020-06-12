---
date: 2020-06-12T14:35:00.000Z
layout: post
title: 基于C++/Python的CFD数据接口开发（一）Fluent Mesh
subtitle: Fluent
description: >-
    Fluent Mesh文件的读取
image: >-
  https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559821647/theme6_qeeojf.jpg
optimized_image: >-
  https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559821647/theme6_qeeojf.jpg
category: CFD
tags:
  - CFD
  - Fluent
author: Xiaoxiao
paginate: true
---
## 前言
Fluent是运用最为广泛的通用商业CFD软件，也是我CFD学习生涯接触的第一个CFD软件。长期以来，对Fluent的数据处理基本局限于Tecplot和CFD-Post，在后处理过程中，大量的精力浪费在软件反复的开启和导入导出、GUI操作和纷繁的Colorful Fluid Dynamics中。作为科研工具，我更希望获得对计算数据的充分掌握和提炼，并且回避大量手动操作（人活着就是为了自动化！）。

最早的想法是采用Fluent的journal脚本进行I/O操作，并采用Python进行文本提取。但是存在两个问题：
* 1、journal文件(.jnl)包括TUI/GUI两类，通常我们在FluentGUI界面直接录制的journal是GUI语言。Fluent自从17版本以后全面贯彻落实了母公司ANSYS“科技源于换壳”的崭新理念（大误），基本上隔一个版本就换一套GUI，因此GUI型的journal文件跨版本往往不能互通。TUI学习成本比较高，而且这个语言本质上还是像GUI语言那样一层一层标签往下找，使用体验还是类似软件界面的点击而非CFD模型的建立，相比之下Abaqus和ls-dyna一类有限元软件采用的keywords形式的脚本反而比较友好。而且按照以前阅读UDF手册和理论书册的经验，遇到冷门一些的功能估计手册还是讲不清，因此直接放弃。 
* 2、每次开启Fluent都要浪费大量的时间启动证书和读取case和data，并占用大量的内存，甚至由于证书的莫名BUG存在启动失败的可能，可靠性并不高。对于没有Fluent的机器，还需要进行安装。总之，这种方式不太适合团队形式的CFD计算及处理（除非你热衷于帮助每个课题组成员排雷）。

本文主要总结Fluent User's Guide(Solution Mode)附录B1中.msh文件的格式规范，并尝试对ascii格式的.msh文件进行读取。由于主要工作集中在I/O操作，Python相比C++性能差异不大, 因此先采用Python。但是考虑到后续对数据灵活处理的需求，不排除采用C++重写或C++/Python混合编程形式的可能。

## .msh格式解读
### 注释(Comment)
括号内编号为0时，后跟注释文本。
```
 (0 "comment text") 
```
 ### 文件头(Header)
括号内编号为1时，后跟文件头，用于指明文件来源和处理方式。
```
 (1 "fluent20.1.0 build-id: 0") 
```
 PS: 官方文档似乎把1写错成了0...
### 维度(Dimensions)
括号内编号为2时，后跟维度ND，取2或3。
```
 (2 ND) 
```
### 节点(Nodes)
括号内编号为10时，表示节点集。`zone-id`为域ID，zone-id=0时，表面当前几何下的节点为计算域中的全部节点；first-index和last-index分别为起始节点编号（一般为1）和结尾节点编号，为十六进制；type默认为1；ND是维度，取2或3。
这是一个很典型的面向C语言的文本，因为原生的C没有类似C++中vector容器的动态数组，读取不确定量的数据前需要明确数据量来申请动态内存。文档也指出结尾节点编号必须大于或等于声明的节点数，否则会导致数组下标越界导致动态内存耗尽。
```
(10 (zone-id first-index last-index type ND)(
   x1 y1 z1
   x2 y2 z2
   .
   .
   .
   )) 
```
示例见下
```
  (10 (1 1 2d5 1 2)(
   1.500000e-01 2.500000e-02
   1.625000e-01 1.250000e-02
     .
     .
     .
   1.750000e-01 0.000000e+00
   2.000000e-01 2.500000e-02
   1.875000e-01 1.250000e-02
   )) 
```
  
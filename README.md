## 错误监控系统 (server）

### 目标

针对 B 端应用难以分析远程错误，设想构建一个错误收集监控系统，以便错误记录与收集并展示。

### 架构

整个系统包含[前端系统](https://github.com/Zwe1/error-monitor-frontend)，source-map [插件](https://github.com/Zwe1/error-monitor-webpack-plugin) 和 后端系统，及数据库服务。前端收集上报发送错误信息到服务端，服务端处理收集存储错误信息到数据库，并支持前端获取处理后的错误信息，支持 source-map 转换后，在前端进行集中展示。

### 基本功能

- [x] 前端系统报错收集。
- [x] 错误信息数据库存储。
- [x] 提供错误信息获取接口。
- [x] 支持上传 source-map。
- [ ] source-map 解析。
- [ ] 输出解析后错误信息。

## 服务端开发

到了服务端，就离不开通信协议、资源利用率、安全与用户识别、进程间通信，和前端开发在领域方面有很大差别。但在软件设计策略方面却又有很多一致性，如解决业务构建逻辑时常用的设计模式，分层式架构，软件设计的基本原则。  
可以先构建一个适用的架构，再填充业务，针对性的解决平台差异带来的不适和业务逻辑难点。

## 设计准则

1. 不求一次到位，先解决问题，就是好方案。
2. 将设计放置在环境中，预想可能存在的漏洞
3. 不断优化架构，让设计不仅适用当前问题，更能适应未来
4. 好的工程应该有好的工程结构规划，层次分明，职责分离，统一入口。

## 亮点

1. 自动创建空目录

为保持 sourcemap 文件原有的目录结构，实现了自动创建目录的 util, 会根据 sourcemap 的目录层级自动创建相应目录。

## 问题

1. sourcemap

整个项目中的难点与闪光点大概就是错误文件的 source-map 文件解析。  
实际的使用场景中，我们需要考虑以下问题：

```md
- 用户身份

使用这个系统的用户群我们设定在单租户多用户群里。那么租户内的每个用户都会对应一个 souce-map 样本堆。它包含了用户身份信息，版本信息与源码内容。

- source-map 多样

我们限定处理用户使用 webpack 打包并生成 source-map 的场景。但是生成 source-map 的方式多样，可能使用 文件名加 hash 也可以是 文件 id 和 contenthash。这在很大程度上，增加了解析文件的难度。可选的方案如下：

约束生成 manifest 文件，这样会降低我们的解析难度。

在插件初始化时，提供可选模式，根据项目环境选择模式，从而确定解析模式。

- source-map 解析

在 source-map 解析过程中，文件名称，行列号是至关重要的三条信息。  
当错误信息不包含这几项时，我们可以试图在错误栈中获取，错误栈的第一条，我们视作出错位置。其中包含了文件名和行列号，对其进行提取，获取信息后，进行 source-map 转换。

- source-map 缓存

随着项目的体积增大，source-map 文件也会逐渐增大。这带来了文件解析的降速以及存储空间的浪费。可选解决方案如下：

使用多进程进行文件解析，解析完成后结果交由主线程输出，从而提高解析效率。

缓存 source-map 文件，利用 contentHash 来缓存文件。这样一个用户可以仅保留一套 source-map 文件，增量叠加。
```

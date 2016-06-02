title: Metal入门一
date: 2016-6-2 4:20:30
categories:
- Metal
tags:
- Metal入门
- 基础
permalink: The-first-article-Metal
---
　　Metal 框架支持 GPU 加速高、 3D 图像渲染以及数据并行计算工作。Metal 提供了先进合理的 API，它不仅为图形的组织、处理和呈现，也为计算命令以及为这些命令相关的数据和资源的管理，提供了细粒度和底层的控制。Metal 的主要目的是最小化 GPU 工作时 CPU 的消耗。

<!-- more -->
　　若你是游戏开发者，Metal 不是最佳选择。苹果官方的 Scene Kit (3D) 和 Sprite Kit (2D) ， Epic提供的 Unreal Engine 或 Unity，是你更好的选择。使用这些引擎，你无需直接使用 Metal 的 API，就可以从 Metal 中获益。
![框架结构图](/resources/Metal/1.png)

## Metal概要

设计目的：3D图形渲染和并行运算
学习目的：开发自己游戏引擎或高性能并行运算库，了解游戏引擎底层原理。
优点：高性能绘图（减少资源开销）、避免多余的校验和编辑
缺点：目前仅限于搭建A7及更新芯片的iPhone、iPad（A7芯片以上的iOS平台）
特点：metal方法是基于协议设计，面向接口开发

## 注意点

- 暂态的对象（创建和销毁是廉价的，它们的创建方法都返回 autoreleased对象）

> 1.Command Buffers
> 2.Command Encoders

- 非暂态的对象（在性能相关的代码里应该尽量重用它,避免反复创建）

> 1.Command Queues
> 2.Buffers
> 3.Textures
> 4.Sampler States Libraries
> 5.Compute States
> 6.Render Pipeline States
> 7.Depth/Stencil States

## 开发步骤

- 七步设置Metal

> 1.创建MTLDevice
> 2.创建一个CAMetalLayer
> 3.创建一个Vertex Buffer
> 4.创建一个Vertex Shader
> 5.创建一个Fragment Shader
> 6.创建一个Render Pipeline
> 7.创建一个Command Queue

- 五步完成渲染

> 1.创建一个Display link
> 2.创建一个Render Pass Descriptor
> 3.创建一个Command Buffer
> 4.创建一个Render Command Encoder
> 5.提交你Command Buffer的内容

- 编写着色器两个函数
> 1.编写顶点函数（顶点数据中每个顶点被执行一次）
> 2.编写片段函数（每个像素被执行一次）

## 开发实践
　　使用Metal编写一个旋转正方形。本文Demo你可以从Github上获得。

- 七步初始化metal

> ```swift
    // MARK: - 七步初始化Metal
    private func setupMetal() {
        // 1.0 创建一个MTLDevice
        device = MTLCreateSystemDefaultDevice()
        // 1.1 创建一个CAMetalLayer
        metalLayer = CAMetalLayer()
        metalLayer.device = device
        metalLayer.pixelFormat = .BGRA8Unorm
        metalLayer.framebufferOnly = true
        metalLayer.frame = view.layer.frame
        view.layer.addSublayer(metalLayer)
        // 1.2 创建一个Vertex Buffer
        let dataSize = vertexData.count * sizeofValue(vertexData[0])
        vertexBuffer = device.newBufferWithBytes(vertexData, length: dataSize, options: MTLResourceOptions.OptionCPUCacheModeDefault)
        // 旋转缓冲区内容时刻变化，这里通过长度创建
        uniformBuffer = device.newBufferWithLength(sizeof(Uniforms), options: MTLResourceOptions.OptionCPUCacheModeDefault)
        let defaultLibrary = device.newDefaultLibrary()
        // 1.3 创建一个Vertex Shader
        let vertexProgram = defaultLibrary?.newFunctionWithName("basic_vertex")
        // 1.4 创建一个Fragment Shader
        let fragmentProgram = defaultLibrary?.newFunctionWithName("basic_fragment")
        // 1.5 创建一个Render Pipeline
        let pipelineStateDescriptor = MTLRenderPipelineDescriptor()
        pipelineStateDescriptor.vertexFunction = vertexProgram
        pipelineStateDescriptor.fragmentFunction = fragmentProgram
        pipelineStateDescriptor.colorAttachments[0].pixelFormat = .BGRA8Unorm
        pipelineState = try! device.newRenderPipelineStateWithDescriptor(pipelineStateDescriptor)
        // 1.6 创建一个Command Queue
        commandQueue = device.newCommandQueue()
    }
 ```

- 五步完成渲染

>```swift
 // MARK: - 五步完成渲染
    private func displayContent() {
        // 2.0 创建一个Display link
        timer = CADisplayLink(target: self, selector: #selector(ViewController.gameloop))
        timer.addToRunLoop(NSRunLoop.mainRunLoop(), forMode: NSDefaultRunLoopMode)
    }
    func render() {
        let drawable = metalLayer.nextDrawable()
        // 2.1 创建一个Render Pass Descriptor
        let renderPassDescriptor = MTLRenderPassDescriptor()
        renderPassDescriptor.colorAttachments[0].texture = drawable?.texture
        renderPassDescriptor.colorAttachments[0].loadAction = .Clear
        renderPassDescriptor.colorAttachments[0].clearColor = MTLClearColor(red: 0.0, green: 104.0/255.0, blue: 5.0/255.0, alpha: 1.0)
        renderPassDescriptor.colorAttachments[0].storeAction = .Store
        // 2.2 创建一个Command Buffer
        let commandBuffer = commandQueue.commandBuffer()
        // 2.3 创建一个Render Command Encoder
        let renderEncoder = commandBuffer.renderCommandEncoderWithDescriptor(renderPassDescriptor)
        renderEncoder.setRenderPipelineState(pipelineState)
        // 将vertexBuffer添加到第0个缓冲区（等下metal着色器要取出值）
        renderEncoder.setVertexBuffer(vertexBuffer, offset: 0, atIndex: 0)
        // 将uniformBuffer添加到第1个缓冲区（等下metal着色器要取出值）
        renderEncoder.setVertexBuffer(uniformBuffer, offset: 0, atIndex: 1)
        renderEncoder.drawPrimitives(.Triangle, vertexStart: 0, vertexCount: 6)
        renderEncoder.endEncoding()
        // 2.4 提交你Command Buffer的内容
        commandBuffer.presentDrawable(drawable!)
        commandBuffer.commit()
    }
```

- 编写顶点函数及片段函数

>```Swift
typedef struct
{
    float4x4 rotation_matrix;
} Uniforms;
typedef struct
{
    float4 position;
    float4 color;
} VertexIn;
typedef struct {
    float4 position [[position]];
    half4  color;
} VertexOut;
///顶点函数：传入一个vertices顶点数据，&uniforms旋转矩阵的引用，vid顶点索引
vertex VertexOut basic_vertex(device VertexIn *vertices [[buffer(0)]],
                              constant Uniforms &uniforms [[buffer(1)]],
                                 uint vid [[vertex_id]])
{
    VertexOut out;
    out.position = uniforms.rotation_matrix * vertices[vid].position;
    out.color = half4(vertices[vid].color);
    return out;
}
/// 片段函数
fragment half4 basic_fragment(VertexOut in [[stage_in]])
{
    return in.color;
}
```


## 参考资料
[objc中国书籍-Metal](http://objccn.io/issue-18-2/)
[使用Metal打造令人惊叹的游戏效果](http://www.cocoachina.com/ios/20141119/10248.html)
[iOS 8 Metal Swift教程 ：开始学习](http://www.cocoachina.com/swift/20140902/9506.html)
[iOS 8 Metal Swift 教程（二）:3D图形](http://www.cocoachina.com/swift/20141107/10155.html)
[Metal Framework基础使用教程](http://www.cocoachina.com/game/20141013/9890.html)
[Metal基本图像处理实例](http://www.cocoachina.com/ios/20141020/9979.html)

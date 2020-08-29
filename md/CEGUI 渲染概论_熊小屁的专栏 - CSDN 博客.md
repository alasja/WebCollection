\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.csdn.net\](https://blog.csdn.net/kongfuxionghao/article/details/6766786)

1、几个重要的类

Direct3D9Renderer 负责 CEGUI 的 Render 的接口

RenderingSurface 渲染接口类

RenderingWindow 可渲染窗口

RenderTarget 的继承关系相关类。

Direct3D9GeometryBuffer 类

其方法

 void draw() const; // 渲染  
    void setTranslation(const Vector3& t);  
    void setRotation(const Vector3& r);  
    void setPivot(const Vector3& p); // 旋转轴  
    void setClippingRegion(const Rect& region); // 裁剪区域  
    void appendVertex(const Vertex& vertex); // 加入顶点  
    void appendGeometry(const Vertex\* const vbuff, uint vertex\_count);  
 // 加入顶点 BUFF  
    void setActiveTexture(Texture\* texture); // 设置当前互动纹理  
    void reset();  
    Texture\* getActiveTexture() const;  
    uint getVertexCount() const; // 顶点个数  
    uint getBatchCount() const; //BatchInfos 个数，BatchInfos 存储纹理信息  
    void setRenderEffect(RenderEffect\* effect); // 当前使用的 RenderEffect  
    RenderEffect\* getRenderEffect();

由于公司不能上传图片，因此这些需要查阅相关 CEGUI 代码。不过这些类绝对关旭 CEGUI 渲染

2、CGUI Image 渲染流程：  
Draw 过程：  
1、渲染区域与目标区域计算偏移。  
2、图片渲染区域（Rect) 与纹理渲染区域（Rect) 的计算。  
3、 构造该图片的顶点信息，Vectex\[6\] 2 个三角形，结构体包括顶点坐标，顶点颜色，顶点纹理 UV。  
4、 图片相关联 GeometryBuffer 设置当前使用纹理，以及加入刚才构造的顶点信息。

Image Draw 并没有做出渲染，只是一些渲染计算，真正渲染要交给 GeometryBuffer 的 Draw 做出。这里重复其 Draw 过程：  
1、SetScissorRect 设置裁剪区域。  
2、世界坐标变换。  
3、设置纹理混合方式。  
4、RenderEffect 的 PASS 的调用  
5、纹理，已经纹理相关顶点渲染

3、CEGUI 窗口的渲染流程：  
其实就是 System 中的 renderGUI 过程  
1、 beginRendering：  
（1）设置顶点格式。  
（2） 设置 PS 和 VS，这里都是空，他没有 Shader 处理，依然是固定管线。  
（3）SetRenderState，设置 device 状态  
（4）SetSamplerState，设置纹理寻址方式  
（5）SetTextureStageState，设置颜色计算方式。  
（6）SetTextureStageState，Alpha 的计算方式。  
（7）SetSamplerState，纹理 Filter  
（8)  设置颜色混合不取纹理颜色，打开 Alpha 混合，  
（9）设置视图矩阵 VIEW  
2、 获取 ActiveSheet 的 RenderingSurface，并清除图形信息  
3、 获取 Window 的上下文相关信息，是通过 windowRenderer 获取的，其中包括 RenderingSurface, 窗体本身，当前窗体坐标，在 Queue 等级。这里解释下 RenderQueueID 其标记 Queue 的渲染等级。  
4、 清空 clearGeometry。  
5、 渲染其本身，然后渲染其子窗口。  
6、阐述下 Render 的过程：  
（1）清空曾经 Cache 到的 GeometryBuffer, 因为是 ReRender 需要清空上次缓存的 Buffer。  
（2）渲染前前置事件调用。  
（3）渲染窗口本身，最底层其实也是 Image 的 Draw  
（4）调用渲染结束事件。
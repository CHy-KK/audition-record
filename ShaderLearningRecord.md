## Shader Porperties
* 如果要在c#脚本中使用Shader.SetGlobalxxx(id, xxx)，那么就需要在Shader的Properties中删除这个xxx属性。如果直接使用material.setXxx，则不需要删除该属性。
* _Time的xyzw分别表示time/2, time, time * 2, time * 3
* 为什么使用flowmap时需要使用uv-flowdir？
  
  答：因为比如red(1, 0) * 2 - 1后得到flowdir(1, -1)，表示我们想让我们的图像向右下移动（是的flowdir表示我们想让图像移动的方向）。那么如果我们直接用uv+flowdir，则实际根据时间增加，我们会越来越往右下角 采样 ，那么得到的结果实际上图像是往左上移动，和右下是相反的，所以要用uv-flowdir
* flowmap差半周期混合采样无缝连接函数原理图：![image](https://raw.githubusercontent.com/CHy-KK/Images/main/flowblendtheory.png)
  
  f1 f2是两个采样偏移函数，f3是混合两个采样值的lerp值，采样偏移值越大则越要消除该值的影响，所以使用lerp方式让f1和f2在达到最大值时权重都为0，这样就保证两个偏移值的加和一直都是0

* 使用模板测试时，先渲染正常物体，然后渲染模板遮罩mask(写入模板值，"Queue" = "Geometry+1")，最后渲染遮罩后面的物体(关闭写入模板值，"Queue" = "Geometry+2"在mask之后渲染)，与当前屏幕模板值做比较考虑是否丢弃像素。遮罩第一次写入的方式为，Comp设置为always表示用远通过，然后Pass设置为replace表示替换当前模板缓冲区值
  
  shader中模板的数据结构为：
  ```
  Stencil {
    Ref referenceValue        // Ref[_ID]，用于和模板缓冲区的值做比较
    ReadMask readMask
    WriteMask writeMask
    Comp comparisonFunction   // 比较方式
    Pass stencilOperation     // 通过的行为
    Fail stencilOperation     // 模板测试失败时的行为
    ZFail stencilOperation    // 通过模板测试但不通过深度测试时的行为
  }
  ```
* 渲染顺序：先看RenderQueue，越小越先渲染，再根据Geometry Transparent决定顺序，Geometry按深度从前到后渲染，Transparent使用画家算法从后往前渲染
* 渲染透明物体时不把Queue设置为Transparent的都是傻逼
* 为了保证early-z的最高效率工作，必须将物体由摄像机距离从近到远先进行一次排序，再进行渲染，因此使用z-pre-pass方式替代early-z。本质上z-pre-pass是利用两个pass，第一个pass只写入深度不输出颜色，然后就得到了一张最小深度图，在第二个pass中先关闭深度写入，ZTest设置为Equal即可![image](https://raw.githubusercontent.com/CHy-KK/Images/main/zprepass.png)
  
  但是这样会带来一个问题：多pass的shader无法进行动态合批，所以我们选择提前分离的PrePass，将第一个pass分离出来作为一个单独的shader作为render feature[参见雨松MOMO的博客](https://www.xuanyusong.com/archives/4759)，原先的材质只留下第二个shader。但是这个renderfeature怎么写的还不清楚

  要注意，prepass不一定能带来性能优化，甚至有时比正常光栅化性能开销更大。prepass更多的应该是当无法很好地对opaque物体从前往后排序或overdraw问题很严重时，再采取的一种优化，这样带来的性能消耗远小于排序物体所带来的消耗。

* 纹理压缩
  
  * DXT1：针对RGB纹理，把4*4共16个像素使用64位存储，压缩比为16 * 8 * 3 / 64 = 6:1
    
    64位，前32位存储两个该4*4区块的两个极端颜色值，每个颜色值的rgb通道分别使用5，6，5位表示，也就是说g通道精度会更高，然后对这两个极端颜色值进行插值（没有任何其他存储的信息能表示插值的数值如何，所以应该认为是均匀插值，即3/4 1/4），剩下的32位对应16个像素，每个像素两位，一共表示四种选择，对应四种颜色（2极端值2插值）

  * DXT5：针对RGBA纹理。把4*4共16个像素使用128位存储，压缩比为16 * 8 * 4 / 128 = 4:1
    
    126位中的64位同DXT1，剩下64位表示A通道，原理也一样，使用前16位表示两个极端A值，然后插值得到一共8个A值，剩下48位分配给16个像素，每个像素3位，一共表示8种选择，对应8种A值

  综上可以看出DXT格式的压缩其实会存在一定的损失，在一个块中存在超过两个明显的颜色时，会丢弃到只剩两个极端颜色

  * ATI1（BC4）：一个数据块存储单颜色通道，使用DXT5中A通道同样的编码方式，因此压缩比应为16 * 8 / 64 = 2:1（有说法时是4:1，wiki给的解释似乎是输入纹理像素精度为16而非上面计算用的8位）
  * ATI2（BC5）：每个块存两个颜色通道，相当于放了俩ATI1的块，所以压缩比同上；如果将法线存储在xy双通道中时，采用BC5方式压缩，那么由于每个通道都有其单独的索引，所以相比BC1有更高的保真度（那么BC1是啥格式）缺点就是内存要两倍，对应的也要两倍带宽
    
    注：众所周知三位的法线只需要两位就能算出来

  PC平台DX选择DXTC作为标准压缩格式，OpenGL选择使用ETC格式，再移动平台上被广泛应用

  * ETC：压缩比同DXTC  1，同样将4*4像素单元压缩为64位块，将像素单元水平或竖直分成两个区域，两边的第一层共24位，存储两个颜色信息（两个RGB444或者RGB333+RGB555）， 两边的第二层存储共16个2位的像素索引；最后第三层存储2个4位的亮度索引值，用于指向16个内置的亮度索引表，每个亮度表有4个亮度；
  
    ![image](https://raw.githubusercontent.com/CHy-KK/Images/main/ETC1.png)

    也就是先用4位的亮度索引值获得对应的亮度表（下面只列了8个），然后用2位的像素索引值选取对应的亮度补偿值，然后用这个亮度补偿值叠加到像素对应分块的基础颜色值上（左边的加左边，右边的加右边)
    ![image](hhttps://raw.githubusercontent.com/CHy-KK/Images/main/ETCTABLE.png)

  总结，[unity官方推荐纹理压缩格式以及对应平台](https://docs.unity3d.com/Manual/class-TextureImporterOverride.html) 

  ![image](https://raw.githubusercontent.com/CHy-KK/Images/main/TEXSummary.png)

* 关于SRP
  * CBUFFER：如果在frag或vert shader中使用了未被CBUFFER包裹的 且 在别的shader中也存在的同名输入变量，那么这个shader将会SRP batcher not compatible，后续在看SRP时可以探究一下

* 如何使用主纹理取到自定义渐变颜色？可以根据主纹理采样得到的rgb计算出一个值（任意，比如算luminance）再对渐变纹理进行采样


* shader里应尽量避免产生判断，所以一些简单的数值比较等，可以用clip()、step()等函数转换为正负比较进而转换为0、1数值然后相乘

* [shader properties标签](https://docs.unity3d.com/ScriptReference/MaterialPropertyDrawer.html)，比如自定义keyword、enum等

* 采样环形纹理时要把uv坐标转换为极坐标，x作为方向角，y作为距离
  ```
  x0 = 0.5 + cos(360 * x) * y / 2;
  y0 = 0.5 + sin(360 * x) * y / 2;
  ```
  <image src="https://github.com/CHy-KK/Images/blob/main/GroundCrack01_v2.png?raw=true" width="50%"><image src="https://github.com/CHy-KK/Images/blob/main/circlemodel.png?raw=true" width="50%">

* 不能随便clip，否则被丢弃的像素会保留之前的值，导致频幕上出现暂留的脏迹，最好是就直接置0了。同时使用clip做深度判断时会产生谜之漏光，用if就不会……
* 体积光方案：
  * sunshaft（径向模糊）：
    1. 参数设置问题，会产生严重的抖动
    2. urp下主光源方向需要手动设置，不然只有directional Light
    3. 由于全局统一的采样次数和步长，所以体积光线都是统一长度，也就是一些近处遮挡的效果会失效，会看上去有点怪
    4. 如果主光源坐标设置过低可能会让光线末端产生弯曲，我认为可以理解为a束光线采样到了相邻b束光线的像素点导致不该亮的地方亮了，所以可以把主光源坐标设置高一点
    5. 做步进采样的时候记得要用光线方向除以采样次数，这样的参数设置可以减少锯齿
  
  * 某不记得的方法：


  * 特效方案：
    1. 能够实现动态体积光

* 在使用Camera.main.projectionMatrix时，要使用GL.GetGPUProjectionMatrix函数进行一次转换，因为unity内的P矩阵是按照opengl的规则进行计算，但是在某些平台上必须稍微变换以符合原生api要求。https://docs.unity3d.com/cn/2017.1/ScriptReference/GL.GetGPUProjectionMatrix.html

* unity的剪裁空间y坐标是从上到下的，而不是传统的从下开始。所以在手动从统一设备坐标ndc转换到裁剪空间CS时，要将y坐标乘以-1w

* 重建世界坐标的方式有两种：
  * 直接把ndc空间转到CS下再用VP矩阵转到WS。即使用float4(uv * 2 - 1, sampleDepth, 1)直接乘VP逆矩阵，然后用position.xyz/position.w。
  * 先用近平面上的像素坐标和camera位置求得屏幕射线，然后将深度贴图采样深度值转换为线性深度Linear01Depth(rawDepth, _ZBufferParams)，用射线方向*线性深度得到世界坐标。




## HDRP篇
1. System Value语义是DX10之后新增的语义绑定，所有系统值以SV前缀开头，例如SV_Target（这个是一直都有）,SV_POSITION；

    SV_POSITION和POSITION用法无不同，唯一区别在于SV_POSITION一旦作为vertex shader输出语义，那么这个最终的顶点位置就被固定了，直接进入光栅化处理（不允许进入几何和细分shader？），在fragment shader中不允许修改这个值

2. HDRP后处理Render函数传入的参数source和destination，传入shader的texture并不是2D格式，而是2D_X格式，这是因为HDRP支持XR，导致默认传入的source不是单纯的2D，而是2D_X，所以使用之前的Blit函数会出现问题，因为其需要的传入参数是Texture2D，所以需要使用HDUtils.BlitCameraTexture用一个pass将source格式转为正常的Textuer2D格式（其实HDRP官方给出的范例就是），然后再进行多pass的渲染。注意使用HDUtils.BlitCameraTexture要用对应的传入纹理名称_BlitTexture。后续可以直接使用cmd.Blit，记得设置传入纹理参数为_MainTex
   
3. 场景做出更改之后记得重新烘焙所有的refelection probe

4. 好像直接使用input.getkeydown非常非常消耗性能
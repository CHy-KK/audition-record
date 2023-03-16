* __给值为1-100的数据块（任何形式），如何给vector数据最快？__
  
  答：使用begin和end函数来获取数据的头尾

* __vector和list从头部插入100数据谁更快？内存池（？看一下吧）__
  
  答：一般情况下，向 vector 和 list 的头部插入数据会比向尾部插入数据更慢，因为需要将已有的元素向后移动来腾出空间。不过，相对而言，向 vector 插入数据要比向 list 更快。

  这是因为 vector 是使用连续的内存块存储元素的，因此在向其头部插入元素时，需要将现有的元素全部向后移动一位，__但由于内存的连续性，这个操作可以通过简单的内存拷贝完成__。

  相比之下，list 是使用链表存储元素的，因此在向其头部插入元素时，需要先创建新的节点，然后将它插入到链表的头部，这个过程需要分配新的内存空间，并且需要更新指针来维护链表结构。

  综上所述，虽然向 vector 头部插入数据的时间复杂度为 O(n)，但由于内存的连续性，实际上向 vector 头部插入数据的速度仍然比向 list 更快。

  另外，由于一般vector存在自动扩容以便加速后续添加元素的机制，因此vector实际占用空间会比给出的vector长度更大，因此通常来说相同长度的vector和list，vector占用空间会更大。长度越大差距越明显。list相对来说一个节点需要多存储两个指针，但是如果data本身非常大（一个巨大的class对象），那么相对多分配的空间来说，指针大小可以忽略不计，毕竟就是两个地址的大小（32bit-4字节，64bit-8字节）

* __由vector聊到如何移动vector的数据？__
* __由list聊到new一个指针的底层实现，如何分配这个空间？考虑到陷入内核态和vector的时间效率比较呢？__
* __map和unordered_map的底层实现？如果有40w数据，不考虑时间效率谁使用空间效率高？__
  
  答：map使用红黑树，每个节点需要存指针，unordered_map使用哈希表，每个数据只有数值本身，所以unordered_map空间复杂度更低

* __静态全局变量的初始化，静态初始化和动态初始化的区别？__
* __memcpy和memmove的区别？memcpy的底层实现？__
* __工厂模式的意义？单例模式的生命周期？__
	1. 工厂模式的意义：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
	2. 优点：
		1. 一个调用者想创建一个对象，只要知道其名称就可以了
		2. 扩展性高，如果想增加一个产品，只要扩展一个工厂类就行
		3. 屏蔽产品的具体实现，调用者只关心产品的接口

* __opengl的上下文与状态机__
* __opengl的设置参数哪些对效率影响最大？drawcall的性能开销主要在哪一方面？__
  
  ⭐⭐⭐⭐⭐⭐Unity中DrawCall和openGL、光栅化等有何内在联系，为什么说DC降低有助于渲染性能优化？ - 文礼的回答 - 知乎  https://www.zhihu.com/question/36357893/answer/569416982 

  https://www.zhihu.com/question/27933010/answer/517754841 为什么应该尽量减少draw call？

  https://zhuanlan.zhihu.com/p/76562300 什么是drawcall setpasscall？
  
  答：drawcall是CPU调用图形编程接口，来命令GPU进行渲染的操作。为了CPU和GPU能并行工作，就需要一个命令缓冲区。命令缓冲区包括一个命令队列，由CPU向其中添加命令，而GPU从中读取命令。这两个过程是独立的，因此CPU和GPU可以互相独立工作。当CPU要渲染一个对象时，可以向命令缓冲区添加命令，而GPU完成了上一次的渲染任务后，就可以从命令队列里取出一个命令执行。
  
  这个命令队列开头一般是一系列设定GPU内部状态寄存器的指令，用来测试诸如是否需要进行ztest，是否需要模板测试，使用哪些buffer作为输入，使用哪些render target作为输出等。在靠近最后的地方，会有一条draw命令。最后可能会有一些与CPU同步的命令，比如更新内存上的某个标志位。
  
  每次调用drawcall之前，CPU要向GPU发送很多内容，包括数据、状态、命令等。CPU需要完成例如检查渲染状态等许多许多工作，而一旦CPU完成了这些工作，GPU就可以开始本次的渲染。GPU的渲染能力很强，因此渲染速度往往快于CPU提交命令的速度。如果drawcall数量太多，CPU就会花费大量时间在提交drawcall命令上，造成CPU的过载。
  
  那么如何减少drawcall？提交大量很小的drawcall会造成CPU的性能瓶颈，即CPU把时间都花在准备drawcall的工作上了。那么很显然，我们可以通过将许多小的drawcall合并为一个大的drawcall，这就是批处理的思路
  
  需要注意的是，由于我们需要在CPU的内存中合并网络，而合并的过程是需要消耗时间的，因此，批处理更适合静态的物体，例如大地、石头等。对于这些静态物体我们只需要合并一次即可，当然，我们也可以对动态物体进行批处理，但是由于这些物体是不断运动的，因此每一帧都需要重新进行合并然后再发送给GPU，这样会对空间和时间造成一定影响。
  
  <font color="#dd0000">但是实际上并不是降低drawcall就可以了！</font>
  
  CPU向GPU提交指令的时候，可以一次提交一个drawcall，也可以一次提交多个drawcall，取决于CPU端的提交策略以及GPU端命令队列缓冲区的长度。一般来说每个drawcall当中会包含一些改变GPU状态寄存器的指令，其中一些状态寄存器的改变可能会改变GPU的工作方式，比如打开or关闭ztest，就好比要重新设定生产流水线上的机器参数一样，这种被称为GPU的contextroll，是GPU管线生产力降低的一个原因
  
  因此，一般有降低drawcall来优化的说法，在比较笼统的层面上，这种说法没有什么太大问题。但如果引擎控制的好，drawcall的数量并不等同于contextrool的数量，比如用100个drawcall绘制100个同样材质的物体，那么就如同工厂流水线上要生产100辆同样的车一样，我们是不必在每个drawcall中都重新设置一遍GPU状态寄存器的。对于这种情况，从第二个drawcall开始GPU端的消耗会很小。相反，如果CPU端将这100个drawcall进行合并，那么可能要将这100个物体的顶点数据进行合并，反而可能非常影响性能。比如合并之后剪裁的粒度变粗了，因为可能一些包围盒在视锥外的物体，在合并后的物体有一部分被包括了，所以连带原来需要被剔除的顶点数据一起被传给了GPU，就要渲染更多顶点了，反而增大了GPU的负担（虽然GPU比CPU快很多就是了）。
  
  但是引擎如果要避免不必要的contextroll，引擎就要对GPU的状态进行跟踪管理，并且清楚地知道哪些命令会发生context roll，由于不同GPU在这方面如别很大，这里需要妥协，从而对于通用引擎（怪不得我的1050ti降低drawcall效果显著）来说，降低drawcall往往是有效果的。
  
  <font color="#dd0000">那么如何减少context roll的发生呢？</font>
  
  比如，我们可以根据材质进行排序。将相同的材质的物体放到一起绘制，那么我们就可以大大减少更换传递参数的次数。或者说，直接对于相同的物体使用实例化的方式批处理绘制（见下面）
  
  __从另一个角度看这个问题__
  
  GPU内部也有cache，也有命中和未命中从显存中取数据导致的效率问题。因此，比如我们要换绑一个texture，那么很可能cache miss，就需要重新从显存中读取texture，甚至可能需要从CPU发送一个新的texture数据，这样消耗的资源是很大的。GAMES104提到的gpu性能限制
  
  ![image](https://github.com/CHy-KK/Images/blob/main/Imagedasdasdas.png?raw=true)
  
  附：25个导致drawcall增加的情况（没太看懂）https://answer.uwa4d.com/question/5ae1eef932493548fbd32ba1另外，由于UI系统包含大量物体，因此会对drawcall调用产生很大影响，如何优化也是提升性能的关键https://www.bookstack.cn/read/fairygui/17aab488526a353b.md

  __从纹理角度来看减少drawcall__

  由于每次渲染状态或参数比如纹理设置的改变CPU都需要重新提交drawcall，导致浪费大量时间，所以有很多纹理设置可以减少drawcall。比如最常用的纹理图集，也就是每次把一组纹理放到一张纹理内，然后要使用的时候就按照uv偏移来找要其中哪一张。这样就可以降低纹理频繁切换带来的消耗。

  __静态批处理 vs SRP Batch vs GPU Instancing vs 动态批处理__

  简单来说，我们主要要减少的是setPassCall，因为实际上消耗资源的是切换GPU的shader参数，也就是setpasscall的内容。即使我们什么都不做，GPU在渲染连续两个连续的完全相同的材质时也不会调用setpasscall，但是会调用drawcall，而单纯的调用drawcall是不太消耗资源的。可以根据下图右边的材质（同颜色表示相同材质）渲染顺序来计算sepasscall的数量，注意有两个必存在的setPassCall——DrawGL和绘制天空盒。
  ![image](https://github.com/CHy-KK/Images/blob/main/setPassCall.png?raw=true)
  
  * SRP Btach能合并使用同一shader的不同mat，不同mesh的物体，但没有减少drawcall，只是减少了切换mat导致的setPassCall。其中SRP batch在GPU中保留Cbuffer中的properties以及不同物体的空间转换矩阵（M），从而避免每个drawcall都要将shader信息从CPU发送到GPU；每个物体对于该保存地址对应有一个offset，渲染该物体的drawcall只需要传送这个offset查询对应数据即可。因此不会减少drawcall数量，只会减少setPassCall的数量，而原来是每个材质（注意不是每个物体）一次setPassCall
  * GPU Instancing能够使用一个draw call来绘制所有使用相同mesh和相同material的物体。CPU将所有使用同一mesh物体的空间变化（objectToWorld）和材质信息放入一个发送到GPU的数组中，GPU会遍历数组绘制材质。从而避免了由切换渲染物体导致的drawcall。GPU instancing根据平台不同有数量限制，如果超出最大值会分成两个drawcall
  * Dynamic Batch能将不同mesh但使用同一材质的物体，组装成一个更大的物体提交GPU，一个批次的动态物体顶点数是有限制的，以下是开启和不开启的对比（sphere由于网格过多所以无法使用dynamicbatch）。对比两张图我们可以清晰的看到，开启DB甚至比不开还慢，且同样都是33个setPassCall。这是因为，首先不开启DB时，物体的渲染顺序也会被调整为优先渲染同一材质，也就是保证了最少的材质切换，另一方面DB也是将同一物体组装成同一
  ![image](https://github.com/CHy-KK/Images/blob/main/dynamicBatch_sphere.png?raw=true)
  ![image](https://github.com/CHy-KK/Images/blob/main/dynamicBatch_cube.png?raw=true)

* CPU与GPU的单向数据传递GPU计算速度的确比CPU快很多，所以可能有的想法就是先让CPU将数据传到GPU中进行计算，等GPU计算完后CPU再将结果读取回来，再基于这个结果进行判断，之后再告诉CPU如何绘制，这个过程叫back-force。由于现代引擎中render和logic是不同步的，但是如果有哪个render步骤需要等back-force的话，这样做会导致有半帧或者一帧的画面和逻辑不同步的现象出现。因此在设计代码时，尽量保证数据的单向传输（CPU -> GPU）,避免计算同步问题，且不要从显卡中读取数据。

* __GPU driven？__
* __⭐⭐⭐⭐⭐GPU底层硬件__
  
  https://www.bilibili.com/video/BV1aM4y1g75f?spm_id_from=333.999.0.0&vd_source=a496344996aebd6de06a773bff299dc1

* __OpenGL的批处理实例化：https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/10%20Instancing/__
  
  简单概括来说，比如我们要渲染一堆草的模型，我们肯定不能一次传入一根草的模型，而是一次性向GPU传一批草。可以使用glDrawArrayInstanced或者glDrawElementInstanced，最后一个参数给定一组顶点数据需要渲染的次数amount。然后在vertexshader中我们需要对传入的顶点数据进行位移，否则即使渲染再多次他们也是叠在一起的，看不出来。我们可以使用gl_InstanceID这个自有变量，这个变量会告诉我们当前这个顶点属于第几个要渲染的实例，然后使用uniform传入一个amount大小的位移矩阵数组。我们也可以传入的一个amount大小的位移矩阵（Model矩阵）数组，也就是使用顶点属性来传入一个mat4矩阵。这里由于顶点属性最大允许的树大小等于一个vec4，因此我们需要绑定四个顶点属性，然后
  
* __是否每一次drawcall都要重新传一次顶点数据？（VAO和VBO到底是如何将数据传给gpu的？）__
  
  learnopengl;
  
  https://blog.csdn.net/u012169524/article/details/113383015
  
  答：不是！首先我们在渲染循环外面，就会把外存上的数据绑定到VBO中，VBO会将绑定的顶点数据发送到显卡上。我们使用glBufferData函数绑定外存的数据，该函数的最后一个参数：
	* GL_STATIC_DRAW ：数据不会或几乎不会改变。
	* GL_DYNAMIC_DRAW：数据会被改变很多。
	* GL_STREAM_DRAW ：数据每次绘制时都会改变。
  
  这个参数会指定我们希望显卡如何管理给定的数据。如果顶点的位置数据一直不变就用staticdraw，如果经常变化就用后两个，以确保显卡把数据放在能够高速写入的显存部分。换句话说，VBO就是在显存中开辟出的一块内存缓冲区，用于储存顶点的各类属性信息，而每个VBO的id都能在Opengl中映射得到这个VBO在显存中的地址，通过这个id可以对特定VBO内的数据进行存取操作。网上很多说VAO是解释VBO的，但我认为这个说法并不对。解释VBO数据的是glVertexAttribPointer，VAO只是将VBO与顶点属性指针的配置保存起来，在每次绘制物体之前绑定VAO，就可以告诉GPU要用哪一块VBO数据（此时已经存在于显存中），以及这一块VBO的顶点属性配置。也就是说，VAO记录了glVertexAttribPointer()函数的结果

* __帧缓冲与渲染缓冲直接渲染都是在默认帧缓冲上的挂载的渲染缓冲进行的。如果要使用我们自己的帧缓冲，一个完整的帧缓冲有以下四个条件__
	* 附加至少一个缓冲（颜色、深度或模板缓冲）。
	* 至少有一个颜色附件(Attachment)。
	* 所有的附件都必须是完整的（保留了内存）。
	* 每个缓冲都应该有相同的样本数。
  
  也就是说，使用我们自己的帧缓冲保存渲染数据，需要挂载一个纹理来存储得到的数据。一般来说，渲染缓冲区和纹理不能同时挂载在同一个帧缓冲上（使用挂载纹理的帧缓冲提前渲染好，然后在默认帧缓冲的渲染缓冲上写入渲染好的数据）

* __UBO&SSBO?__
* __opengl内存泄露如何查问题__
* __opengl的win32主消息循环处理__
* __pbr为什么要把diffuse和specular分开？意义是什么__
  
  cooktorrence的brdf分为两个部分，漫反射部分brdf为左边一项表示光子进入平面被折射的部分，高光反射表示光子直接被反射的部分。其中绝缘体表面粒子不会捕获光子，因此光子进入绝缘体表面在碰到其他粒子多次反射后会向各个方向反弹出表面，也就是形成漫反射。金属表面电子会捕获光子，所以进入表面的光子就出不来了，也就是理论上金属没有漫反射。kd表示进入平面的光子比例（光线被表面折射的部分），flambert是color/π，color即表面颜色。因此，漫反射的brdf本质上就是常数。而高光反射的brdf的fcool-torrence就是DFG项。分母部分是标准化因子。

* __IBL的PBR__
  
  IBL其实就是基础PBR的环境光照部分。IBL给出了入射光的计算方式，漫反射部分就是模糊环境贴图，高光反射部分需要用splitsum分解积分公式，然后预计算出LUT。
  
* __什么时候pbr使用次表面散射，什么时候单纯diffuse__
  
  答：一般的diffuse材质是基于物理的渲染做了一个简化的假设：平面上每一点的折射光都会被完全吸收而不会散开。而次表面散射材质认为光线并非会被完全吸收，而是可能会与表面的其他粒子碰撞数次后最终离开表面。所以次表面散射一般用于能有一点透光效果的材质，例如玉石、人的皮肤、蜡等

* __pbr的导体和绝缘体有什么区别？__
  
  答：绝缘体的R0（基础反射率，或者说F0）比较低，金属的R0一般高于0.5。由于绝缘体一般不反射本身颜色，所以R0都是一个值（指RGB三通道都一样）。因此，绝缘体的高光反射不会反射出颜色，而金属高光反射会反射自身颜色。同时，金属表面几乎不存在漫反射。不同频率光谱的R0值不一样，所以金属的反射率会用RGB三通道表示。金属材质不会发生次表面反射，只用反射颜色（即反射出的金属光泽）。最后，半导体一般不会出现在渲染场景中，不考虑其反射率

* __PBR在工业界的具体实现方法？https://zhuanlan.zhihu.com/p/420379757__
  
  答：一般分为两种工作流，分别是metallic-roughness和specular-glossiness。首先我们要知道，计算给定入射和观察方向的BRDF其实我们只需要知道几个量，分别是基础反射率R0或者说F0（用于描述菲涅尔项）、roughness（用于描述表面法线分布），以及表面的法线N（不是微表面法线）、入射光方向L、观察方向V。然后三个方向都是给定的，我们输入的贴图就是为了得到roughness和R0。
  * SG：首先diffuse就是直接计算漫反射项（漫反射下的BRDF视为一个常数），这部分不参与微表面BRDF的计算，然后后面的specular其实就是R0，Glossniess是1-roughnessgames104中提到的SG方式的具体代码
  * MR：MR其实就是在SG上面再套了一层壳，来让基础反射率R0不会因为一些不合理的设置导致出离谱的结果。MR首先给出一个basecolor，然后通过metalic去决定这个basecolor如何使用。下面给出的是M-R转S-G参数的代码。我们可以发现在计算高光分量的时候用了一个线性插值：也就是说，metalic即金属性，会决定有多少表面的基础颜色会被作为高光反射，如果是非金属绝缘体，那么高光反射的量其实是被锁死在一个非常小的值，也就是图中的dielectriSpecularColor。同时diffuse项则是金属性越低则越反射本身的颜色，金属本身其实不具有漫反射颜色。最后roughness就是roughness（吐槽一句这个convert函数里做了g=1-r，然后上面SG里面又把r=1-g，原地tp了属于是）
  * 
* __一个gameobject必备的组件有哪些__
  
  答：transform
* __transform除了空间属性以外还有什么意义__
  
  答：transform本质是一个基本类，内容包括一系列属性以及实例方法。内部还包括他的parent属性，其空间属性都是基于父物体的空间属性的。

* __纹理填充率和像素填充率https://blog.csdn.net/OnafioO/article/details/49151109__
  
  答：像素填充率是指图形处理单元在每个单位时间内（即一秒内）所渲染的像素数量，单位是MPixels/S。像素填充率＝显卡的显示核心频率乘以像素渲染管线数量。纹理填充率就是指GPU在单位时间内所能处理的纹素的数量，单位是MTexels/S。纹理填充率＝核心频率乘以像素渲染管线数量再乘以纹理贴图单元数量=像素填充率*纹理贴图单元数量
  
* __prefab和单纯的gameobject有什么区别__
  
  答：prefab是一个游戏对象及其组件的集合。1、频繁创建物体时，使用prefab可以节省内存2、使用prefab可以动态的加载已经设置好的物体3、所有的prefabs实例都是prefab的克隆，只要原型prefab发生改变，则所有的实例都会发生变化4、gameobject本质是一个抽象的类，prefab是类似配置文件的性质，保存了实例化gameobject用的参数的配置文件5、实例化本质是使用prefab中保存的数据构造了一个gameobject对象？

* __骨骼动画蒙皮组件了解吗__
  
* __UGUI晚点看一下延迟渲染、lumenGAMES202一些知识点小总结__
  
  延迟渲染问题在于不支持MSAA等屏幕空间抗锯齿，透明物体无法渲染（需要一个另外的前向渲染pass），太占显存带宽，屏幕上所有物体只有一个光照模型
  
* __动态物体使用什么进行空间划分空间划分？__
  
  答：BVH更好用。本质上是一个树的节点的增删合并的过程，BVH是按照对象进行划分，允许空间重叠，本质上是对每个物体的包围盒进行合并，这样对于运动的物体更加方便。

* __游戏引擎使用组件模式的缺点__
  
  答：效率上不如直接写class，因为每次运行需要找到每个组件的接口去访问。 详见ESC模式，把同样的组件、数据都放到一起，以避免切换组件浪费的效率。同时还要考虑组件之间进行通信造成的一些浪费，需要去调用专门的接口获取数据等等。

* __纹理压缩__
  
  实际上如果每次向GPU传输纹理，如果传递整张纹理数据那么数据量是非常大的。 所以我们可以应用一些硬件编码纹理压缩技术来减少纹理传递的数据量。这也是提升GPU速度的有效手段

  纹理压缩格式具体请见ShaderLeraningRecord笔记

  unity中纹理压缩格式可在texture的inspector窗口中可以调整，default下基本都使用不压缩方式，PC和Android端下有可选的覆盖选项，安卓端的可选压缩方式非常多，为了节约带宽；注意，如果要使用纹理压缩最好关闭mipmap，否则mipmap会出现问题
  ![image](https://github.com/CHy-KK/Images/blob/main/textureformat.png?raw=true)
  
* __在游戏引擎中通常采用block思想：__
  
  将纹理划分为多个小块（block），然后进行压缩。
  
  以DXTC格式举例，对于每个划分的小块，取得其中最亮和最暗的像素点，那么我们就可以通过插值处理从而求得二者中间一系列的颜色


* __Shader管理？__

* __LOD？__
  
  https://zhuanlan.zhihu.com/p/32700416
  mipmap中的lod：在纹理精度大于屏幕像素精度时，采用四个邻近像素投影到纹理上得到的不规则四边形的最长边包含的纹素数量，来决定使用哪一级的mipmap

* __为什么unity/ue里开启各向异性过滤的纹理内存不是3倍而是仅仅1/3呢？__
  
  答：重用mipmap，也就是本质还是在mipmap上采样而非使用ripmap（横竖做压缩的mipmap消耗是三倍），将四个像素点投影到纹理空间得到不规则四边形，取最短边作为level值得到对应level的mipmap；然后较长边会创建一条各向异性的线穿过这个四边形方块中心，按照过滤等级的高低在这条线上进行多次采样并合成，得到最终采样的结果。采样次数根据各向异性过滤的值，长:短边的值，向上取整得到采样次数。比如长是短边的2.5倍，那么采样3次。

  但是这种方法计算消耗会更大，因为采样次数增加了。目前最大是16x（采128次），当然，只有最大角度也就是45度时才需要采最大次数

* __为什么mipmap可以节省带宽？__
  
  首先我们要知道，GPU会优先将读取纹理的指令排列在一起，也就是可能这条指令是读(0.1, 0.2)处纹素，第一条指令是读(0.9, 0.7)处纹素。而GPU在读取纹理时是将读取uv处周围一片纹理值全部读入L1 L2缓存(cache)，那么如果下一个读取指令的读取位置在上一条的附近，当然就可以命中cache数据。但是显然当纹理过大时比如level=0的mipmap，像上面说的两个点几乎是不可能命中cache的，也就需要重新加载一片纹素；那么当level=5或者更高(1/32倍了已经)，纹理很小的情况下，上面两个点的纹理值很可能就一次都load进L1L2缓存了，大大提高了cache命中率，减少了GPU加载纹理的次数，节省了带宽。



* __当物体坐标离世界原点太大导致的浮点精度过低怎么办？__
  
  答：将原点改换成为摄像机，即将物体转换到摄像机空间坐标。浮点精度损失：浮点数之所以叫浮点，就是因为小数点前后位数是动态的，如果整数部分太大就会压缩浮点部分的位数，浮点部分位数太长整数部分就不能太大。float是32位，其中一位符号位，double64位。

* __纹理压缩__
  
  为了更好的支持图像文件的随机读取，且降低了内存需求（去掉了GPU不需要的格式信息，简化为只有RGB的数据），使GPU能够快速寻址并采样。

* __bump mapping & displacement mapping__
  
  bump mapping即法线贴图（是虚假的视觉效果）
  
  displacement mapping是通过改变顶点位置来实现（是真实的），但是对建模精度有更高要求。如果模型精细度达不到，可以通过曲面细分shader之后再应用。
  
  这里要注意曲面细分在不对顶点位置改变时跟不细分是完全一样的，只不过把大三角形变成了很多没有凹凸起伏的小三角形dsaddasdaf

* __texnD(s,t,ddx,ddy)__
  
  使用微分查询二维纹理。每个相邻纹素的颜色值之间都有一个差值，通过给定一个微分值来控制相邻纹素值大于该微分值时才取出该纹素，否则不取出该纹素

  ddx(value)/ddy(value):取该像素与邻接像素在value值上的差值，可以用来做边界检测。以下是shader代码结果
  
  ```
  color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, fin.uv);
  color = ddx(color) + ddy(color);
  ```
  ![image](https://github.com/CHy-KK/Images/blob/main/dasdadsa2.png?raw=true)
* __Light Probe__
  
  在空间中放很多个采样点（探针）对采样点处的光照进行采样，然后要计算空间中任何一点的光照时可以通过采样的方式进行。另外还有reflection probe，采样密度会更低，但是精度会更高，因为反射一般对高频信息非常敏感。

* __切线空间的意义是什么？__
  
  法线存储在切线空间最大的意义在于只有在切线空间，才可以应对各种形状的物体，如果是固定方向的世界空间法线，以正方体六边形为例，那么将只有一个面的法线是正确的，其他面的法线都错误。

  另外，unity在非移动平台上会将法线贴图转换为DXRT5nm格式，这个格式只有两个有效通道，因此节约了空间。（移动平台还是rgb三通道）

  注意，切线空间中的up方向是z轴而不是y轴。
  
  切线空间的具体应用请参照geoshader艹渲染一节

* __视差贴图__
  
  相对法线贴图增加了模型各个顶点的高度信息(__记录的是实际高度距离建模平面的值，实际高度=1-采样值，也就是不可能在最初命中点的后面会有比该点更高的 且  能挡住该点的点，所以每次采样只需要向前查找，不会需要向后查找__），看起来会有遮挡关系。实现方式是通过根据A顶点处其高度值以及viewDir计算出一个偏移量，取偏移处顶点b的uv作为纹理采样的uv，而实际应该取的是B处的uv，可以看出来偏差其实还是挺大的，所以偏移法精度比较低。

  ![image](https://github.com/CHy-KK/Images/blob/main/parallaxmapping.png?raw=true)

  还有另一种steep parallax mapping，计算偏移的算法不同，采用步进（ray march）的思想，从A点每次往前走一点，直到当前高度值高于步进位置的高度值D点。

  ![image](https://github.com/CHy-KK/Images/blob/main/steepparallaxmapping.png?raw=true)

  浮雕贴图使用更复杂的算法步进+二分查找

* __为什么需要伽马校正？__
  
  人眼对于暗部的变化更加敏感，越亮对变化越不敏感——韦伯定律。因此对于物理均匀增加的亮度，人眼会觉得并不均匀（会显得暗部远小于亮部）。因为人眼不易识别亮部的颜色差异，所以在早期的电子图象存储技术中，人们降低亮部的采样数来节约磁盘空间——或者说，在不增加数据量的前提下，分配更多的采样数给暗部以提高可辨识精度。具体的做法是：相机捕获到现实世界真实的光的信息，然后对这些数据开n次方根（n即gamma值，一般取1.8~2.2），然后通过采样存储图像。这里的开2.2次方根，因为rgb值是小数所以本质是增大，相当于是原来比较暗比较小的值，现在用更大的值去表示这个较暗的值，也就是分配了更多值给暗部，压缩了亮部的存储空间。最后在显示图片时需要经过一次逆gamma来显示正确的值。
  
  同时因为处理过所以经过gamma校正的值无法直接使用该值进行光照计算，要先解码（逆gamma）之后才能进行颜色相加等操作。所以一定要非常注意导入的texture到底时使用什么空间（线性 or gamma），否则会出现计算出错。同时由于gamma本质是把数值增大了，所以如果本来是在gamma空间下的值但导入后以为他是线性空间下所以没有做逆gamma的话，多次颜色相加之后会出现过曝的问题（颜色值大于1）

* __GPU架构__(GPU架构及运行机制学习笔记 - 最短鹿的文章 - 知乎
https://zhuanlan.zhihu.com/p/393485253)

  层级：SPA(Streaming Processor Array) - TPC/GPC(Texture/Graphics Processor Cluster) - SM(Streaming Multiprocessor) - SP(Streaming Processor/CUDA core)

  * 以下是一个SM的结构

      ![image](https://github.com/CHy-KK/Images/blob/main/SM.png?raw=true)

    * PolyMorph Engine：多边形引擎负责属性装配（attribute Setup）、顶点拉取(VertexFetch)、曲面细分、栅格化（这个模块可以理解专门处理顶点相关的东西）。
    * 指令缓存（Instruction Cache）
    * 2个Warp Schedulers：这个模块负责warp调度，一个warp由32个线程组成，warp调度器的指令通过Dispatch Units送到Core执行。warp是最小的线程调度单位，一个warp下的所有线程执行相同的指令。
    * 指令调度单元(Dispatch Units) 负责将Warp Schedulers的指令送往Core执行
    * 128KB Register File（寄存器）
    * 16个LD/ST（load/store）用来加载和存储数据
    * Core （Core，也叫流处理器Stream Processor）
    * 4个SFU（Special function units 特殊运算单元）执行特殊数学运算（sin、cos、log等）
    * 内部链接网络（Interconnect Network）
    * 64KB共享缓存
    * 全局内存缓存（Uniform Cache）
    * 纹理读取单元(Tex)
    * 纹理缓存（Texture Cache）

  * GPU程序上会把所有线程按照网格（grid）划分，把网格划分为不同的block，每个block可能包含128-512个线程（不能太少也不能太多），GPU会将block分配到SM上，block下的线程会以warp为单位进行划分然后执行。

  * 尽量少使用寄存器
  * 每个线程有自己的本地内存，每个block有共享内存空间（访问速度接近L1 cache），每个grid会访问全局内存。每个SM有64K共享内存，由16个4字节大小的bank组成。一般来说现在的GPU是半个warp去访问这16个bank。如果不同线程访问的bank地址一致就会产生冲突，冲突时需要冲突的线程轮流访问该bank，浪费时间。（或者使用访问一次后广播该bank数据的方式降低消耗），还是要尽量避免产生冲突


* __几何着色器与曲面细分着色器__(https://roystan.net/articles/grass-shader/)

  几何着色器在曲面细分着色器(tessellation shader,包含两个可编程的着色器：hull shader和domain shader)之后（如果没有做曲面细分则是在vertex shader之后），在fragment shader之前（顺便光栅化在fragmen shader前一步执行）。
  
  几何着色器以一个图元为输入，输出一个或多个图元。要注意的是，由于一般vertex最后输出的是clipsapce下的坐标，然后fragment接受，所以理所当然的，geoshader输出的也应该是clipspace下坐标，记得做转换，同时将vertex的输出改为不做转换的坐标，其实就是将toclipspace延迟到了geoshader中做。

  几何着色器如何知道TriangleStream里的vertex是如何组成三角形的？答案是每个新的vertex将和前两个vertex组成三角形。如果需要更多三角形流（triangle strip），可以在TriangleStream中使用RestartStrip函数
  
  ![image](https://github.com/CHy-KK/Images/blob/main/grass-construction.gif?raw=true)
  
* __SSAO__
  
  先对屏幕空间下进行世界坐标重建，然后对于每个像素使用法线贴图取法线信息（需要deferred pass），然后使用法线建立切线坐标系，按照给定的采样半径在半球上采样，对于采样点计算剪裁空间坐标，w值为深度值，然后除以w在规范到01空间得到ndc坐标，使用xy取得对应位置深度图上深度，与采样点深度比较，若小于说明遮挡住，ao+=1。

  优化：length(采样向量)得到weight值，即采样点距离越大ao越没用；同时要忽略掉深度差距过大的情况，这种情况下不会对ao做出贡献；双边滤波模糊

  引擎中的AO：烘焙，优势在于不受物体本身的UV影响，操作简单，缺点在于无法烘焙静态物体
  
  SSAO：灵活，实时，但效果不佳且消耗大(烘焙完用贴图就行)

* __移动端TB(D)R架构tile based （defered） rendering__
  
  * Soc（system on chip），将gpu、cpu、内存等其他手机硬件模块组合在一起的芯片，其中gpu和cpu共享一片内存地址，但都有各自的SRAM的cache缓存，也叫on chip memory，一般几百k~几M大，读取速度比内存读取速度快几十到百倍。在TBDR（延迟TBR）下，on-chip memory会存储Tile的颜色、深度和模板缓冲，读写修改速度都很快。
  * 像素填充率 = ROC运行时钟频率 * ROP个数 * 每个时钟ROP能处理像素的数量（rop即光栅化处理单元，就是光栅化元件）
  * Stall：当一个GPU两次运算结果之间有依赖关系而必须串行时的等待过程。

  TB(D)R简单来说就是屏幕被分为数个16 * 16或32 * 32的像素块（tile-瓦片）来渲染。这里的defer不是传统意义上的延迟管线，而是指 阻塞+批处理 GPU处理 一帧 的多个数据，然后一起处理。目前市面上基本上所有手机都是TBDR。与之相对的PC端架构为IMR（immediate mode rendering立即渲染架构），可以理解为我们一般认为的渲染管线。使用TBDR的目的当然是因为移动端gpu算力较差无法同时对一帧进行处理。
  
  TBDR渲染流程（宏观）：
  
  1. 执行所有与几何相关的处理得到所有的primitive list（图元列表，一般来说就是三角形列表），并确定每个tile上有哪些图元。
  2. 逐tile执行光栅化及其后续处理并写入tile buffer，在完成所有tile后从tile buffer写回到system memory也就是frame buffer中。

  IMR图示

  ![image](https://github.com/CHy-KK/Images/blob/main/IMR.png?raw=true)

  TBDR图示

  ![image](https://github.com/CHy-KK/Images/blob/main/TBDR.png?raw=true)
  总结来说，IMR少了tile环节，光栅化后渲染shading数据直接写入framebuffer。

  * TBDR优势：
  
    * 给消除overdraw提供了便利（overdraw即同一个像素多次着色，给gpu很大的压力，减少被遮挡像素的texturing和shading可以解决overdraw）
    * cache friendly，由于tile buffer在cache中读写很快，以降低render rate为代价，降低带宽，省电。也就是慢一点，但是功耗小一点。

  * 缺点：
    
    * binning是在vertex之后，要将几何数据写入内存中，然后才被fragment shader读取，其中需要很多几何数据的管线， 容易在此处有性能瓶颈。
    * 一个图元可能在几个tile交界处，那么每个tile都要绘制一次该图元，也就是总的渲染次数会多于IMR。

  TBDR的两个defer过程（或批处理过程）
  * binning（类似四叉树）：确定一个图元由几个tile渲染（覆盖了几个tile）
  * early-depth-test（early-z?）

* __⭐⭐⭐⭐⭐移动端优化方案__：https://www.bilibili.com/video/BV1Bb4y167zU/?p=2&spm_id_from=pageDriver&vd_source=a496344996aebd6de06a773bff299dc1 最后十分钟

* __为什么后处理把图像渲染在一个覆盖全屏幕的三角形上比用两个三角形更好？__

  因为这样可以有效节省两个三角形边界像素的overdraw，我们需要知道GPU处理像素时并非是一个像素一个像素处理的，而是以2 * 2或者8 * 8为一组像素并行渲染的，所以会被overdraw的并非只有边界上的一个像素，而是靠近边界上的一组像素都会被绘制两次（光栅化和fragment部分应该都会？）。
  
  另外需要注意，这个全屏幕三角形无论多大都无所谓，因为只要他只覆盖了屏幕上的像素，那么需要处理的就只有屏幕上的像素数量，哪怕是屏幕的1k倍也无所谓。

* __SRP__
  
  不知道从哪讲起好，以后可能再补充吧
  
  1. 首先，我们需要创建一个SRP asset，继承RenderPipelineAsset类，给他挂个CreateAssetMenu的tag我们就可以在菜单栏里创建这个asset文件。该asset类中的CreatePipeline函数我们需要重写，直接返回一个我们创建的SRP实例即可。我们可以在这个asset类中自定义属性传给这个SRP实例。

      ``` c#
      [CreateAssetMenu(menuName = "Rendering/Custom Render Pipeline")]
      public class CustomRenderPipelineAssets: RenderPipelineAsset
      {
          [SerializeField] public bool useDynamicBatching = true;
          [SerializeField] public bool useGPUInstancing = true;
          [SerializeField] public bool useSRPBatcher = true;
          protected override RenderPipeline CreatePipeline()
          {
              return new CustomRenderPipeline(useDynamicBatching, useGPUInstancing, useSRPBatcher);
          }
      }
      ```

      在自定义的SRP类中，我们需要继承RenderPipeline这个类，然后对其中的Render函数进行重写，Render函数传入一个ScriptableRenderContext参数和一个Camera队列，也就是SRP允许的多个Camera叠加渲染。我们需要创建一个CameraRenderer类，来专门对context和每个camera单独进行处理，同时可以将之前在asset配置文件中自定义的参数一并传入。至此才准备开始真正的渲染部分。
      ```c#
      public class CustomRenderPipeline : RenderPipeline
      {
          CameraRenderer renderer = new CameraRenderer();
          private bool useDynamicBatching;
          private bool useGPUInstancing;

          public CustomRenderPipeline(bool useDynamicBatching, bool useGPUInstancing, bool useSRPBatcher)
          {
              this.useDynamicBatching = useDynamicBatching;
              this.useGPUInstancing = useGPUInstancing;
              GraphicsSettings.useScriptableRenderPipelineBatching = useSRPBatcher;
          }
          
          protected override void Render(ScriptableRenderContext context, Camera[] cameras)
          {
              // 将每个camera单独渲染，≈URP中的scriptable renderers
              foreach (Camera camera in cameras)
              {
                  renderer.Render(context, camera, useDynamicBatching, useGPUInstancing);
              }
          }
      }
      ```

      上一步中我们将context和camera以及一些参数传给了render函数，接下来开始真正的渲染流程。首先我们来简单说一下ScriptableRenderContext，context即常说的上下文信息，context首先会根据传入的摄像机设置渲染器中的相机参数，然后commandBuffer会将渲染过程相关的指令写入到context中，最后context使用submit提交指令将渲染状态和指令提交到GPU。对于渲染状态的更新，最明显的体现在下面这条指令
      ```c#
      public unsafe void DrawRenderers(
        CullingResults cullingResults,
        ref DrawingSettings drawingSettings,
        ref FilteringSettings filteringSettings,
        ShaderTagId tagName,
        bool isPassTagName,
        NativeArray<ShaderTagId> tagValues,
        NativeArray<RenderStateBlock> stateBlocks
      )
      ```
      * cullingResults用来记录物体、灯光、反射探针剔除的结果
      * drawingsetting使用sorttingSettings也就是对世界物体进行排序后的结果来设置绘制顺序，同时设置使用该配置绘制的pass（通过pass中的Tags中的"LightMode"来识别）
        ```c#
        var sorttingSettings = new SortingSettings(camera)
        {
            // 注意，CommonOpaque和CommonTransparent都是先绘制opaque物体再绘制transparent物体
            criteria = SortingCriteria.CommonOpaque // 以opaque绘制模式，从前向后绘制
        };
        // 这里的unlitShaderTagId值为"SRPDefaultUnlit"
        var drawingSettings = new DrawingSettings(unlitShaderTagId, sorttingSettings)
        {
            enableDynamicBatching = useDynamicBatching,
            enableInstancing = useGPUInstancing
        };
        ```
      * FilteringSetting设置过滤参数来渲染指定的layer
        ```c#
        // 只绘制不透明物体
        var filteringSettings = new FilteringSettings(RenderQueueRange.opaque);
        ```
      * isPassTagName作为说明后面的tagValues是pass中的tag-LightMode，还是subshader中的tag-RenderType
      * tagValues和后面的stateBlocks长度匹配，可以说是一一对应的关系。渲染器会比对shader中的tag和tagValues中的值，如果匹配上了就使用对应的stateBlocks中的值。
        ```c#
        if (renderTypes.Length != stateBlocks.Length)
            throw new ArgumentException(string.Format("Arrays {0} and {1} should have same length//后面省略
        ```
      * stateBlocks更改stateblock来重载深度、模板写入方法
        ```c#
        // 以下摘自urp管线drawobjectpass
        if (stencilState.enabled)
        {
            m_RenderStateBlock.stencilReference = stencilReference;
            m_RenderStateBlock.mask = RenderStateMask.Stencil;
            m_RenderStateBlock.stencilState = stencilState;
        }
        ```

      这里说明以下context.ExecuteCommandBuffer(commandBuffer)这个函数的作用，他其实只是把commandBuffer中的指令注入到context中，真正向GPU提交绘制指令和绘制状态的是context.Submit()。也就是说我们可以使用ExecuteCommandBuffer函数来决定在哪个渲染阶段将commandbuffer注入到context中

  2. 关于setRenderTarget的作用，目前来看其实就是先调用SetRenderTarget，然后调用DrawMesh或者其他绘制接口，这样就把绘制结果放在了一张RT或者RTT上而不是屏幕上，然后我们可以对这个RT做任意操作，和Blit其实挺类似的。


* __光追中的透明材质小trick__

  由于在光追中我们会用折射光线去计算一些透明材质，那么如果模拟空心的半透材质，可以使用在原本实心半透物体中间套一个法线方向全部相反的稍小一点的同物体，这样相当于光线从空气->折射介质->空气（空气物体内部）->折射介质->空气。关键就在于需要内部这个物体的法线全部反向，原因在于我们对于单个折射材质的折射率的计算方式是，如果入射方向和法线方向相反，视为从空气射入该半透物体，取折射率的倒数计算；法线反向之后，同样是光线射入稍小的那个物体，但是折射率不用取倒了，也就是相当于从半透物体射入空气。

* __景深-光追__

  首先我们要搞清楚，为什么物体在屏幕上能够清晰锐利，唯一的原因就在于，对于一个像素的多次采样，均落在了一个点的附近，如果一个像素的多次采样得到的结果覆盖了很大的范围，那么这个点当然是模糊的。那么景深需要模仿在透镜焦距处的物体清晰，焦距范围之外的物体模糊，要如何做呢？首先，我们需要设置焦距，也就是实际上近平面到相机的距离，然后我们会把camera视为一个透镜，其实就是把在以camera为中心的一个平面圆上采样作为采样光线的起始点，也就是原来的采样光线起点只有cameraposition，现在是一个范围。

  现在对于近平面上的每个像素，从camera平面圆上射出的光线都会覆盖一个范围，但这些光线又会在一个位置汇聚为一点，那么这个点上打中的物体当然就是清晰的。这个位置在哪里呢？非常显而易见，就是在近平面上，因为近平面上一个像素的采样光线，当然全都必须经过这个像素，这说明什么？这说明所有位置处于近平面附近，或者说距离相机距离为焦距的物体都是清晰的！那么这个相机平面圆是什么？当然就是一个透镜！我们也可以反过来理解，就是处于近平面上物体的一点，只对一个像素有贡献，别的像素采样不到这个点，那么他当然是清晰锐利的。

  这时我们再考虑或近或远的物体，从下面这张图就可以清晰地看出来。
  ![git链接挂了后面再加上图片depthoffield]()

  对于camera平面圆的大小，只影响模糊的程度，不会影响模糊和清晰的范围。平面圆半径=0时其实就相当于单相机成像，也就是没有景深效果。


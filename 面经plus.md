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

* __虚拟内存__
意义：程序可以使用一系列相邻的虚拟地址来访问物理内存中不相邻的大内存缓冲区，这篇缓冲区是大于可用物理内存的内存缓冲区，实际保存在外存磁盘中。当物理内存的供应量变小时，内存管理器会将物理内存页保存到磁盘文件。数据和代码页会根据需要在物理内存与磁盘之间移动。不同进程使用的虚拟地址彼此隔离，一个进程中的代码无法更改另一进程正在使用的物理内存。进程在使用虚拟内存时，会维护一个内存地址，该地址指向实际外存中保存的数据地址。

* __opengl的上下文与状态机__

---

## __渲染性能优化__ ⭐⭐⭐⭐⭐
  
[Unity中DrawCall和openGL、光栅化等有何内在联系，为什么说DC降低有助于渲染性能优化？ - 文礼的回答](https://www.zhihu.com/question/36357893/answer/569416982)

[DrawCall、Batches、SetPassCalls的区别和联系](https://imgtec.eetrend.com/blog/2022/100558021.html)

[什么是setpasscall？](https://zhuanlan.zhihu.com/p/76562300) 

答：drawcall是CPU调用图形编程接口，来命令GPU进行渲染的操作。为了CPU和GPU能并行工作，就需要一个命令缓冲区。命令缓冲区包括一个命令队列，由CPU向其中添加命令，而GPU从中读取命令。这两个过程是独立的，因此CPU和GPU可以互相独立工作。当CPU要渲染一个对象时，可以向命令缓冲区添加命令，而GPU完成了上一次的渲染任务后，就可以从命令队列里取出一个命令执行。

这个命令队列开头一般是一系列设定GPU内部状态寄存器的指令，用来测试诸如是否需要进行ztest，是否需要模板测试，使用哪些buffer作为输入，使用哪些render target作为输出等。在靠近最后的地方，会有一条draw命令。最后可能会有一些与CPU同步的命令，比如更新内存上的某个标志位。

每次调用drawcall之前，CPU要向GPU发送很多内容，包括数据、状态、命令等。CPU需要完成例如检查渲染状态等许多许多工作，而一旦CPU完成了这些工作，GPU就可以开始本次的渲染。GPU的渲染能力很强，因此渲染速度往往快于CPU提交命令的速度。如果drawcall数量太多，CPU就会花费大量时间在提交drawcall命令上，造成CPU的过载。

那么如何减少drawcall？提交大量很小的drawcall会造成CPU的性能瓶颈，即CPU把时间都花在准备drawcall的工作上了。那么很显然，我们可以通过将许多小的drawcall合并为一个大的drawcall，这就是批处理的思路

需要注意的是，由于我们需要在CPU的内存中合并网络，而合并的过程是需要消耗时间的，因此，批处理更适合静态的物体，例如大地、石头等。对于这些静态物体我们只需要合并一次即可，当然，我们也可以对动态物体进行批处理，但是由于这些物体是不断运动的，因此每一帧都需要重新进行合并然后再发送给GPU，这样会对空间和时间造成一定影响。

比如用100个drawcall绘制100个同样材质的物体，那么就如同工厂流水线上要生产100辆同样的车一样，我们是不必在每个drawcall中都重新设置一遍GPU状态寄存器的。对于这种情况，从第二个drawcall开始GPU端的消耗会很小。相反，如果CPU端将这100个drawcall进行合并，那么可能要将这100个物体的顶点数据进行合并，反而可能非常影响性能。比如合并之后剪裁的粒度变粗了，因为可能一些包围盒在视锥外的物体，在合并后的物体有一部分被包括了，所以连带原来需要被剔除的顶点数据一起被传给了GPU，就要渲染更多顶点了，反而增大了GPU的负担（虽然GPU比CPU快很多就是了）。

但是引擎如果要避免不必要的contextroll，引擎就要对GPU的状态进行跟踪管理，并且清楚地知道哪些命令会发生context roll，由于不同GPU在这方面如别很大，这里需要妥协，从而对于通用引擎（怪不得我的1050ti降低drawcall效果显著）来说，降低drawcall往往是有效果的。

<font color="#dd0000">那么如何减少context roll的发生呢？</font>

比如，我们可以根据材质进行排序。将相同的材质的物体放到一起绘制，那么我们就可以大大减少更换传递参数的次数。或者说，直接对于相同的物体使用实例化的方式批处理绘制（见下面）

__从另一个角度看这个问题__

GPU内部也有cache，也有命中和未命中从显存中取数据导致的效率问题。因此，比如我们要换绑一个texture，那么很可能cache miss，就需要重新从显存中读取texture，甚至可能需要从CPU发送一个新的texture数据，这样消耗的资源是很大的。GAMES104提到的gpu性能限制

![image](https://github.com/CHy-KK/Images/blob/main/Imagedasdasdas.png?raw=true)

附：25个导致drawcall增加的情况（没太看懂）https://answer.uwa4d.com/question/5ae1eef932493548fbd32ba1另外，由于UI系统包含大量物体，因此会对drawcall调用产生很大影响，如何优化也是提升性能的关键https://www.bookstack.cn/read/fairygui/17aab488526a353b.md

__从纹理角度来看减少drawcall__

由于每次渲染状态或参数比如纹理设置的改变CPU都需要重新提交drawcall，导致浪费大量时间，所以有很多纹理设置可以减少drawcall。比如最常用的纹理图集，也就是每次把一组纹理放到一张纹理内，然后要使用的时候就按照uv偏移来找要其中哪一张。这样就可以降低纹理频繁切换带来的消耗。

### __drawcall、批次 与 setpasscall__

* drawcall：实际上就是一次图形API绘制指令的调用，比如opengl的glDrawArrays或者glDrawElement之类的。
* batch批次：在调用Drawcall之前需要进行渲染状态的设置，我们把设置渲染状态，加载网格数据然后调用drawcall的过程叫做一个批次。一般来说drawcall数量=batch数量，但也有例外情况，比如一个模型网格过于巨大，一个batch要多次drawcall才能进行绘制。
* setpasscall：shader中有pass的概念，比如一个shader有三个pass，那么应用这个shader的材质就会按照给定的pass顺序渲染3遍，因为每个渲染批次都会设置一个pass，一个pass就会有新的渲染状态，那么就必须要切换渲染状态，于是一个新的pass需要开启一个新的批次。所以setpasscall会引起batch增加，但是batch增加不一定要改变渲染状态，因为可能连续渲染几批相同材质的物体，也就是下面所提到的合批。注意，同一个one pass shader下，shader参数发生了变化也会触发setpasscall。目前能够合并同一shader不同mat的只有SRPbatcher。

### __静态批处理 vs SRP Batch vs GPU Instancing vs 动态批处理__

[关于静态批处理/动态批处理/GPU Instancing /SRP Batcher的详细剖析](https://zhuanlan.zhihu.com/p/98642798)

简单来说，我们主要要减少的是setPassCall，因为实际上消耗资源的是切换GPU的shader参数，也就是setpasscall的内容。即使我们什么都不做，GPU在渲染连续两个连续的完全相同的材质时也不会调用setpasscall，但是会调用drawcall，而单纯的调用drawcall是不太消耗资源的。可以根据下图右边的材质（同颜色表示相同材质）渲染顺序来计算sepasscall的数量，注意有两个必存在的setPassCall——DrawGL和绘制天空盒。
![image](https://github.com/CHy-KK/Images/blob/main/setPassCall.png?raw=true)

* SRP Btach能合并使用同一shader的不同mat，不同mesh的物体，但没有减少drawcall，只是减少了切换mat导致的setPassCall。其中SRP batch在GPU中保留Cbuffer中的properties以及不同物体的空间转换矩阵（M），从而避免每个drawcall都要将shader信息从CPU发送到GPU；每个物体对于该保存地址对应有一个offset，渲染该物体的drawcall只需要传送这个offset查询对应数据即可。因此不会减少drawcall数量，只会减少setPassCall的数量，而原来是每个材质（注意不是每个物体）一次setPassCall
* GPU Instancing能够使用一个draw call来绘制所有使用相同mesh和相同material的物体。CPU将所有使用同一mesh物体的空间变化（objectToWorld）和材质信息放入一个发送到GPU的数组中，GPU会遍历数组绘制材质。从而避免了由切换渲染物体导致的drawcall。GPU instancing根据平台不同有数量限制，如果超出最大值会分成两个drawcall。所以GPU instancing是通过使用数组来减少draw call，而非其他合批想要解决set pass call。
* Static Batch能合并所有相同material不同mesh的vertex/index buffer合并为一个大的vertex/index buffer，然后会记录每一个子物体在大buffer中的位置，然后只需要一次setPassCall，并调用多次drawcall来绘制每一个子物体。但问题在于合并之后的物体会额外占用很大的内存空间，还会导致打包之后的应用体积变大（合并的数据保存到本地了）。
* Dynamic Batch能够使用一次drawcall绘制多个不同mesh但使用同一材质的物体。他会先将共享材质的模型顶点信息变换到世界空间，然后调用一个drawcall进行绘制。但是由于顶点变换是由CPU完成的，会占用额外的CPU开销。一个批次的动态物体顶点数是有限制的，以下是开启和不开启的对比（sphere由于网格过多所以无法使用dynamicbatch）。对比两张图我们可以清晰的看到，开启DB甚至比不开还慢，且同样都是33个setPassCall。这是因为，首先不开启DB时，物体的渲染顺序也会被调整为优先渲染同一材质，也就是保证了最少的材质切换。

  动态批处理和静态相比，减少了预先复制模型顶点到一个大vertexbuffer的过程，减少了内存占用和打包体积，但CPU的性能消耗不小，静态批处理在这一点上会更高效。但是两者都是采用把小物体打包成大物体的方式进行合批，
  ![image](https://github.com/CHy-KK/Images/blob/main/dynamicBatch_sphere.png?raw=true)
  ![image](https://github.com/CHy-KK/Images/blob/main/dynamicBatch_cube.png?raw=true)

* 一些打断批处理的情况
  1. 物体如果都符合条件会优先参与静态批处理，再是GPU Instancing，然后才到动态批处理，假如物体符合前两者，此次批处理都会被打断
  2. 拥有lightmap的物体含有额外（隐藏）的材质属性，比如：lightmap的偏移和缩放系数等。所以，拥有lightmap的物体将不会进行批处理（除非他们指向lightmap的同一部分）
  3. 使用Multi-pass Shader的物体会禁用Dynamic batching，因为Multi-pass Shader通常会导致一个物体要连续绘制多次，并切换渲染状态。这会打破其跟其他物体进行Dynamic batching的机会

* 透明物体的批处理：半透明的着色器要求物体按照从后往前的次序来渲染，所以合批时会严格根据半透明物体的次序合批。也就是说，两个使用相同材质的半透明物体，如果排完序后，中间有使用其他材质的半透明物体，那么这两个物体就不能合批。

### __set pass call传递的到底是什么？batch传递的是什么？__
一次setpasscall只有设置渲染器参数的指令，并不包括纹理、顶点数据，只是由于setpasscall会导致纹理、顶点数据的切换，所以这里也放入消耗的内容，主要包含以下内容（以下内容按照数据量从大到小进行排序）:
1. 消耗最大的是传递纹理，但是纹理并不一定需要每次都传，如果之前有材质用到过该纹理那么可以复用，或者传递一张包含多张纹理的巨大纹理（<font color="#dd0000">纹理加载发生在光栅化之后，fragment shader之前，所以其实不算在setpasscall的内容</font>，只是setpasscall一般发生在切换shader时，伴随着的可能要切换纹理，非常消耗性能）。
2. 顶点数据：由于setpasscall会引发批次变化，所以我们当然需要传递顶点数据给GPU，但是相对纹理数据来说会小一些，具体传递的顶点数据包括：
    * VBO（vertex buffer object）：所有的顶点属性，包括模型空间坐标、uv坐标、normal、gpuInstancing id等等
    * VertexAttribPointer：告诉OpenGL应该如何解析VBO，即每一个属性数据在一条顶点数据中的偏移量
    * EBO（element buffer object）：也叫IBO，即Index。如果没有EBO，VBO需要列出每一个三角形的顶点数据，如果有相邻三角形，那么就会出现三角形顶点重复，那么就VBO内就会有重复数据导致数据量过大。如果使用EBO，VBO中只需要存储不同的顶点数据，然后由EBO来解释每个三角形使用了哪一个顶点。
    * VAO（vertex array objecy）：包裹上面三个东西的结构。
（<font color="#dd0000">VBO实际上是在渲染管线的几何处理阶段，vertex shader之前发送给GPU的</font>）
3. uniform数据，即我们需要给渲染器传递的参数，包括空间变换矩阵、光照参数、自定义pass用到的shader参数、使用的纹理单元（一般使用TextureID标识）等。
4. 渲染状态：包括深度测试、模板测试、颜色混合模式、裁剪状态等一系列渲染状态测试。
5. 切换渲染目标：
    1. 新RenderTarget的指针或索引：CPU需要告诉GPU要切换到哪个RenderTarget，这可以通过RenderTarget的指针或索引来完成。这个RT可以是提前被分配好的。（注意这里不要考虑将RT写入到本地texture上）
    2. 清除颜色和深度值：如果切换到的RenderTarget需要清除，CPU需要告诉GPU清除颜色和深度值的数值。
    3. 视口设置：如果RenderTarget的视口不同于当前的视口，CPU需要告诉GPU新的视口设置。
6. 切换使用的shader pass：由于shader在编译阶段就变成指令码从CPU发送给了GPU，并存储在显存中，所以在setpasscall时需要切换使用的shader的话，只需要指定shader在显存中的地址即可。

* __GPU会从内存中读取那些数据？__

  顶点数据（坐标、法线、uv、颜色、顶点索引等）、纹理数据、渲染目标（frame buffer、depth buffer）、shader、uniform数据

* CPU与GPU的单向数据传递GPU计算速度的确比CPU快很多，所以可能有的想法就是先让CPU将数据传到GPU中进行计算，等GPU计算完后CPU再将结果读取回来，再基于这个结果进行判断，之后再告诉CPU如何绘制，这个过程叫back-force。由于现代引擎中render和logic是不同步的，但是如果有哪个render步骤需要等back-force的话，这样做会导致有半帧或者一帧的画面和逻辑不同步的现象出现。因此在设计代码时，尽量保证数据的单向传输（CPU -> GPU）,避免计算同步问题，且不要从显卡中读取数据。

* __GPU driven？__
* __⭐⭐⭐⭐⭐GPU底层硬件__
  
  https://www.bilibili.com/video/BV1aM4y1g75f?spm_id_from=333.999.0.0&vd_source=a496344996aebd6de06a773bff299dc1

* __OpenGL的批处理实例化：https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/10%20Instancing/__
  
  简单概括来说，比如我们要渲染一堆草的模型，我们肯定不能一次传入一根草的模型，而是一次性向GPU传一批草。可以使用glDrawArrayInstanced或者glDrawElementInstanced，最后一个参数给定一组顶点数据需要渲染的次数amount。然后在vertexshader中我们需要对传入的顶点数据进行位移，否则即使渲染再多次他们也是叠在一起的，看不出来。我们可以使用gl_InstanceID这个自有变量，这个变量会告诉我们当前这个顶点属于第几个要渲染的实例，然后使用uniform传入一个amount大小的位移矩阵数组。我们也可以传入的一个amount大小的位移矩阵（Model矩阵）数组，也就是使用顶点属性来传入一个mat4矩阵。这里由于顶点属性最大允许的树大小等于一个vec4，因此我们需要绑定四个顶点属性，然后
  
* __是否每一次drawcall都要重新传一次顶点数据？（VAO和VBO到底是如何将数据传给gpu的？）__

  // 该答案写于很早之前不一定有参考价值
  
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
  
  相对法线贴图增加了模型各个顶点的高度信息（__记录的是实际高度距离建模平面的值，实际高度=1-采样值，也就是不可能在最初命中点的后面会有比该点更高的 且  能挡住该点的点，所以每次采样只需要向前查找，不会需要向后查找__），看起来会有遮挡关系。实现方式是通过根据A顶点处其高度值以及viewDir计算出一个偏移量，取偏移处顶点b的uv作为纹理采样的uv，而实际应该取的是B处的uv，可以看出来偏差其实还是挺大的，所以偏移法精度比较低。

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

  [在Unity中使用Tessellation - 异次元的归来的文章 - 知乎](https://zhuanlan.zhihu.com/p/542453747)

  几何着色器在曲面细分着色器(tessellation shader,包含两个可编程的着色器：hull shader和domain shader)之后（如果没有做曲面细分则是在vertex shader之后），在fragment shader之前（顺便光栅化在fragmen shader前一步执行）。
  
  几何着色器以一个图元为输入，输出一个或多个图元。要注意的是，由于一般vertex最后输出的是clipsapce下的坐标，然后fragment接受，所以理所当然的，geoshader输出的也应该是clipspace下坐标，记得做转换，同时将vertex的输出改为不做转换的坐标，其实就是将toclipspace延迟到了geoshader中做。

  hull shader接受几何着色器输出的图元为输入，返回值为图元中的一个控制点，可以通过若干指令来控制输出控制点的信息。包括：输出控制点的数量（如果输入三角形图元那么就是3）、GPU细分三角形的方式、用哪个函数（patch constant function）来细分每个图元等等。
  
  patch constant function是人为定义的，接受一个图元为输入，输出一个叫TessellationFactors的结构， 该结构定义了三角形三条边的细分系数和三角形内部的细分系数。细分系数即一个图元需要被细分为几个新的图元。在grassshader中三角内部和边的细分细数是相等的，还不确定不等的情况是啥样的。

  domain shader其实就是完成类似之前vertex shader做的事，会把新细分出来的顶点插值计算出顶点属性值，此处会引入重心坐标进行插值计算。

  曲面细分的算法：[Games101笔记：曲面细分、简化 - 所以然的文章 - 知乎](https://zhuanlan.zhihu.com/p/382074891)

  可以大致概括为：取图元边上中点（Catmull-Clark算法还会取图元平面中心点）然后连接出新的图元，在获得新图元之后，分别对细分出来的新顶点和原有顶点进行坐标的调整。一般来说顶点的度越大周围顶点的坐标值在调整中权重越大。

  几何着色器如何知道TriangleStream里的vertex是如何组成三角形的？答案是每个新的vertex将和前两个vertex组成三角形。如果需要更多三角形流（triangle strip），可以在TriangleStream中使用RestartStrip函数
  
  ![image](https://github.com/CHy-KK/Images/blob/main/grass-construction.gif?raw=true)
  
* __SSAO__
  
  先对屏幕空间下进行世界坐标重建，然后对于每个像素使用法线贴图取法线信息（需要deferred pass），然后使用法线建立切线坐标系，按照给定的采样半径在半球上采样，对于采样点计算剪裁空间坐标，w值为深度值，然后除以w在规范到01空间得到ndc坐标，使用xy取得对应位置深度图上深度，与采样点深度比较，若小于说明遮挡住，ao+=1。

  优化：length(采样向量)得到weight值，即采样点距离越大ao越没用；同时要忽略掉深度差距过大的情况，这种情况下不会对ao做出贡献；双边滤波模糊

  引擎中的AO：烘焙，优势在于不受物体本身的UV影响，操作简单，缺点在于无法烘焙静态物体
  
  SSAO：灵活，实时，但效果不佳且消耗大(烘焙完用贴图就行)



* __为什么后处理把图像渲染在一个覆盖全屏幕的三角形上比用两个三角形更好？__

  因为这样可以有效节省两个三角形边界像素的overdraw，我们需要知道GPU处理像素时并非是一个像素一个像素处理的，而是以2 * 2(quad)或者8 * 8为一组像素并行渲染的，所以会被overdraw的并非只有边界上的一个像素，而是靠近边界上的一组像素都会被绘制两次（光栅化和fragment部分应该都会？）。
  
  另外需要注意，这个全屏幕三角形无论多大都无所谓，因为只要他只覆盖了屏幕上的像素，那么需要处理的就只有屏幕上的像素数量，哪怕是屏幕的1k倍也无所谓。

## __OverDraw问题__

为什么要减少overdraw？前面提到过，overdraw就是一帧中同一个像素被重复绘制的次数。所以解决overdrwa就是为了解决GPU端像素填充率和计算量的瓶颈问题。像素填充率就是GPU在一帧之内可以向帧缓存写入像素的数量，一般以百万像素/s或十亿像素/s来衡量。通常按照光栅化单元*该GPU的核心频率来计算。而每一个像素至少要执行一次光栅化，片元着色器和输出合并三个阶段，所以降低需要参与光栅化渲染的像素数量使性能优化的重中之重。

### __1. 不透明物体overdraw优化__
对于不透明物体，最简单的手段之一即是从前向后排序，这样可以免去被遮挡的片元进行深度测试之后的blend等操作，并不用写入帧缓存直接被抛弃。

* early-z：其改进之一是使用early-z方法，在进行片元着色之前先做一次深度测试，保留通过测试的片元进行着色计算。实用early-z之前最好还是将物体从前到后进行一次排序，这样可以保证early-z发挥最高效率，如果反过来还是会把被遮挡片元写入深度缓冲区和帧缓冲，这样会损失一部分性能。如果开启alpha test这类丢弃片元的测试，或者手动写入深度值等，gpu会自动关闭early-z，因可能导致early-z的结果不正确。

* pre-z：另一种方式是pre-z，即在开始正式的渲染之前，先用一个pass只写入深度，然后在正式渲染中深度测试选择equal。但是因为选择了多pass渲染，所以会打断批处理，使用的时候要注意。

### __2. Quad Over Draw__
[那些关于OverDraw的前世今生（2） - Coresi7的文章 - 知乎](https://zhuanlan.zhihu.com/p/76562370)

那么实际渲染中，导致overdraw的元凶是什么？首先上面提到过，GPU渲染片元的时候并不是一个片元一个片元渲染，而是一个quad一个quad渲染，一个quad一般包含四个映射到相邻像素的片元。不考虑抗锯齿的情况，就是每个像素中心属于哪个三角形，该像素就是什么颜色。那么现在有一个问题，那就是如果一个像素被多个不同三角形的片元覆盖到，那么这个像素就会被绘制多次，而由于gpu一次绘制一整个quad，所以其实是有四个像素需要多次重新绘制。在每一次绘制一个三角形之后，gpu会丢弃像素中心不属于该三角形的片元。考虑下图，一个quad被四个三角形都覆盖到了，那么就一共要绘制4*4=16次，而真实情况会更加复杂，消耗更加亏贼！而且如果这几个三角形属于一个材质还好，如果分属无法合批的材质，那么每次还要setpasscall，其消耗究极亏贼！还有个问题是GPU不一定使用2x2的quad，也可以用更大的范围比如8x8来进行光栅化，那消耗就是超级加倍的亏贼！
![image](https://github.com/CHy-KK/Images/blob/main/quadoverdraw.jpg?raw=true)

### __3. 其他情况下的overdraw__

* forward render pipeline中的多光源渲染，需要对每个光源重绘一边遍。
* 投影时的overdraw（后面再查）

### __遮挡剔除__

[剔除：从软件到硬件 - 洛城的文章 - 知乎](https://zhuanlan.zhihu.com/p/66407205)

注：early-z、z-culling都是硬件层面的剔除

#### __Occlusion Query__

OQ的全过程还是建议看文章，因为我没太搞明白，尝试简单概括：

* 我们首先用一个depth only的pass绘制整个场景，并在绘制前后插入occlusion query的指令，在绘制结束之后GPU会返回一个passed sample count，即通过深度测试的采样数目，来判断标记的某个物体是否完全被挡住。然后在正常的渲染流程中剔除被标记为完全遮挡的模型。

  OQ有一个显而易见的优化是可以采用包围盒深度来代替模型深度，可以减小查询所需要消耗的计算量。另一个缺点是由于需要从GPU读取查询结果，GPU向CPU传递数据是非常慢的，可能会导致CPU等待GPU导致性能损耗。想解决这个问题一个方法是每一帧使用上一帧的OQ结果，虽然对于运动较快的场景可能会出错。但由于是包围盒所以总体来说影响也不明显，UE4就是默认用这种方案。但是还有另一种方案，将渲染队列中上一帧可见的物体直接进行渲染，对于上一帧不可见的物体先插入一个查询到查询队列中，当我们没有可以直接用于渲染的模型时，再去读取查询结果并更新查询物体的可见状态，并渲染可见物0体。

  这里的疑点在于，到底是什么时候插入的OQ？是在depth only pass直接进行所有查询？还是一边渲染可见物体一边进行查询？相对于pre-z也是先绘制一次depth only，为什么说pre-z的depth only pass可能会打断批处理，不可以也对整个场景绘制深度然后再正常渲染吗？




### __移动端TBDR解决overdraw__
见下方TBDR架构篇章

### __其他渲染路径解决多光源overdraw__

* 延迟光照

* forward+

  [Tile Base Render (Forward+) - 不知名书杯的文章 - 知乎](https://zhuanlan.zhihu.com/p/553907076)

  先利用深度对tile内无关光源进行剔除，只对tile有影响的光源进行光照计算。大大降低了多光源的计算压力。主要分三个阶段：depth prepass、light culling、final shading。

  ![image](images/forward%2B.jpg)

  * depth prepass：深度贴图pass
  * __light culling__：核心步骤。将屏幕划分为16*16的tile，在每个tile中寻找影响该tile的光源，形成一个light list，后续在计算该tile中的像素着色时只需要计算该tile的light list中的光源即可。light culling需要用compute shader进行计算，根据屏幕空间坐标和深度重建世界坐标我们可以得到每个tile对应的视锥体（其实一般在view space进行，不需要到世界坐标），然后会用compute shader使用depth map计算出一个tile中的最大和最小深度值，就可以构造出一个tile的实际视锥体了（类似一个包围盒），最后我们使用光源球体（即光源坐标以及光照半径构成的球体）与每个tile视锥体进行求交，如果有交就把该光源加入tile的light list（球体求交不用我说了吧）
  * final shading：每个tile根据light list计算着色

* 群组渲染



## __移动端TB(D)R架构__

TB(D)R架构tile based （defered） rendering
  
在正式开始介绍TB(D)R架构之前我们先介绍几个概念

* Soc（system on chip），将gpu、cpu、内存等其他手机硬件模块组合在一起的芯片，其中gpu和cpu共享一片内存地址，但都有各自的SRAM的cache缓存，也叫on chip memory，一般几百k~几M大，读取速度比内存读取速度快几十到百倍。在TBDR（延迟TBR）下，on-chip memory会存储Tile的颜色、深度和模板缓冲，读写修改速度都很快。

* 像素填充率 = ROC运行时钟频率 * ROP个数 * 每个时钟ROP能处理像素的数量（rop即光栅化处理单元，就是光栅化元件）

* Stall：当一个GPU两次运算结果之间有依赖关系而必须串行时的等待过程。

* 移动端是CPU、GPU分离式的架构，也就是说GPU和CPU之间交互需要数据传输，当然他们都在SOC中。但是GPU自己没有显存。

* IMR（immediate mode rendering）：PC端GPU渲染架构，上面紫色的部分对应正常的渲染管线，灰色的部分对应GPU的独立显存，包括几何信息、纹理信息、深度缓存和帧缓存。这种架构下的信息传输特点就是对于每一个具体的绘制，GPU会直接和显存进行数据传输，箭头即表示读取和写入。在这种架构下，每一次渲染完的color和depth数据要写入frame buffer和depth buffer会占用很大的带宽消耗，所以IMR架构中也有使用L1 cache和L2 cache俩优化这部分大量消耗的带宽（先写入cache，然后结束后再写入内存）。同时，由于传统渲染管线是处理完一个图元立刻把生成的fragment进行着色处理，所以IMR的执行效率非常高。

![image](https://github.com/CHy-KK/Images/blob/main/IMR.png?raw=true)

那么移动端可以沿用IMR架构吗？答案是否。由于手机的功耗不足以支持GPU和显存传输数据需要的大量带宽，所以移动端需要一种新的架构TBR。
注：带宽单位为bps(bit per seconds)，即每秒传输的bit率，用来描述数据传输速度。TBR架构根本上节约带宽的方法就是使用更高速带宽的cache进行读写，最后再从cache将buffer写入低带宽的内存，而不是直接和低带宽的内存进行读写。


### __TBR__

我们还是要先看懂架构图，最下面一层代表的是系统内存，也就是说TBR的几何信息、纹理、帧缓存等是保存在系统内存中，而非显存中，因为移动端的TBR架构没有独立显存，或者说TBR的显存和系统内存实际用的是同一片物理内存，但是GPU在逻辑上有自己独立的一片内存区间，这片内存区间由GPU来驱动管理。

那么现在GPU要如何读取渲染数据呢？那就是图中中间的那一层：On-Chip Memory。On-Chip Memory可以理解为L1、L2 cache，但是这个缓存空间很小，所以TBR就利用这个特点来做渲染。简单来说就是TBR会将屏幕分为数个16 * 16或32 * 32的像素块（tile-瓦片）来渲染，然后把渲染结果存储到On-Chip Memory的depth buffer和color buffer中。也就是TBR从原来IMR对 __显存内__ 的Color/Depth buffer进行读写变成了对 __高速缓冲On-Chip Memory__ 的读写，减小了最影响性能的系统内存传输的开销。注意，TBR中的深度测试使用的是对每一个tile进行Early-Z，可以看到visibility test在texture and shade环节之前。

![image](https://github.com/CHy-KK/Images/blob/main/TBR.jpg?raw=true)

在TBR架构中，并不是来一个drawcall就直接绘制好所有tile的，考虑下图，其中DRAM就是物理内存。TBR的策略一般是对于CPU过来的绘制，先只对他们做顶点处理，也就是图中的Geometry processor部分，然后存回内存，在一定要刷新framebuffer的时候，也就是系统告诉GPU现在一定要用framebuffer时，GPU才知道不能拖了，然后读取这片几何数据并做光栅化，然后tile based rendering。

![image](https://github.com/CHy-KK/Images/blob/main/TBR-on-chip%20memory.jpg?raw=true)

说完了这些，应该就能理解为什么TBR能降低功耗了，对于上图中TBR和系统内存进行数据传输的箭头，读取只发生在需要几何及纹理信息时，写回也只发生在整个frame buffer都渲染完成之后，带宽消耗最大的depth buffer和color buffer读写都发生在On-chip memory上。当然我们必须要认识到一点，TBR只是为了解决功耗难题，但实际上效率最高，处理速度最快的方案还是IMR架构直接在DRAM上读写，分块渲染是牺牲了效率换区带宽功耗。

### __TBDR__

TBDR在TBR的基础上再加上了一个deferred。TBR主要是为了解决IMR架构的带宽问题，而TBDR则是为了解决overdraw的问题。

我们可以看到在绘制管线中对了一个新阶段：HSR & Depth Test。我们一般解决overdraw的方式，也是TBR使用的是early-z，但是early-z无法完全解决，因为真正的复杂场景中，我们是没有办法根据每个物体与摄像机的距离来对他们的绘制进行排序的。所以HSR被提出了，不需要在软件层面对物体进行排序，而是直接在硬件层面做到 0 Overdraw。

![image](https://github.com/CHy-KK/Images/blob/main/TBDR.png?raw=true)

* HSR流程：当一个像素通过了early-z，在进入fragment shader之前，先不进行绘制，而是指标记该像素应该由哪一个图元来画，也就是上面结构图中的tag buffer，等到这个tile上所有的图元都处理完之后，再开始绘制每个已经被标记好属于哪一个图元的像素点。我们可以看到texture and shade节点会读取primitive list（图元列表）和vertex data进行绘制。

至于为什么TBDR能进行这种像素级别的深度测试，是因为每次光栅化和着色计算时我们拿到的都是整个场景的图元和顶点数据（我们在TBR说过在几何阶段之后会先把一帧所有的几何数据存储到内存中，等之后有需要时再进行光栅化），而非IMR模式下顶点、图元、光栅化、片元的流水线处理，IMR架构没有办法对一个tile所有像素都深度测试完之后再着色计算，<font color="#dd0000">__即使是TBR的early-z也是每个tile的每个片元通过深度测试之后立即进行着色，这就是TBDR中的Defferd的来源，即将每个tile所有的像素着色都延迟到HSR之后，再利用tag buffer进行着色计算__</font>。


TBDR渲染流程（宏观）：

1. 从内存读取顶点数据，执行所有与几何相关的处理得到所有的primitive list（图元列表，一般来说就是三角形列表）并确定每个tile上有哪些图元，然后写回内存。
2. 逐tile执行光栅化及其后续处理并写入tile buffer，在完成所有tile后从tile buffer写回到system memory也就是frame buffer中。




* TBDR优势：

  * 给消除overdraw提供了便利（overdraw即同一个像素多次着色，给gpu很大的压力，减少被遮挡像素的texturing和shading可以解决overdraw）
  * cache friendly，由于tile buffer在cache中读写很快，以降低render rate为代价，降低带宽，省电。也就是慢一点，但是功耗小一点。

* 缺点：
  
  * binning是在vertex之后，要将几何数据写入内存中，然后才被fragment shader读取，其中需要很多几何数据的管线， 容易在此处有性能瓶颈。
  * 一个图元可能在几个tile交界处，那么每个tile都要绘制一次该图元，也就是总的渲染次数会多于IMR。

TBDR的两个defer过程（或批处理过程）
* binning（类似四叉树）：确定一个图元由几个tile渲染（覆盖了几个tile）
* early-depth-test（early-z?）

__⭐⭐⭐⭐⭐移动端优化方案__：https://www.bilibili.com/video/BV1Bb4y167zU/?p=2&spm_id_from=pageDriver&vd_source=a496344996aebd6de06a773bff299dc1 最后十分钟



## __渲染瓶颈优化问题__（见RTR3提炼总结12章）

### __瓶颈定位__
基础逻辑是，判断一个环节是否出现瓶颈，那么就让该环节的负载大大减小，如果性能提升非常大，那么瓶颈就出在这个环节。反之则是，如果减少其他环节的负载而性能差别不大，说明其他环节没有瓶颈。

####  __1. 光栅化阶段瓶颈定位__
光栅化阶段分为三个阶段：三角形设置，光栅化，fragment shader。其中三角形设置就是把顶点链接成三角形，几乎不会是瓶颈。
* 测试光栅化操作是否为瓶颈：光栅化操作的主要瓶颈与帧缓冲带宽相关，即将片元数据（颜色、深度）写入显存中的传输速度。将颜色输出的位深度从32/24位减小到16位，或者改变深度缓冲的位深度。（只找到了申请RT中颜色位数，没找到在哪改相机输出）因为颜色深度越大，光栅化需要将片元数据传递到显存中的数据量就越大。

* 测试fragment shader：改变屏幕分辨率。减少会产生的片元数量，如果瓶颈在这里，那么性能会被大大提升。

* 测试纹理带宽（渲染管线中，加载纹理通常发生在fragment shader之前，光栅化之后）：内存中出现纹理读取请求时就会消耗纹理带宽，一般会让纹理访问更高的mipmap等级（更模糊）来测试。[如何指定mipmap等级偏差](https://www.xuanyusong.com/archives/4699)，在代码中可以通过texture.mipMapBias来设置。


####  __2. 几何阶段瓶颈定位__
几何阶段有两个主要区域可能出现瓶颈：顶点与索引传输和顶点变换阶段（vertex shader）。

* 测试顶点与索引传输：通过调整顶点格式大小来判断是否传输是应用程序的瓶颈。我的理解是在VBO里塞一些没用的数据。这部分瓶颈也可能在CPU上，可以通过CPU降频来试试看。

* 测试vertex shader：通过添加一些无意义的指令来加重vertex shader负载（注意要添加"有意义"的指令，以防被编译器优化掉）。顶点处理的速度与GPU核心频率有关。

####  __3. 应用程序阶段瓶颈定位__（这部分建议直接看文章）

CPU阶段可能产生瓶颈的原因：过多的物体需要进行排序、空间结构非常差/没有使用加速结构、合批要打包物体顶点数据等等

* 用profiler工具查看CPU占用情况

* 对CPU阶段降频，如果性能和CPU速率成正比，那么瓶颈就在CPU。当然，降频也可能导致之前一个不是瓶颈的阶段成为瓶颈。

* GPU阶段没有瓶颈的话，CPU阶段就是瓶颈。

### __瓶颈优化__

#### __1. CPU优化__

1. 减少CPU和GPU之间的同步，减少CPU等待GPU的时间，即减少CPU访问GPU正在占用的资源。（不过一般来说都是GPU等待CPU指令）

2. 批处理。由于CPU提交的总顶点数量是一定的，那么最大化每个batch提交的顶点数量，减少提交的batch数量，那就能减少CPU的性能消耗，也就是前面批处理章节说的减少状态切换。具体操作包括合并纹理减少因为纹理切换打断批处理、使用shader varients（具体原因是什么？）对一些着色器常量使用lookup table(LUT)方法

#### __2. 应用程序阶段优化__

这一部分的内容其实更像是在说明C++程序应该如何优化。

* 内存优化：
  * 在代码中连续访问的内容在内存中也应该连续存储，以避免多次寻址甚至cache无法命中等造成的性能损失。经典范例一个二维数组逐行访问比铸列访问要快，原因就在于逐行访问元素地址连续，而逐列访问地址不连续要计算偏移。

  * 默认的分配内存（malloc free、new delete等）比较慢，建议使用内存池。

  * 尽量避免在渲染循环中分配或者释放内存。可以在进循环之前就先分配好要用到的一些数组等其他数据结构对象。

  * 对数据结构使用不同的组织形式

* 代码优化：
![image](https://github.com/CHy-KK/Images/blob/main/codeOpt.png?raw=true)

#### __3. 图形api优化__

这里说的其实也是批处理，但是给出了一些使用instancing的具体场景。例如在渲染多人同屏时，我们可以对所有人用统一的材质只改变人物模型贴图来使用GPU实例化。而因为我们可以把所有人的贴图都合并在一张贴图中，所以所有人使用的是完全一致的材质，而非相同shader不同贴图的相异材质。这种被称之为共享材质，这样共享纹理可以减少纹理缓存的抖动。

另一种应用场景，比如当一个物体不会形变，或者形变完全可以由shader来完成，那么可以直接在GPU中存储物体的数据，而不用每次都由CPU把物体形变的模型数据传递给GPU。这样这个物体的顶点缓冲对象VBO就可以存储在GPU的静态缓冲区，以便更快的读取。这个场景的范例就是使用几何shader进行草渲染。

### __4. 几何阶段优化__

* 降低各个几何变换使用到的数据精度，包括空间变化、光照计算、几何shader变换、不可编程的裁减过程等

* 利用视锥裁减、occlusion clip、LOD等简化模型或者减少需要发送给GPU进行渲染的顶点数量。

* 利用缓存感知布局算法，将顶点按照cache友好的方式进行排列以提高cache的命中率。

* 将顶点数据存储在压缩格式中

* 对于一些每帧只改变一次的计算放到cpu来做，比如方向光源到view space的方向计算，不需要在每个顶点shader都做一次重复工作。

### __5. 光照阶段优化__

* 对于静态物体（标记为static）使用lightmap，生成一张在光源上的光照贴图，类似阴影贴图一样采样得到片元的光照信息。lightmap只存储亮度值，不存储颜色值。但是因为是静态物体，所以对整个场景只需要计算一次lightmap。

  对于动态的阴影，则需要每一帧对实时光源生成一次shadowmap。两个贴图可以一起使用。

* 减少光源数量

* defered render

* 应用顶点光照而不是像素光照

### __6. 光栅化阶段优化__

* 背面裁剪：注意，由于计算每个三角形是否是正面朝向也要消耗一定计算性能，所以当全部都是正面物体的时候，开启背面裁剪反而会加重消耗性能。

计算朝向只需要将转换到view space的顶点坐标，按顶点顺序连成两条向量，计算叉乘正负即可。比如如果我们规定正方向是顺时针也就是叉乘为正（相机空间正方向跟世界的z轴正方向有锤子关系）。

* 不清除frame buffer的渲染。如果能确保屏幕上的每个像素都会被覆盖，那么可以不用进行clear操作。比如先渲染天空盒，然后渲染室内场景，由于天空盒会被完全覆盖，所以可以直接对frame buffer进行写入（不记得天空盒之后有没有clear操作了，但是类似情景是存在的）

* 进行合适的纹理压缩。

* 基于距离使用简化的shader（这个可能用shader variant实现？），注意不是简化模型，而是简化shader，减少需要进行的光照计算，取消specular等。lod shader

#### __加速fragment shader__

* 对于一些需要使用复杂shader的物体，可以考虑在进行正式的颜色渲染pass之前，先进行一个仅包含深度通道的pass，在高深度复杂场景中，可以减少需要进行复杂着色的片元数量，这种方法也就是pre-z。当然要注意一点，使用多pass会打断批处理。

* 使用LUT

* 将片元工作移动到vertex shader

* 减少精度

* 不需要进行归一化向量时就别归一化

* 



---
## __SRP__
  
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
  ![image](https://github.com/CHy-KK/Images/blob/main/depthoffield.jpg?raw=true)

  对于camera平面圆的大小，只影响模糊的程度，不会影响模糊和清晰的范围。平面圆半径=0时其实就相当于单相机成像，也就是没有景深效果。


* __raytracing BVH思路总结__

  BVHnode同样也是继承自hittable类，因为我们想构建的BVH树的叶节点是正常hittable的子类，也就是sphere、运动sphere这些。

  我们构建BVH的方式是，将之前随机生成的hittableList列表作为参数传递给BVHnode，在BVH的构造函数中，会对hittableList中的objects以某一个维度(随机x y z)先进行一次排序，然后将objecs列表对半分位两个子列表，然后再对两个子列表递归的构建两个子BVHnode。当objects长度<=2时，也就是需要构建非BVHnode的叶节点（别忘了我们的节点类型是hittable，而BVHnode继承自hittable）。这样我们就完成了第一步，自上而下的构建BVH结构。

  在构建了BVH结构之后，我们还要对每一个BVHnode计算包围盒，这一步是自下而上的，先从叶节点开始计算。因为我们的项目比较简单所以暂时只有运动/静止的sphere，sphere的aabb非常好构建，就是origin ± vec3(radius)。运动的sphere按照时间计算origin即可。构建完叶节点的aabb之后，递归结束后上一层会根据子节点构建的aabb计算包裹两个子aabb的父aabb，然后直到最上层最大的那个aabb。至此我们完成了整个BVH的构建。

  由于BVHnode继承自hittable，所以也有hit函数，BVhnode的hit就直接调用其aabb属性的hit即可。然后也要同样递归地计算两个子节点的hit。由于我们只有一个BVH结构，所以投射的光线都是先击中最大的那个BVHnode（头结点）然后向下查找，直到击中某个叶节点或没有。hit函数中引用传递的hitrecord参数会记录t最小的那个击中点。然后就是正常的颜色计算等等，这就是一个完整的BVH结构的raytracing过程。

* __一些空间加速算法的总结__

  大题上看这几类结构构建的树其实都很容易在增加、删除节点之后出现不平衡树，除非重新构建一次树结构。

  * 八叉树
    * 基本概念：把空间按照与三个轴平行等分的方式，划分成8个子空间，经过多次划分之后，空间物体都保存在叶节点中（也不一定，后面会说），然后在求ray casting、碰撞检测（粗略检测）、邻近查询（如查询某物体一定范围内物体）等方面都能提高空间搜索效率。
    * 建立方法：首先将场景空间作为最大的根节点，然后遍历空间内物体，按照每个物体包围盒（一般用aabb）相对切分轴（xyz切分轴都是父包围盒坐标的1/2）来确定物体属于哪一个子节点。
    
      我们可以设定一个阈值，比如一个节点内物体数量超过阈值10个，就向下划分子节点，或者说我们对一个场景最多只划分四次就不再向下划分。但是空间物体难免涉及一个问题就是有物体会跨包围盒存在，有两种解决方法：
      
      一个是让所有包围该物体的节点都拥有该物体的空间信息；
      
      一个是破坏只有叶节点允许包含实际空间物体的规则，允许中间节点也包含。

      第一种方式在实际情况下无疑会大大增加计算量，所以一般不考虑。第二种比较常用，但有个问题是，如果物体恰好在场景中心，那么即使很小也会被多个子节点包含，也会损失一部分性能。

      所以事实证明，严格按照空间划分是没有前途的，然后松散八叉树被提出了，即扩大每个子节点的范围，允许子节点之间存在一定重复空间。松散八叉树能一定程度上解决跨包围盒问题，但终究如何使用八叉树还要看实际场景情况，比如使用两三层八叉树，然后继续的细分使用扁平方式存储实际空间物体的包围盒也是很常见的。

      对物体的更新其实只要先看有没有出当前节点包围盒，超出了就相当于该节点先删除物体，然后从根节点重新查找插入点。其中涉及到删除后可能的节点合并以及增添节点的再次划分。

  * KD-Tree：[KD-Tree原理详解 - yachen zhang的文章 - 知乎](https://zhuanlan.zhihu.com/p/112246942)
              
    [kd-tree 原理深入理解 - k近邻（ranged-kNN，NN指Nearest Neighbor）、kd-Tree的插入删除、搜索性能](https://zhuanlan.zhihu.com/p/529487972)

    简单梳理一下原理，其基本规则是，在三维空间下，一个树节点只对一维坐标做切分。我们会先将该节点下物体每一维度的坐标值都求出方差，然后取方差最大的那个维度（其实就是分布最散），比如取 x 轴作为划分轴，接下来切分和BVH类似，我们取所有物体 x 坐标的中位数，然后分割为左右两边（或者上下两边之类的）作为子节点继续切分，这样我们就完成了一个节点的构建。 有的方式只有初始根节点按照方差指定切分轴，然后按照顺序依次切分，但是更多的方法每个节点切分时都求一次方差已达到更合理的空间结构。

    列一下树节点的数据结构
    ```c++
    template <typename PointType>
    struct TreeNode {
      TreeNode* father_;
      TreeNode* left_child_;
      TreeNode* right_child_;
      PointType point_data_;
      int partition_axis_;
      float partition_value_; // not necessary.
    };
    ```
  
  * BVH 见上面
  
* __关于视锥剔除的一些总结__

  [Unity中使用ComputeShader做视锥剔除（View Frustum Culling）](https://zhuanlan.zhihu.com/p/376801370)

  首先我们已经有了上面提到的树形空间结构，以及一个摄像机和他的视锥参数，我们要做的就是遍历树形结构的aabb，然后确认哪些aabb在视锥内，直到具体物体的aabb。

  首先我们的判断标准为：如果一个aabb的8个顶点都在视锥体6个平面的某一个平面之外，那么这个包围盒就是要被剔除的。那么为什么我们不直接看一个aabb是否有一个顶点在视锥体内来判断？原因很简单，考虑下图的D，虽然是二维，但是很显然D的四个顶点都不在2D视锥体内，但他依然和视锥体有交集。如果按照第二种方式，这个包围盒就要被剔除了。
   
  ![image](https://github.com/CHy-KK/Images/blob/main/culling.jpg?raw=true)

  当然这种方式也会让一部分需要剔除的物体进入GPU，但是显然为了避免该显示的显示不了，这点性能损耗是可以接受的。比如下图种田橙色的部分。

  ![image](https://github.com/CHy-KK/Images/blob/main/culling2.png?raw=true)

  具体计算一个点是否在包围盒内其实就是高中数学了，我们知道一个3维平面由平面方程：空间内任意一点(x, y, z)与已知平面上一点(x0, y0, z0)的连线向量(x - x0, y - y0, z - z0) 与已知平面上法线(A, B, C)的点乘=0确立。如果>0 / <0说明该点在法线方向的正方向 / 反方向。原理很简单，其实就是看(x, y, z)到平面上任意一点与平面法线的夹角为正/负，就是我们一直在用的确认各种入射反射方向是否在平面正半球。

  扯远了，所以我们只要给出每个视锥面的指向视锥体内部的法线，以及各个平面上一点，就可以得到六个平面的方程。一般形式即A(x - x0) + B(y - y0) + C(x - x0) = 0，化简为 A'x + B'y + C'z + D' = 0。转换到代码里就是每个平面对应四个参数A' B' C' D'，代入aabb的8个顶点计算正负即可，如果存在一个平面所有点结果都为负那么就被剔除。

  各个平面上的法线也很好求，我们是知道相机坐标以及远近平面共八个顶点坐标的（不知道也能用参数算），这样在各个平面我们都能计算出两个向量，向量方向也是知道的，那叉乘一下就可以得出朝内的法线（注意别搞错叉乘顺序了就行）。视锥平面上一点也很好求，就是camera position以及look at方向上到远近平面上的交点，其中相机坐标可以作为四个边平面上共同的点。

* __走样与锯齿__

  锯齿的来源是场景的定义在空间中是连续的，而最终显示的像素是一个离散的二维数组，所以判断一个点是否被一个像素覆盖的时候单纯是一个01问题，丢失了连续性的信息，导致了锯齿的出现。或者从信号的角度看，我们使用离散的采样，即使频率再大，也无法真正意义上还原一个连续的函数，始终都会造成走样。所以抗锯齿要做的，就是利用各种技术去减轻这种现象。

  从纹理角度来看，锯齿或走样产生的原因有两方面，一种是以低频的像素采样高频的纹理信息造成了纹理走样，也就是下图中minification；另一种是一个texel映射到了多个像素点上，表现为格子边缘的锯齿，也就是下图中的magnification。对于minification，我们熟知的解决办法就是mipmap，对图片进行预模糊或各向异性模糊之后再根据层级采样。
  ![image](https://pic2.zhimg.com/80/v2-6a60e25a1420c59856fbf9a753bf5ab1_720w.webp)

  从几何角度看就是一般我们说的抗锯齿，三角形在光栅化中产生的锯齿问题。解决方法首先是多次采样如SSAA或MSAA；另外使用合适的模糊算法可以将锯齿平滑，例如FXAA；基于历史帧的方式也有TAA。

  从着色角度看，对某些在空间变化较快的部分（如法线、高光等）采样不足也会造成走样，比如flat shading直接应用面法线代替具体的片元法线，就是一种对法线的低频采样。

  * MSAA的具体细节见印象笔记面经，这里不提。
  * FXAA等使用模糊算法的抗锯齿就是基于后处理的方式了，FXAA

* __GPU分支__

[Shader中的 if 和分支 - YAO的文章 - 知乎](https://zhuanlan.zhihu.com/p/122467342)

  在shader中，如hlsl语句，如果使用条件分支语句，例如if，并不一定会产生分支。分支反应在汇编程序上也就是使用jmp、je、jne等跳转指令，跳转到分支语句地址执行指令。jmp为无条件指令，可以只修改IP（指令寄存器）也可以修改IP和CS（代码段寄存器），在进行分支跳转时，CPU会将跳转目标地址存入IP中，下一条要执行的指令将从IP中取出，这样就完成了指令跳转。

  如果GPU产生了分支，由于GPU中线程是以warp（32or64线程）为单位执行指令，所以一旦产生分支，那么将会导致一个warp中的部分线程需要等待另一个分支的线程执行结束之后才能执行指令。这个等待的过程涉及到一些硬件上的操作等，会比两边都执行一遍然后使用movc指令来选择结果效率低。

  当然使用if并不意味着一定会产生分支，编译器一般会默认执行flatten流程，即将分支两侧的逻辑都执行一遍，然后使用movc指令来选择结果。由于避免了分支计算的"等待"，所以效率会高一些。在shader代码中我们可以使用[flatten]或[branch]等流程控制关键字来控制GPU使用的分支方式。
  
  * 静态分支：用uniform来控制的分支，也就是keyword等，可以认为基本不消耗性能
  * 动态分支：需要进行jmp跳转执行分支语句

  在动态分支下，最好的优化方向是保证一个warp内判断变量的相关性最大甚至一致，也就是一个warp内判断结果大部分都是同一分支，只有少数线程处于另一分支。以下图噪声图测试为例，运行的代码逻辑为以噪声值为条件对输出颜色做不同操作。结果是噪声图密度越小（越模糊）效果越好，说明更多的warp保持了一致的分支结果，也就无需进行"等待"操作，节约了性能。(测试结果分别为block 从 1x1 到 32x32，即一格噪声的大小)
  ![image](https://pic4.zhimg.com/v2-2285958b555b349fac48224694a1f357_b.jpg)
  ![image](https://pic2.zhimg.com/v2-6c516b920e708396a647526c74fafae1_b.jpg)
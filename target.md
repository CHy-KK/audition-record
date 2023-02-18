1. URP源代码学习
2. ShdaerLab文档学习
3. (类似上)Shader学习
4. command buffer
5. Houdini??
6. 曲面细分与大规模草地渲染 [百人计划3.3曲面细分与几何着色器](https://www.bilibili.com/video/BV1XX4y1A7Ns?p=2&vd_source=a496344996aebd6de06a773bff299dc1)
7. compute shader(high priority)
8. hdrp(low priority)
9. 动态合批、静态合批、SRP Batch
10. TBR\TBDR[百人计划3.7现代移动端TBR\RBDR](https://www.bilibili.com/video/BV1Bb4y167zU)
11. 

## Demo大赛需求
1. 深度雾等后效
2. 体积雾：先确保管线里面Light下面volumetic开启了，然后看光源下面的volumetic，最后在box volume里面可以单挂一个Fog
3. 直接挂一个空物体当太阳，下面放一个direction Light，sunshaft就用这个坐标当作太阳位置，当然让太阳和月亮出现在视野里也没什么问题
4. 刀光修改：目前看来其实把刀光的后段渐变消失改成淡化，而不是使用x坐标来控制，同时黑色部分同样也淡化，且高光部分时间更短，黑色部分时间更长
5. 径向模糊做一个遮罩，周围模糊
6. 优化：把一些内置的后处理自己实现简化版本
7. 场景天光，记得先烘焙无天光再放天光，不如boss场景会出问题
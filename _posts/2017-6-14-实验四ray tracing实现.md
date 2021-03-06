﻿实验四 ray tracing的实现
郭奚源 201400301235  
  
源代码在仓库CGproject4中~


## 一、完成内容:
1）实现图形对象的光线跟踪渲染算法；  
2）能够根据材质设置，实现不透明和透明物体的渲染；  
3）实现物体阴影； 
 
## 二、算法描述:
物体上的颜色由三部份来组成,分别是物体漫反射出来的光,物体镜面反射出来的光和物体透射的光。  
漫反射的光由各个光原与物体漫反射系数所决定,镜面反射和透射则由根据法线所反射的光线和穿过物体的其他物体光线组成,而射他物体的光线也是一样的方法形成,所以渲染则变成一个递归问题。  
![Markdown](http://i2.muimg.com/1949/42b7baffc1e6e744.png)
而终止条件则是光线反射过一定次数后,过者没有再碰到物体后则停止。  在虚拟场境中设定好视点,视角,屏幕,光源,物体及其反射系数后,就可以从视点朝各像素开始一一跟踪光线,渲染出整个画面。

## 三、代码描述:
在场境中设置了下列的自定义类:  
Point:储存点的坐标或矢量方向;(参看Point.h)  
Light:储存光源坐标,光的颜色和光的强度;(参看Light.h)  
Ray:储存射线的起始点和方向;(参看Ray.h)  
Object:储存场境中物体的位置,反射系数等,此为基类,派生出了两个子类:(参看Object.h)  
Plane:储存了平面的位置和其法矢量;  
Sphere:储存了球体的球心和半径;  
Scene:储存了场境中的所有数据,包括视点,光源,物体等等;(参看Scene.h)  
用OpenGL设置好窗口等的基本函数后开始计算面个像素的颜色:  

在这次试验中 我没有选择使用phong shading,而是由于每个交点和反射折射角度反正都已经求出来，自己实现了一个"ray shading"
(基于自己实现的neareastobject方法判断最近物体)，而这个方法也是ray tracing算法中的重点和主要部分（如何跟踪光线的相交，透射，反射，折射）。下面着重介绍一下这一部分代码。（剩下的代码无非就是设定一些平面和球的位置大小，记录一下光源的位置，光的颜色，光的强度，建立环境等等。具体实现的代码在仓库的文件中，在实验报告里就不详细赘述）

漫反射:  
漫反射光受一个光源反射出来的颜色,其值为光的颜色乘以物体的漫反射系数,又因为漫反射光与法矢量和光线的夹角有关,所以再乘以光的反向点积法线方向,最后再乘以光的强度,得到其漫反射光。  
镜面反射:  
根据入射光线和法矢量求出反射矢量,再以同样的返法求出反射矢量所接收到的光,即进行递归,最后再乘以镜面反射系数,得到其镜平反射光。  
<pre><code>if(depth<maxDepth){  
			if(object->ks!=0){  
				rRay.position=hitPoint;  
				rRay.direction=ray.reflect(object->getNormal(hitPoint));  
				rColor=rayTracing(rRay,depth+1);  
				if(object->type==1){  
					//cout<<"rcolor: "<<rColor<<endl;  
				}  
				rColor+=rColor*object->ks;  
			}  </code></pre>  

透射光:  
当视点射线Ray射到的物体为透明或半透明,即透射系数不等于0时,还要加上其后面透射的光,其计算值与反射光相似,只是方向是原来的返向,将得到的透射光乘以透射系数则得到其透射光;  
<pre><code>if(object->kt!=0){  
			tRay.position=hitPoint;  
			tRay.direction=ray.direction;//未加折射率  
			tColor=rayTracing(tRay,depth+1);  
			}  </code></pre>  
进行归一化:  
正常情况下,上述几种光加起来将会大于1,即得到的结果为白光,所以我将物体的各个反射系数看成矢量求其模,再以此模对以上几种光作加权平均数,才能更接近真实情况下的光照;  
阴影:  
这里求的是漫反射光形成的阴影,当光源到物体之日有其他物体阻碍时,则将原本光原的光乘以阻碍物体的透射系数,当成新的光源射到物体上,形成阴影。  
<pre><code>for(int i=0;i<lights.size();i++){  
		if(object->isNormalSide(lights[i]->position)){  
			sRay=lights[i]->position-hitPoint;  
			sRay.normalize();  
			shadowRay.position=lights[i]->position;  
			shadowRay.direction=hitPoint-lights[i]->position;  
			shadowRay.direction.normalize();  
			int nearIndex=nearestObject(shadowRay);  
			if(objects[nearIndex]==object){  
			sColor+=lights[i]->color*object->kd*sRay.dot(object->getNormal(hitPoint))*lights[i]->Ia;  
			}else{  
			sColor+=lights[i]->color*object->kd*sRay.dot(object->getNormal(hitPoint))*lights[i]->Ia;  
				sColor*=objects[nearIndex]->kt;  
			}  
		}  </code></pre>  

## 四、计算直线与面的相交点:
场境中有平面和球面两种,计算平面与直线交点时,根据公式平面表示方法为  
(P-P0)*N=0;(P为面上的点,P0为面上已知点,N为法矢量)  
直线可表示为  
P=P1+tV;(P为在线的点,P1为射线起点,V为直线方向,t常数)  
联立两方程求出t,再代入直线方程则可求出交点;  
而球面则表示为:  
(P-O)^2=R^2;(P为面上的点,O为球心,R为半径);  
照样联立方和可以求出两个交点,要注意的则是t必定大于0,较小的交点则是先相交的交点。  

## 五、遇到的困难:
写程序的时候按照一步一步来写,先加入平面和漫反射,再来是镜平反射,和球体,最后是阴影等等,而一个较常出现的错误则是有时物体上会出现黑点,经多次调试才发现是浮点数的精度问题,浮点数的尾数会出现一些误差,所以判断点是否在某个面上时,要加大其容错范围才能得到理想效果。  
算法结果:
![Markdown](http://i2.muimg.com/1949/1dfebd3d9c1761ab.png)  

上图为的的算法所渲染出来的结果：  
1.视点前面,上面和下面是只有漫反射的'平面',左右两边是'镜子'；  
2.中间是'镜子球',反射360度的视角（由于上下是绿色的，所以反射到球上，显示出来 球大部分也是绿色的。至于从左边镜子里看到的黄色部分，则是球背后的黄色背景反射出来的效果）；  
3.而右边蓝色的则是一个'半透明的蓝色球'（可以在后面的镜子球里看到蓝色球的反射）
4.设了两个'光源',一个在视点后,所以成了图中较浅的阴影,另一个在球的上方,成了颜色较深的阴影。  

## 六、可改进的地方:
1.目前的屏幕是一个平面,所以左右两点映射出来的物体会出现较大的歪斜,因此可以将屏幕设成一个弧面,让结果更接近人眼。  
2.透明物体遇没有设定折射率,若设定后能够形成类似玻璃球的效果。  
3.透明物体的折射率和反射率与光线的入射角度有关,若增加相关算法则可更加贴更真实。  
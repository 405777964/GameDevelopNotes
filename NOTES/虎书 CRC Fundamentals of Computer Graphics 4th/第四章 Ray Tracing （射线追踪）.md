# 4.1 The Basic Ray-Tracing Algorithm

### 一个基础的射线追踪有三个部分:

1. 射线生成（ray generation）：根据相机几何形状计算每个像素的viewing ray起点和方向
2. 射线相交（ray intersection）：找到与viewing ray相交的对象里最近的那个
3. 着色（shading）：基于相交结果计算像素颜色

## 4.3 Computing Viewing Rays

### 射线与平面相交的公式

![Untitled](NOTES/虎书%20CRC%20Fundamentals%20of%20Computer%20Graphics%204th/第四章%20Ray%20Tracing%20（射线追踪）/Untitled.png)

$$
p(t) = e + t(s - e)
$$

## 4.4.1 Ray-Sphere Intersection

给定一个射线 **p**(t) = e + td 和一个隐式曲面 f(**p**) = 0 ，当射线上的点满足隐式方程就会出现相交点

$$
f(p(t)) = 0   
$$

$$
或者f(e + td)=0
$$

中心点 c = (xc, yc, zc) 半径为R的球的隐式方程表示：

$$
(x − x_c)^2 + (y − y_c)^2 + (z − z_c)2 − R^2 = 0
$$

$$
同等于
(p − c) · (p − c) − R^2 = 0

$$

![Untitled](NOTES/虎书%20CRC%20Fundamentals%20of%20Computer%20Graphics%204th/第四章%20Ray%20Tracing%20（射线追踪）/Untitled%201.png)

最后都是求判别式B^2 − 4AC，大于0有两个交点，等于0有一个交点，小于0则没有交点。

## 4.4.2 Ray-Triangle Intersection

![Untitled](NOTES/虎书%20CRC%20Fundamentals%20of%20Computer%20Graphics%204th/第四章%20Ray%20Tracing%20（射线追踪）/Untitled%202.png)

![Untitled](NOTES/虎书%20CRC%20Fundamentals%20of%20Computer%20Graphics%204th/第四章%20Ray%20Tracing%20（射线追踪）/Untitled%203.png)

射线与三角形的交点

![Untitled](Untitled%204.png)

其中当且仅当 β >0 和 γ > 0 且 β+γ<1时交点在三角形内。

将其向量形式展开：

![Untitled](Untitled%205.png)

可以这么写：

![Untitled](Untitled%206.png)

## 4.4.3 Ray-Polygon Intersection （没弄懂）

多边形有P1到Pn个顶点，表面法线为n，首先计算射线与多边形的交点

$$
(p − p_1) · n = 0.
$$

把 p = e + td 代入进去

![Untitled](Untitled%207.png)

## 4.5 Shading （老生常谈了不写）
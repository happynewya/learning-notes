#### 概率统计

- 实现在以为$R_0$为半径的圆内，以均匀分布随机采样。

  错误做法:red_circle: ❌：

  ```c
  void sampleCircle_wrong(double *x, double *y){
  	double r = R0 * rand(); 	//U[0, R0]
  	double theta = 2 * M_PI * rand();	//U[0, 2*M_PI]
     	*x = r * cos(theta);	
      *y = r * sin(theta);
  }
  ```

  检验是不是均匀分布：

  一个圆内的均匀分布应该满足的概率，假设$x$是横坐标，$y$是纵坐标：
  $$
  P((X,Y))=f(x,y)dxdy=\frac{dxdy}{\pi R_0^2}\tag1
  $$
  转化为极坐标$x=r\cos\theta$, $y=r\sin\theta$得到（注意转化时的雅阁比不等式）：
  $$
  g(r,\theta)=\frac{rdrd\theta}{\pi R_0^2}\tag2
  $$
  如果按照上面的`sampleCircle_wrong`做法，得到的概率显然和上面的式子不一样：
  $$
  g_w(r,\theta)=\frac{dr}{R_0}\frac{d\theta}{2\pi}=\frac{drd\theta}{2\pi R_0}\tag3
  $$
  

  如果需要从$r$和$\theta$直接计算概率，将公式(2)做一个变化：
  $$
  g(r,\theta)=\frac{d(\frac{r^2}{R_0})d\theta}{2\pi R_0}\tag4
  $$
  对比(3)和(4)可以知道，当$\frac{r^2}{R_0}$满足分布$U(0, R_0)$时，可以满足要求。即：
  $$
  \frac{R^2}{R_0}\backsim U(0,R_0)\tag5
  $$
  即：
  $$
  R\backsim R_0\sqrt{U(0, 1)}
  $$
  所以正确做法应该为：

  ```c
  void sampleCircle(double *x, double *y){
  	double r = R0 * sqrt(rand()); 	
  	double theta = 2 * M_PI * rand();	
     	*x = r * cos(theta);	
      *y = r * sin(theta);
  }
  ```

  


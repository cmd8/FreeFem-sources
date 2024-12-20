---
name: Convects
category: PDE
layout: example
---

# Convection using 2 different methods
### - Characteristics-Galerkin methods
### - discontinuous Galerkin methods
A unit bell rotates in a disk, according to
$$
\partial_t c + u\nabla c =0,
\quad
c(x,0) =e^{-\lambda[|x-x_0|^2-|y-y_0|^2]}
$$
where $u=[u_1,u_2]^T$ is the convective velocity assumed to be that of an incompressible fluid (i.e.$\nabla\cdot u=0$) tangent to the boundary of the disk, $u\cdot n=0$ on $\partial\Omega$, where $n$ is the normal to the boundary.
The mesh is generated by
~~~freefem
border C(t=0, 2*pi){x=cos(t); y=sin(t);}
mesh Th = buildmesh(C(100));
~~~
The problem has an analytical solution : $c$ is constant on the curves $t\to X(t)$
$$
\dot X(t)=u(X(t),t), \quad \frac{d c(X(t),t)}{d t}=0
$$
From these it is easy to show that
$$
c(x,t)\approx c(x-u_1(x,t)\delta t,y-u_2(x,t)\delta t,t-\delta t)
$$
In our example $u_1=y,~u_2=-x, x_0=0.3, y_0=0.3$.
~~~freefem
verbosity = 1;
real dt = 0.17;
fespace Uh(Th, P1);
Uh cold, c = exp(-10*((x-0.3)^2 +(y-0.3)^2));
Uh u1 = y, u2 = -x;
~~~
The FreeFem function convect does the following computation
$$
\begin{align*}&
\texttt{convect}([u_1,u_2],-dt,c)
\cr&=c(x-u_1(x,t)\delta t,y-u_2(x,t)\delta t,t-\delta t)
\end{align*}
$$
Consequently, the solver is the time loop
~~~freefem
real t = 0;
for (int m = 0; m < 2*pi/dt; m++) {
	t += dt;
	cold = c;
	c = convect([u1, u2], -dt, cold);
	plot(c, cmm=" t="+t + ", min=" + c[].min + ", max=" +  c[].max);
}
~~~
However the following is much less diffusive, at the cost of a linear solver at each time step
~~~freefem
Uh ch;
t=0;
c = exp(-10*((x-0.3)^2 +(y-0.3)^2));
for (int m = 0; m < 2*pi/dt; m++) {
	t += dt;
	cold = c;
	solve a(c,ch) = int2d(Th)(c*ch)
	      - int2d(Th)(convect([u1, u2], -dt, cold)*ch);
	plot(c, cmm=" t="+t + ", min=" + c[].min + ", max=" +  c[].max);
}
~~~
Finally one may improve the computing time by reusing the matrice (although FreeFem does it behind the scene) but the method is again too diffusive because it has one extra interpolation.
~~~freefem
varf aa(c,ch)= int2d(Th)(c*ch) ;
varf rhs(c, ch) = int2d(Th)(c*ch);

matrix  A = aa(Uh, Uh, verb=1);
matrix B = rhs(Uh, Uh);
c = exp(-10*((x-0.3)^2 +(y-0.3)^2));
for (t = 0; t < 2*pi ; t += dt) {
	cold = c;
	Uh f=convect([u1, u2], -dt, cold);
	ch[] = B* f[];
	c[] = A^-1*ch[];
	plot(c, fill=1, cmm="t="+t + ", min=" + c[].min + ", max=" +  c[].max);
}
~~~
The same problem can be solved with a Discontinuous Galerkin method
~~~freefem
real u, al=0.5;
dt = 0.05;

fespace Vh(Th, P1dc);
Vh w, ccold, v1 = y, v2 = -x, cc = exp(-10*((x-0.3)^2 +(y-0.3)^2));

macro n()(N.x*v1+N.y*v2) //
problem  Adual(cc, w, init=t)
	= int2d(Th)((cc/dt + (v1*dx(cc) + v2*dy(cc)))*w)
	+ intalledges(Th)((1-nTonEdge)*w*(al*abs(n) - n/2)*jump(cc))
//  - int1d(Th, C)((n(u)<0)*abs(n(u))*cc*w)
// unused because cc=0 on d(Omega)^-
	- int2d(Th)(ccold*w/dt);

for (t = 0; t < 2*pi; t += dt) {
	ccold = cc;
	Adual;
	plot(cc, fill=1, cmm="t="+t + ", min="
	+ cc[].min + ", max=" +  cc[].max);
}
~~~
To force the level lines at fixed values one can define viso:
~~~freefem
real [int] viso=[-0.1, 0, 0.05, 0.1, 0.15, 0.2,
0.25, 0.3, 0.35, 0.4, 0.45, 0.5, 0.55, 0.6, 0.65,
0.7, 0.75, 0.8, 0.9, 1];
plot(c,cc, wait=1, value=1, ps="convectDG.ps", viso=viso);
~~~
Another implementation of the DG to speed up computing time:
~~~freefem
varf aadual(cc, w)
	= int2d(Th)((cc/dt + (v1*dx(cc) + v2*dy(cc)))*w)
	+ intalledges(Th)((1-nTonEdge)*w*(al*abs(n) - n/2)*jump(cc));

varf bbdual(ccold, w) = -int2d(Th)(ccold*w/dt);

matrix  AA = aadual(Vh, Vh, verb=1);
matrix BB = bbdual(Vh, Vh);

// Loop
Vh f = 0;
for (t = 0; t < 2*pi ; t += dt) {
	ccold = cc;
	f[] = BB* ccold[];
	cc[] = AA^-1*f[];
	plot(cc, fill=0, cmm="t="+t + ", min=" + cc[].min + ", max=" +  cc[].max);
}

// Plot
plot(cc, wait=1, fill=1, value=1, ps="convectDG.eps", viso=viso);
~~~

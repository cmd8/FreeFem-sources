---
name: adaptsphere
category: mesh
layout: 3d
---

## Construct three 3d meshes of a sphere by varying one parameter in tetgen

The mesh is built by mapping the mesh of a square on a sphere and then call $\texttt{tetg}$.
~~~freefem
load "tetgen"
load "medit"

mesh Th=square(10,20,[x*pi-pi/2,2*y*pi]);  //  $]\frac{-pi}{2},frac{-pi}{2}[\times]0,2\pi[ $
//  a parametrization of a sphere 
func f1 =cos(x)*cos(y);
func f2 =cos(x)*sin(y);
func f3 = sin(x);
//  partiel derivative of the parametrization DF
func f1x=sin(x)*cos(y);   
func f1y=-cos(x)*sin(y);
func f2x=-sin(x)*sin(y);
func f2y=cos(x)*cos(y);
func f3x=cos(x);
func f3y=0;
// $  M = DF^t DF $
func m11=f1x^2+f2x^2+f3x^2;
func m21=f1x*f1y+f2x*f2y+f3x*f3y;
func m22=f1y^2+f2y^2+f3y^2;

func perio=[[4,y],[2,y],[1,x],[3,x]];  
real hh=0.1;
real vv= 1/square(hh);
verbosity=2;
Th=adaptmesh(Th,m11*vv,m21*vv,m22*vv,IsMetric=1,periodic=perio);
Th=adaptmesh(Th,m11*vv,m21*vv,m22*vv,IsMetric=1,periodic=perio);
plot(Th,wait=1);

verbosity=2;

// construction of the surface of spheres
real Rmin  = 1.;
func f1min = Rmin*f1;
func f2min = Rmin*f2;
func f3min = Rmin*f3;

meshS ThS=movemesh23(Th,transfo=[f1min,f2min,f3min]);

real[int] domain = [0.,0.,0.,145,0.01];
mesh3 Th3sph=tetg(ThS,switch="paAAQYY",nbofregions=1,regionlist=domain);

int[int] newlabel = [145,18];
real[int] domainrefine = [0.,0.,0.,145,0.0001];
mesh3 Th3sphrefine=tetgreconstruction(Th3sph,switch="raAQ",region=newlabel,nbofregions=1,regionlist=domainrefine,sizeofvolume=0.0001);

int[int] newlabel2 = [145,53];
func fsize = 0.01/(( 1 + 5*sqrt( (x-0.5)^2+(y-0.5)^2+(z-0.5)^2) )^3);
mesh3 Th3sphrefine2=tetgreconstruction(Th3sph,switch="raAQ",region=newlabel2,sizeofvolume=fsize);
/*
medit("sphere",Th3sph,wait=1);
medit("sphererefinedomain",wait=1,Th3sphrefine);
medit("sphererefinelocal",wait=1,Th3sphrefine2);
*/
plot(Th3sph);
plot(Th3sphrefine);
plot(Th3sphrefine2);
~~~
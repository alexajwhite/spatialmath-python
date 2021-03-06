[![made-with-python](https://img.shields.io/badge/Made%20with-Python-1f425f.svg)](https://www.python.org/)
[![Build Status](https://travis-ci.com/petercorke/spatialmath-python.svg?branch=master)](https://travis-ci.com/petercorke/spatialmath-python)
[![Coverage](https://codecov.io/gh/petercorke/spatialmath-python/branch/master/graph/badge.svg)](https://codecov.io/gh/petercorke/spatialmath-python)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/petercorke/spatialmath-python/graphs/commit-activity)
[![GitHub stars](https://img.shields.io/github/stars/petercorke/spatialmath-python.svg?style=social&label=Star&maxAge=2592000)](https://GitHub.com/petercorke/spatialmath-python/stargazers/)

# Spatial Maths for Python

This is a Python implementation of the [Spatial Math Toolbox for MATLAB<sup>&reg;</sup>](https://github.com/petercorke/spatial-math), which is a standalone component of the [Robotics Toolbox for MATLAB<sup>&reg;</sup>](https://github.com/petercorke/robotics-toolbox-matlab).

Spatial mathematics capability underpins all of robotics and robotic vision where we need to describe the position, orientation or pose of objects in 2D or 3D spaces.


* GitHub repository [https://github.com/petercorke/spatialmath-python](https://github.com/petercorke/spatialmath-python)      
* Documentation [https://petercorke.github.io/spatialmath-python](https://petercorke.github.io/spatialmath-python)
* Dependencies: `numpy`, `scipy`, `matplotlib`, `ffmpeg` (if rendering animations as a movie)


# What it does

Provides a set of classes:

 * `SO2` for orientation in 2-dimensions
 * `SE2` for position and orientation (pose) in 2-dimensions
 * `SO3` for orientation in 3-dimensions
 * `SE3` for position and orientation (pose) in 3-dimensions
 * `UnitQuaternion` for orientation in 3-dimensions

which provide convenience and type safety.  These classes have methods and overloaded operators to support:

 * composition, using the `*` operator
 * exponent, using the `**` operator
 * normalization
 * inversion
 * exponential and logarithm
 * conversion of orientation to/from Euler angles, roll-pitch-yaw angles and angle-axis forms.
 * list operations such as append, insert and get

These are layered over a set of base functions that perform many of the same operations but represent everything in terms of `numpy` arrays.

The class, method and functions names largely mirror those of the MATLAB toolboxes, and the semantics are quite similar.

# Examples


## High-level classes

These classes abstract the low-level numpy arrays into objects that obey the rules associated with the mathematical groups SO(2), SE(2), SO(3), SE(3) as well as twists and quaternions.

Using classes ensures type safety, for example it stops us mixing a 2D homogeneous transformation with a 3D rotation matrix -- both are 3x3 matrices.

For example, to create an object representing a rotation of 0.3 radians about the x-axis is simply

```python
>>> R1 = SO3.Rx(0.3)
>>> R1
   1         0         0          
   0         0.955336 -0.29552    
   0         0.29552   0.955336         
```
while a rotation of 30 deg about the z-axis is

```python
>>> R2 = SO3.Rz(30, 'deg')
>>> R2
   0.866025 -0.5       0          
   0.5       0.866025  0          
   0         0         1    
```
and the composition of these two rotations is 

```python
>>> R = R1 *R2
   0.866025 -0.5       0          
   0.433013  0.75     -0.5        
   0.25      0.433013  0.866025 
```

We can find the corresponding Euler angles

```python
>> R.eul
array([-1.57079633,  0.52359878,  2.0943951 ])
```

Frequently in robotics we want a sequence, a trajectory, of rotation matrices or poses. These pose classes inherit capability from the `list` class

```python
>>> R = SO3()   # the identity
>>> R.append(R1)
>>> R.append(R2)
>>> len(R)
 3
>>> R[1]
   1         0         0       
   0         0.866025 -0.5    
   0         0.5       0.866025            
```
and this can be used in `for` loops and list comprehensions.

An alternative way of constructing this would be

```python
>>> R = SO3( [ SO3(), R1, R2 ] )       
>>> len(R)
 3
```

Many of the constructors such as `.Rx`, `.Ry` and `.Rz` support vectorization

```python
>>> R = SO3.Rx( np.arange(0, 2*np.pi, 0.2))
>>> len(R)
 32
```
which has created, in a single line, a list of rotation matrices.

Vectorization also applies to the operators, for instance

```python
>>> A = R * SO3.Ry(0.5)
>>> len(R)
 32
```
will produce a result where each element is the product of each element of the left-hand side with the right-hand side, ie. `R[i] * SO3.Ry(0.5)`.

Similarly

```python
>>> A = SO3.Ry(0.5) * R 
>>> len(R)
 32
```
will produce a result where each element is the product of the left-hand side with each element of the right-hand side , ie. `SO3.Ry(0.5) * R[i] `.

Finally

```python
>>> A = R * R 
>>> len(R)
 32
```
will produce a result where each element is the product of each element of the left-hand side with each element of the right-hand side , ie. `R[i] * R[i] `.

The underlying representation of these classes is a numpy matrix, but the class ensures that the structure of that matrix is valid for the particular group represented: SO(2), SE(2), SO(3), SE(3).  Any operation that is not valid for the group will return a matrix rather than a pose class, for example

```python
>>> SO3.Rx(0.3)*2
array([[ 2.        ,  0.        ,  0.        ],
       [ 0.        ,  1.91067298, -0.59104041],
       [ 0.        ,  0.59104041,  1.91067298]])

>>> SO3.Rx(0.3)-1
array([[ 0.        , -1.        , -1.        ],
       [-1.        , -0.04466351, -1.29552021],
       [-1.        , -0.70447979, -0.04466351]])
```


## Low-level spatial math

Import the low-level transform functions

```
>>> import spatialmath.base as tr
```

We can create a 3D rotation matrix

```
>>> tr.rotx(0.3)
array([[ 1.        ,  0.        ,  0.        ],
       [ 0.        ,  0.95533649, -0.29552021],
       [ 0.        ,  0.29552021,  0.95533649]])

>>> tr.rotx(30, unit='deg')
array([[ 1.       ,  0.       ,  0.       ],
       [ 0.       ,  0.8660254, -0.5      ],
       [ 0.       ,  0.5      ,  0.8660254]])
```
The results are `numpy` arrays so to perform matrix multiplication you need to use the `@` operator, for example

```
rotx(0.3) @ roty(0.2)
```

We also support multiple ways of passing vector information to functions that require it:

* as separate positional arguments

```
transl2(1, 2)
array([[1., 0., 1.],
       [0., 1., 2.],
       [0., 0., 1.]])
```

* as a list or a tuple

```
transl2( [1,2] )
array([[1., 0., 1.],
       [0., 1., 2.],
       [0., 0., 1.]])

transl2( (1,2) )
Out[444]: 
array([[1., 0., 1.],
       [0., 1., 2.],
       [0., 0., 1.]])
```

* or as a `numpy` array

```
transl2( np.array([1,2]) )
Out[445]: 
array([[1., 0., 1.],
       [0., 1., 2.],
       [0., 0., 1.]])
```

trplot example
packages, animation

There is a single module that deals with quaternions, unit or not, and the representation is a `numpy` array of four elements.  As above, functions can accept the `numpy` array, a list, dict or `numpy` row or column vectors.

```
>>> from spatialmath.base.quaternion import *
>>> q = qqmul([1,2,3,4], [5,6,7,8])
>>> q
array([-60,  12,  30,  24])
>>> qprint(q)
-60.000000 < 12.000000, 30.000000, 24.000000 >
>>> qnorm(q)
72.24956747275377
```

## Symbolic support

Some functions have support for symbolic variables, for example

```
import sympy

theta = sym.symbols('theta')
print(rotx(theta))
[[1 0 0]
 [0 cos(theta) -sin(theta)]
 [0 sin(theta) cos(theta)]]
```

The resulting `numpy` array is an array of symbolic objects not numbers &ndash; the constants are also symbolic objects.  You can read the elements of the matrix

```
a = T[0,0]

a
Out[258]: 1

type(a)
Out[259]: int

a = T[1,1]
a
Out[256]: 
cos(theta)
type(a)
Out[255]: cos
```
We see that the symbolic constants are converted back to Python numeric types on read.

Similarly when we assign an element or slice of the symbolic matrix to a numeric value, they are converted to symbolic constants on the way in.









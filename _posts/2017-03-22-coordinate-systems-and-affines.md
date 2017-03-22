---
layout: post
title:  Coordinate Systems and Affines
date:   2017-03-22 23:16:41 +08:00
categories: MedicalImageProcessing
tags:
- Nibabel
- Affine Matrix
- RAS
- Python

---

* content
{:toc}



## Scanner XYZ

$$scanner\ left-right\ \\ X: \qquad \rightarrow$$

$$scanner\ floor-ceiling\ \\ Y: \qquad \uparrow$$

$$scanner\ bore\ \\ Z: \qquad \odot$$


## Scanner RAS

+ A **subject-centered** scanner coordinate system, where the **scanner axes** are reordered and flipped.
+ RAS : left to **Right**, posterior to **Anterior**, and inferior to **Superior**, respectively.
+ RAS+: Right, Anterior, Posterior are all **positive values** on these axes.

## Affine Matrix

 A **coordinate transform** from voxel coordinates (matrix indices) to scanner RAS+ coordinates.

$$(x, y, z) = f(i, j, k)$$

+ The scanner collects data on a regular grid. This means that the relationship between $$(i,j,k)$$ and $$(x,y,z)$$ is linear (actually affine).
+ Then can be encoded with affine transformations comprising translations, rotations and zooms.


### Scaling (zooming)

in three dimensions can be represented by a diagonal 3x3 matrix. <br>
Here’s how to zoom the first dimension by p, the second by q and the third by r units:

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} =
  \begin{bmatrix}
   pi \\
   qj \\
   rk
  \end{bmatrix} =
  \begin{bmatrix}
   p & 0 & 0 \\
   0 & q & 0 \\
   0 & 0 & r
  \end{bmatrix}
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix}
$$

### Rotation

A rotation by $$\theta$$ radians around the **third** array axis:

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} =
  \begin{bmatrix}
   cos(\theta) & -sin(\theta) & 0 \\
   sin(\theta) &  cos(\theta) & 0 \\
   0        &        0  & 1
  \end{bmatrix}
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix}
$$

A rotation by $$\phi$$ radians around the **second** array axis:

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} =
  \begin{bmatrix}
    cos(\phi)  & 0  &  sin(\phi)  \\
        0   & 1  &       0  \\
   -sin(\phi)  & 0  &  cos(\phi)
  \end{bmatrix}
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix}
$$

A rotation by $$\gamma$$ radians around the **first** array axis:

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} =
  \begin{bmatrix}
    1  &        0  &        0  \\
    0  & cos(\gamma)  & -sin(\gamma) \\
    0  & sin(\gamma)  &  cos(\gamma)
  \end{bmatrix}
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix}
$$

### Translation

A translation of $$a$$ units on the first axis, $$b$$ on the second and $$c$$ on the third might be written as:

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} =
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix} +
  \begin{bmatrix}
   a \\
   b \\
   c
  \end{bmatrix}
$$

### Integration

Write $$f$$ as a combination of matrix multiplication by some 3x3 rotation / zoom matrix $$M$$ followed by addition of a 3x1 translation vector $$(a,b,c)$$

$$
  \begin{bmatrix}
   x \\
   y \\
   z
  \end{bmatrix} = M
  \begin{bmatrix}
   i \\
   j \\
   k
  \end{bmatrix} +
  \begin{bmatrix}
   a \\
   b \\
   c
  \end{bmatrix}
$$

In fact, 4x4 image affine array does include exactly all information:

$$
A =
\begin{bmatrix}
    m_{11}  & m_{12} &  m_{13}  & a\\
    m_{21}  & m_{22} &  m_{23}  & b\\
    m_{31}  & m_{32} &  m_{33}  & c\\
       0  &    0  &     0  & 1
\end{bmatrix}
$$

$$
  \begin{bmatrix}
   x \\
   y \\
   z \\
   1
  \end{bmatrix} =
  \begin{bmatrix}
    m_{11}  & m_{12} &  m_{13}  & a\\
    m_{21}  & m_{22} &  m_{23}  & b\\
    m_{31}  & m_{32} &  m_{33}  & c\\
       0  &    0  &     0  & 1
  \end{bmatrix}
  \begin{bmatrix}
   i \\
   j \\
   k \\
   1
  \end{bmatrix}
$$

$$ \Longrightarrow \qquad
  \begin{bmatrix}
   x \\
   y \\
   z \\
   1
  \end{bmatrix} = A
  \begin{bmatrix}
   i \\
   j \\
   k \\
   1
  \end{bmatrix}
$$

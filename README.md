# MiniQhull

[![CI](https://github.com/gridap/MiniQhull.jl/workflows/CI/badge.svg)](https://github.com/gridap/MiniQhull.jl/actions)
[![Codecov](https://codecov.io/gh/gridap/MiniQhull.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/gridap/MiniQhull.jl)

[MiniQhull](https://github.com/gridap/MiniQhull.jl) ([Qhull](http://www.qhull.org/)-based Delaunay triangulation).

## Documentation

The `MiniQhull` julia package provides a single function `delaunay` overloaded with 3 methods:

```julia
delaunay(dim::Integer, numpoints::Integer, points::AbstractVector) -> Matrix{Int32}
```

Compute the Delaunay triangulation of a cloud of points in an arbitrary dimension `dim`. The length of vector `points` must be `dim*numpoints`. Vector `points` holds data in "component major order", i.e., components are consequitive within the vector. The returned matrix has shape `(dim+1, nsimplices)`, where `nsimplices` is the number of
simplices in the computed Delaunay triangulation.

```julia
delaunay(points::AbstractMatrix) -> Matrix{Int32}
```

In this variant, the cloud of points is specified by a matrix with `size(matrix) == (dim, numpoints)`.

```julia
delaunay(points::AbstractVector) -> Matrix{Int32}
```

In this variant, the cloud of points is specified with a vector of `dim`-element vectors or
tuples. It is also possible to use a vector of other tuple-like types, like `SVector` from 
[StaticArrays.jl](https://github.com/JuliaArrays/StaticArrays.jl).

### Adjusting Qhull flags

You can override the default set of flags that Qhull uses by passing
an additional `flags` argument:

```julia
delaunay(dim::Integer, numpoints::Integer, points::AbstractVector, flags::AbstractString) -> Matrix{Int32}
delaunay(points::AbstractMatrix, flags::AbstractString) -> Matrix{Int32}
delaunay(points::AbstractVector, flags::AbstractString) -> Matrix{Int32}
```

The default set of flags is `qhull d Qt Qbb Qc Qz` for up to 3 dimensions, and `qhull d Qt Qbb Qc Qx` for higher dimensions. The flags you pass override the default flags, i.e. you have to pass all the flags that Qhull should use.

## Examples

```julia
using MiniQhull
dim          = 2
numpoints    = 4
coordinates  = [0,0,0,1,1,0,1,1]
connectivity = delaunay(dim, numpoints, coordinates)
# output
3×2 Array{Int32,2}:
 4  4
 2  3
 1  1
```

```julia
using MiniQhull
coordinates  = [0 0 1 1; 0 1 0 1]
connectivity = delaunay(coordinates)
# output
3×2 Array{Int32,2}:
 4  4
 2  3
 1  1
```

```julia
using MiniQhull, StaticArrays
dim = 5
npts = 500
pts = [SVector{dim, Float64}(rand(dim)) for i = 1:npts];
flags = "qhull d Qbb Qc QJ Pp" # custom flags
connectivity = delaunay(pts, flags)
```

## Installation

`MiniQhull` is a registered Julia package. If your system fulfills all the requirements (see below), `MiniQhull` can be installed using the command:

```
pkg> add MiniQhull
```

If, for any reason, you need to manually build the project, write down the following commands in the Julia REPL:
```
pkg> add MiniQhull
pkg> build MiniQhull
```

### Requirements

The `MiniQhull` package requires the [Qhull](http://www.qhull.org/) reentrant library installed in your system. 

  - By default, if `julia  >= 1.3`, it will use [`Qhull_jll` library](https://github.com/JuliaBinaryWrappers/Qhull_jll.jl) and you don't need to manually install anything.
  - If you need to perform a custom `Qhull` installation. It can be installed in any path on your local machine as long as you export the environment variable `QHULL_ROOT_DIR` pointing to the installation directory. 
  - If none of the previous choices are used, `MiniQhull` will try to find the library in the usual linux user library directory (`/usr/lib`).

`MiniQhull` also requires any C compiler installed on the system.

#### Qhull installation

##### From Sources

Custom installation of `Qhull` can be performed as described in the official [Qhull installation instructions](http://www.qhull.org/README.txt). 
You can find the latest source code in the oficial [Qhull download section](http://www.qhull.org/download/).

##### Debian-based installation from package manager

The reentrant `Qhull` library can be installed with `apt` in recent Debian-based linux distributions as follows:

```
$ sudo apt-get install update
$ sudo apt-get install libqhull-r7 libqhull-dev
```

If you need to install a C compiler, it can be also obtained by means of `apt` tool:
```
$ sudo apt-get update
$ sudo apt-get install gcc build-essential
```

## Continuous integration

In order to use `MiniQhull` in continuous integration jobs, you must ensure that its installation requirements are fullfilled in the CI environment.

For `julia < 1.3` jobs, if your CI process is based on `Travis-CI` you can add the following block at the beginning of your `.travis.yml` file:

```
os:
  - linux
dist:
  - bionic
addons:
  apt:
    update: true
    packages:
    - gcc
    - libqhull-r7
    - libqhull-dev
```


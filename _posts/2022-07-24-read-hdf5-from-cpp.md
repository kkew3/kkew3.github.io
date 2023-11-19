---
layout: post
title:  "Read HDF5 file from C++"
date:   2022-07-24 12:20:10 +0800
tags:   dev--cpp
---

## Introduction

[HDF5](https://portal.hdfgroup.org/display/HDF5/HDF5) is a cross-platform data format used to save (high dimensional) arrays.
There are various language bindings out there for manipulating HDF5 files, including C++.
I record here, after stumbling around many hours, how to read data using C++.

## Read scalars

```cpp
// note the header is not "hdf5.h"
#include "H5Cpp.h"

int main()
{
	H5::H5File file("/path/to/data.h5", H5F_ACC_RDONLY);
	H5::DataSet dataset = file.openDataSet("dataset/path");
	H5::DataSpace filespace = dataset.getSpace();
	// it might be more than sufficient to use `1` here
	hsize_t shape[1];
	// `_dims` must be 0;
	// `shape` shouldn't be touched
	int _dims = filespace.getSimpleExtentDims(shape);
	H5::DataSpace mspace(0, shape);  // where 0 comes from `_dims`
	double buf[1];
	dataset.read(buf, H5::PredType::NATIVE_DOUBLE, mspace, filespace);

	// the scalar is in `buf[0]`

	return 0;
}
```

## Read vector to array

```cpp
#include "H5Cpp.h"

int main()
{
	H5::H5File file("/path/to/data.h5", H5F_ACC_RDONLY);
	H5::DataSet dataset = file.openDataSet("dataset/path");
	H5::DataSpace filespace = dataset.getSpace();
	// `1` corresponds to 1D array (vectors);
	// if reading 2D array (matrices), replace `1` with `2`, so forth
	hsize_t shape[1];
	// `_dims` is the actual N in N-D array; should be the same as
	// previously set; `shape` has now been set
	int _dims = filespace.getSimpleExtentDims(shape);
	H5::DataSpace mspace(1, shape); // replace `1` with `2` if like above
	double *buf = new double[shape[0]];
	// if reading 2D array the previous line should be replaced by:
	//double *buf = new double[shape[0] * shape[1]];
	// so forth
	dataset.read(buf, H5::PredType::NATIVE_DOUBLE, mspace, filespace);

	// the vector (or flatten matrix if reading matrix) is in `buf`

	delete[] buf;
	return 0;
}
```

Note that arrays are stored contiguously.
Read arrays using something like `double buf[M][N]` is not allowed.
See this [answer](https://stackoverflow.com/a/17110562/7881370).

## Read vector to `std::vector`

Basically the same ...

```cpp
#include "H5Cpp.h"
#include <vector>

int main()
{
	H5::H5File file("/path/to/data.h5", H5F_ACC_RDONLY);
	H5::DataSet dataset = file.openDataSet("dataset/path");
	H5::DataSpace filespace = dataset.getSpace();
	// `1` corresponds to 1D array (vectors);
	// if reading 2D array (matrices), replace `1` with `2`, so forth
	hsize_t shape[1];
	// `_dims` is the actual N in N-D array; should be the same as
	// previously set; `shape` has now been set
	int _dims = filespace.getSimpleExtentDims(shape);
	H5::DataSpace mspace(1, shape); // replace `1` with `2` if like above
	// must preserve enough space here
	std::vector<double> buf(shape[0]);
	// likewise, previous line should be written as
	//std::vector<double> buf(shape[0] * shape[1]);
	// if reading 2D array, so forth
	// note the `.data()` here
	dataset.read(buf.data(), H5::PredType::NATIVE_DOUBLE, mspace, filespace);

	// the vector is in `buf`

	return 0;
}
```

## Compile above code

I'm not quite sure how to compile on Windows, but for Linux and macOS, `Makefile` should be written like this.

```make
LDFLAGS = \
	-L/path/to/hdf5/incstall/directory/lib
# note the library names here; only `-lhdf5` is not enough
LDLIBS = \
	-lhdf5 \
	-lhdf5_cpp \
	-lhdf5_hl_cpp
CPPFLAGS = \
	-I/path/to/hdf5/install/directory/include
CXX = clang++

# I haven't tried what if `-std=c++11` is not added, but I guess it
# should be okay
a.out : source.cpp
	$(CXX) $(CPPFLAGS) $(LDFLAGS) -std=c++11 -o $@ $^ $(LDLIBS)
```

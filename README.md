colormap
========

This repository contains a number of somewhat connected utilities which
originated in the development of the [*svm-order-params*][3] project, but are
independent from this application. It's components are split amongs several headers:

* `color.hpp`: Provides a `color::color` class representing a grayscale, RGB, or RGBA
  color, including the ability to mix colors.
* `map.hpp`: Provides the `color::map`, a functor that maps real numbers to a
  color by interpolating between colors at pre-defined support points.
* `palettes.hpp`: Defines a variety of ready-to-use `color::map`s, mostly
  inspired by [ColorBrewer][4] and the [gnuplot-palettes][5] repository by
  *Gnuplotting* (a.k.a. Hagen Wierstorf). The palettes are exposed through a
  global variable `color::palettes`, a `std::map` that maps names
  (`std::string`s) to `color::map`s.
* `map_iterator_adapter.hpp`: Defines an iterator adapter that lazily applies a
  functor on a base iterator upon dereferenciation. Notable API are the
  non-member functions
  - `map_iterator(BaseIterator, Functor&)` which returns a
    `map_iterator_adapter<BaseIterator, Functor>`, and
  - `map(Container const&, Functor&)` which returns a quasi-container object
    `mapped<Container, Functor>`.
* `grid.hpp`: Provides a class `grid` which represents multi-dimensional uniform
  grids which can be initialized very easily and are cheap and iterable.
* `pixmap.hpp`: Provides a class `color::pixmap` which can write iterators over `color`s to disk in PPM (or PGM) format, both in binary, and in ASCII form.


Using colormaps
---------------

The following code snippet showcases some of the features of the headers in this
repository. For a compilable example, see [`test/mandelbrot.cpp`][1].

```cpp
// set up the grid: 701 x 401 grid points (pixels) representing the region
// [-2.5, 1] x [-1, 1].
grid<2, major_order::col> g { {701, {-2.5, 1.}}, {401, {-1., 1.}} };
using grid_point_t = typename decltype(g)::grid_point_type;

// set up the functor to be visualized on the grid, e.g. the mandelbrot set
auto mandelbrot = [] (grid_point_t c_arr) { /* omitted */ };

// Iterator adapter mapping grid points to function values.
// Result is a `mapped` object that behave like a container.
auto val = itadpt::map(g, mandelbrot);

// find the maximum value
double max = *std::max_element(val.begin(), val.end());

// get a colormap and rescale it
auto pal = color::palettes.at("inferno").rescale(1, max);
// and use it to map the values to colors
auto pix = itadpt::map(val, pal);

// Construct a PPM object from the `mapped` object `pix`. Color space is
// inferred from color type of pix. "inferno" is an RGB palette, so `pmap`
// will represent a PPM image. For a grayscale colormap, it would result in
// a PGM image.
color::pixmap<decltype(pix.begin())> pmap(pix.begin(), g.shape());

std::ofstream os("appleman." + pmap.file_extension(),
                 std::ios_base::binary);
pmap.write_binary(os);
```

produces the image `appleman.ppm`:

![appleman][2]

Color Palettes
--------------

![palettes][6]


[1]: test/mandelbrot.cpp
[2]: doc/appleman.png
[3]: https://gitlab.physik.uni-muenchen.de/LDAP_ls-schollwoeck/svm-order-params
[4]: http://colorbrewer2.org/
[5]: https://github.com/Gnuplotting/gnuplot-palettes
[6]: doc/palettes_labelled.png
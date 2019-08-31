# hmm

`hmm` is a <b>h</b>eight<b>m</b>ap <b>m</b>eshing utility.

If you've done any 3D game development, 3D printing, or other such things,
you've likely wanted to convert a grayscale heightmap image into a 3D mesh. The
naive way is pretty simple but generates huge meshes with millions of
triangles. After hacking my way through various solutions over the years, I
finally decided I needed to write a good tool for this purpose.

`hmm` is a modern implementation of a nice algorithm from the 1995 paper
[Fast Polygonal Approximation of Terrains and Height Fields](http://mgarland.org/files/papers/scape.pdf)
by Garland and Heckbert. The meshes produced by `hmm` satisfy the Delaunay
condition and can satisfy a specified maximal error or maximal number of
triangles or vertices. It's also very fast.

![Example](https://i.imgur.com/xLGcmWS.png)

### Dependencies

- C++11 or higher
- [glm](https://glm.g-truc.net/0.9.9/index.html)

### Installation

```bash
brew install glm # on macOS
sudo apt-get install libglm-dev # on Ubuntu / Debian

git clone https://github.com/fogleman/hmm.git
cd hmm
make
make install
```

### Usage

```
usage: hmm --zscale=float [options] ... infile outfile.stl
options:
  -z, --zscale       z scale relative to x & y (float)
  -x, --zexagg       z exaggeration (float [=1])
  -e, --error        maximum triangulation error (float [=0.001])
  -t, --triangles    maximum number of triangles (int [=0])
  -p, --points       maximum number of vertices (int [=0])
  -b, --base         solid base height (float [=0])
  -q, --quiet        suppress console output
  -?, --help         print this message
```

`hmm` supports a variety of file formats like PNG, JPG, etc. for the input
heightmap. The output is always a binary STL file. The only other required
parameter is ZSCALE, which specifies how much to scale the Z axis in the output
mesh.

```bash
$ hmm input.png output.stl -z ZSCALE
```

You can also provide a maximal allowed error, number of triangles, or number of
vertices. (If multiple are specified, the first one reached is used.)

```bash
$ hmm input.png output.stl -z 100 -e 0.001 -t 1000000
```

### Z Scale

The required `-z` parameter defines the distance between a fully black pixel and a fully white pixel in the vertical Z axis, with units equal to one pixel width or height. For example, if each pixel in the heightmap represented a 1x1 meter square area, and the vertical range of the heightmap was 100 meters, then `-z 100` should be used.

### Z Exaggeration

The `-x` parameter is simply an extra multiplier on top of the provided Z scale. It is provided as a convenience so you don't have to do multiplication in your head just to exaggerate by, e.g. 2x, since Z scales are often derived from real world data and can have strange values like 142.2378.

### Max Error

The `-e` parameter defines the maximum allowed error in the output mesh, as a percentage of the total mesh height. For example, if `-e 0.01` is used, then no pixel will have an error of more than 1% of the distance between a fully black pixel and a fully white pixel. This means that for an 8-bit input image, an error of `e = 1 / 256 ~= 0.0039` will ensure that no pixel has an error greater than one full grayscale unit. (It may still be desireable to use a lower value like `0.5 / 256`.)

### Base Height

When the `-b` option is used to create a solid mesh, it defines the height of the base before the lowest part of the heightmesh appears, as a percentage of the heightmap's height. For example, if `-z 100 -b 50` were used, then the final mesh would be about 150 units tall (if a fully white pixel exists in the input).

### Performance

Performance depends a lot on the amount of detail in the heightmap, but here are some figures for an example heightmap of a [40x40 kilometer area centered on Mount Everest](https://i.imgur.com/1i9djJ0.jpg). Various heightmap resolutions and permitted max errors are shown. Times computed on a 2018 13" MacBook Pro (2.7 GHz Intel Core i7).

#### Runtime in Seconds

| Image Size / Error | e=0.01 | e=0.001 | e=0.0005 | e=0.0001 |
| ---: | ---: | ---: | ---: | ---: |
| 9490 x 9490 px (90.0 MP) | 6.535 | 13.102 | 19.394 | 58.949 |
| 4745 x 4745 px (22.5 MP) | 1.867 |  4.903 |  8.886 | 33.327 |
| 2373 x 2373 px  (5.6 MP) | 0.559 |  2.353 |  4.930 | 14.243 |
| 1187 x 1187 px  (1.4 MP) | 0.168 |  1.021 |  1.961 |  3.709 |

#### Number of Triangles Output

| Image Size / Error | e=0.01 | e=0.001 | e=0.0005 | e=0.0001 |
| ---: | ---: | ---: | ---: | ---: |
| 9490 x 9490 px (90.0 MP) | 33,869 | 1,084,972 | 2,467,831 | 14,488,022 |
| 4745 x 4745 px (22.5 MP) | 33,148 | 1,032,263 | 2,323,772 | 11,719,491 |
| 2373 x 2373 px  (5.6 MP) | 31,724 |   935,787 | 1,979,227 |  6,561,070 |
| 1187 x 1187 px  (1.4 MP) | 27,275 |   629,352 | 1,160,079 |  2,347,713 |

### TODO

- pre-triangulation filters? e.g. gaussian blur
- export a normal map?
- automatically compute some z scale?
- better error handling, especially for file I/O
- better overflow handling - what's the largest supported heightmap?
- OpenCL rasterization?
- mesh validation?
- reconstruct grayscale image?
- explicit lithophane support?

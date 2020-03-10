# fi_c44 for Linux

* `c44` - is one of several DJVU encoders from opensource [DjVuLibre](http://djvu.sourceforge.net) project
* `fi_c44` - is a fork of c44 made by [monday2000](http://djvu-soft.narod.ru/soft/fi_c44.htm). "fi_" stands for [FreeImage](http://freeimage.sourceforge.net) - the library that allows to perform operations with images in most popular formats. Original `c44` could read only images in PNM/PBM/PGM or JPEG. With help of FreeImage `fi_c44` can read images in PNG or TIFF, etc. `fi_44` is used in [DjVu Imager](http://djvu-soft.narod.ru/scan/djvu_imager.htm) tool written by monday2000.

Unfortunately, although `c44` was crossplatform, the `fi_44` was designed for Windows only. Some windows-specific code was used in it and it lacks of built tools.

I've decided to adapt it a little to make more crossplatform:

* The Windows specific code was replaced.
* Some project files were added to make it a standalone CMake project.
* The original `c44.cpp` file was updated with an up-to-date copy to be able to compile with latest DjVuLibre.

Now encoder could be build for Linux.
I'm going to port whole DjVu Imager tool to Qt to make it native for Linux.

# Compilation (Linux)
1. Clone the project.
2. Install the dependancies: `sudo apt-get install libfreeimage-dev`
3. Get the latest DjVuLibre sources and unpack them to the root folder of the project.
4. If the subfolder with DjVuLibre has name other then `djvulibre-3.5.27` - open CMakeFIles.txt and correct the path for `djvu_libre_sources_path` variable in it.
5. ```mkdir build
      cd build
      cmake ...
      make```
5. To test - run `./fi_c44 ` from the `build` subfolder




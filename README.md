# c44-fi - is a crossplatform

* `c44` - is one of several DJVU encoders from opensource [DjVuLibre](http://djvu.sourceforge.net) project
* `fi_c44` - is a fork of c44 made by [monday2000](http://djvu-soft.narod.ru/soft/fi_c44.htm). "fi_" stands for [FreeImage](http://freeimage.sourceforge.net) - the library that allows to perform operations with images in most popular formats. Original `c44` could read only images in PNM/PBM/PGM or JPEG. With help of FreeImage `fi_c44` can read images in PNG or TIFF, etc. `fi_44` is used in [DjVu Imager](http://djvu-soft.narod.ru/scan/djvu_imager.htm) tool written by monday2000.
* `c44-fi` - is a renamed version of `fi_c44`, adapted for Linux.

Unfortunately, although `c44` was crossplatform, the `fi_44` was designed for Windows only. Some windows-specific code was used in it and it lacks of build tools.

I've decided to adapt it a little to make more crossplatform:

* The Windows specific code was replaced.
* Some project files were added to make it a standalone CMake project.
* The original `c44.cpp` file was updated with an up-to-date copy to be able to compile with latest DjVuLibre.
* The project is renamed to avoid "_" character usage in package names. Also I hope it'll be easy to remember.

Now encoder could be build for Linux.
I've also ported [DjVu Imager](https://github.com/trufanov-nok/DjVu_Imager-Qt) tool to Qt to build it natively for Linux. `DjVu Imager Qt` is using `c44-fi`.

# Build
1. Clone the project with djvulibre as a submodule (note `--recursive` parameter):  
```
      git clone --recursive https://github.com/trufanov-nok/c44-freeimage.git
      cd c44-freeimage
```
2. Install the dependencies: `sudo apt-get install libfreeimage-dev`
3. Build the project:
```
      mkdir build
      cd build
      cmake ..
      make
```
4. To test - run `./c44-fi ` from the `build` subfolder

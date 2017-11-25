## [linux] [R](https://mirrors.tuna.tsinghua.edu.cn/CRAN/)
<2017.1016>
refer to: https://pastebin.com/2V8PwY2z
### download R
```
wget https://mirrors.tuna.tsinghua.edu.cn/CRAN/src/base/R-3/R-3.4.2.tar.gz
tar -xzvf R-3.4.2.tar.gz
cd R-3.4.2
./configure --prefix=/home/wuxiaolong/tools/R-3.4.2
``` 
1. WARNING: cannot run mixed C/Fortran code
   configure: error: Maybe check LDFLAGS for paths to Fortran libraries?
> `gfortran --version` # 4.8.2
> `find / -name "libgfor*.so"` # mix 4.4.4 and 4.8.2
> `LDFLAGS="-L/home/wuxiaolong/tools/gcc-4.8.2/lib64 -L/home/wuxiaolong/tools/gcc-4.8.2/lib -lgfortran" ./configure --prefix=/home/wuxiaolong/tools/R-3.4.2`

2. checking if zlib version >= 1.2.5... no
checking whether zlib support suffices... configure: error: zlib library and headers are required
> `wget http://prdownloads.sourceforge.net/libpng/zlib-1.2.11.tar.gz?download`
> `tar -xzvf zlib-1.2.11.tar.gz`
> `cd zlib-1.2.11`
> `./configure --prefix=$HOME/tools/zlib-1.2.11`
> `make && make install`


3. checking if bzip2 version >= 1.0.6... no
checking whether bzip2 support suffices... configure: error: bzip2 library and headers are required
> `wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz`
> `tar bzip2-1.0.6.tar.gz`
> `cd bzip2-1.0.6`
> `make -f Makefile-libbz2_so`
> `make clean`
> `make`
> `make install --prefix=$HOME/tools/bzip2-1.0.6`
> add `export PATH=~/tools/bzip-1.0.6/bin:$PATH` to ~/.bashrc

4. configure: error: "liblzma library and headers are required"
> `wget https://tukaani.org/xz/xz-5.2.3.tar.gz`
> `tar xz-5.2.3.tar.gz`
> `cd xz-5.2.3`
> `./configue --prefix=$HOME/tools/xz-5.2.3`
> `make -j3`
> `make install`

5. checking if PCRE version >= 8.20, < 10.0 and has UTF-8 support... no
checking whether PCRE support suffices... configure: error: pcre >= 8.20 library and headers are required
> `wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz`
> `tar -xzvf pcre-8.41.tar.gz`
> `cd pcre-8.41`
> `./configure --prefix=$HOME/tools/pre-8.41`
> `make -j4`
> `make install`

6. curl-7.55.1
error:libcurl>=7.22.0 library and headers are required with support for https
make sure your curl is not the one under anaconda/miniconda

7. final:

```
LDFLAGS="-L/$HOME/tools/gcc-4.8.2/lib64 -L/$HONME/tools/gcc-4.8.2/lib -lgfortran \
-L/$HOME/tools/zlib-1.1.11/lib \
-L/$HOME/tools/bzip2-1.0.6/lib \
-L/$HOME/tools/xz-5.2.3/lib \
-L/$HOME/tools/pcre-8.41/lib \
-L/$HOME/tools/curl-7.55.1/lib \
-L/$HOME/tools/libiconv-1.15/lib" \
CPPFLAGS="-I/$HOME/tools/zlib-1.2.11/include \
-I/$HOME/tools/bzip2-1.0.6/include \
-I/$HOME/tools/xz-5.2.3/include \
-I/$HOME/tools/pcre-8.41/include \
-I/$HOME/tools/curl-7.55.1/include \
-I/$HOME/tools/libiconv-1.15/include" \
./configure --prefix=/home/wuxiaolong/tools/R-3.4.2 --with-readline=yes --with-x=yes --enable-R_shlib`
```
warning info:
- configure: WARNING: you cannot build info or HTML versions of the R manuals
- configure: WARNING: you cannot build PDF versions of the R manuals
- configure: WARNING: you cannot build PDF versions of vignettes and help pages

8. make
1> /usr/bin/ld: Dwarf Error: found dwarf version '4', this reader only handles version 2 information.
`update binutils-2.29.1`
2> ../usr/bin/ld: Dwarf Error: found dwarf version '4', this reader only handles version 2 information.
`change gcc to gcc -fPIC` in bzip2's Makefile
3> /usr/bin/ld: warning: libpcre.so.1, needed by ../../lib/libR.so, not found (try using -rpath or -rpath-link)
/usr/bin/ld: warning: liblzma.so.5, needed by ../../lib/libR.so, not found (try using -rpath or -rpath-link)
add `export LD_LIBRARY_PATH=~/tools/pcre-8.41/lib:~/tools/xz-5.2.3/lib:$LD_LIBRARY_PATH` to ~/.bashrc
3> ../../lib/libR.so: undefined reference to `libiconv'
../../lib/libR.so: undefined reference to `libiconv_close'
../../lib/libR.so: undefined reference to `_libiconv_version'
../../lib/libR.so: undefined reference to `libiconv_open'
install libiconv-1.15.tar.gz
```
wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz
tar -xzvf libiconv-1.15.tar.gz
cd libiconv-1.15
./configure --prefix=/$HOME/tools/libiconv-1.15
make && make install
```
9. make install
1> /usr/bin/install: cannot stat 'NEWS.pdf': No such file or directory
/usr/bin/install: cannot stat 'NEWS.pdf': No such file or directory
```
cd ~/software/R-3.4.2/doc
vim Makefile
去掉L18、L19行的NEWS.pdf
```
10. 配置
add `export PATH=~/tools/R-3.4.2/bin:$PATH` to ~/.bashrc
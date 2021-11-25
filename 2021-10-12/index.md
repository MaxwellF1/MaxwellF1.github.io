# Spack学习


在强师兄的推荐下了解到了Spack这款包管理的工具，好像在HPC领域用的很多。查阅了一些资料，在自己使用的过程中也遇到了一些问题。为了加深自己的认识，以及防止狗熊掰棒子的行为，在这里记录一下自己的理解以及遇到的问题。

## Spack

## 问题

1. Q: 自己想用spack安装新的软件，但显然自己只是个小用户，是没有sudo权限的。

   A: 师兄给出的使用说明中，说可以自己安装spack后再将全局的spack配置为自己spack的上游。spack由于依赖的东西非常少，安装起来只需要根据官方文档的说明就可以完成。关于配置为上游的过程，在本地目录中我一直没有找到相关的配置文件。百度和Google上也没找到相关问题，后来在[官方文档](https://spack.readthedocs.io/en/latest)的**Chaining Spack Installations**部分找到了相关解答，需要自己在\$SPACK_ROOT\$/etc/spack/目录下新建文件`upstreams.yaml`，在填写上配置信息保存即可，花括号中的替换为对应想要设置为上游的spack路径。

   ```yaml
   upstreams:
     spack-instance-1:
       install_tree: {/path/to/other/spack}/opt/spack
     spack-instance-2:
       install_tree: {/path/to/another/spack}/opt/spack
   ```

   

2. Q: Load不上Compilers

   A: 根据使用说明，spack find查看可用包后想load上gcc@7.5.0来用，之后紧接着load cmake@xxx，但却提示我

   ```bash
   Error: No compilers for operating system centos 7 satisfy spec gcc\@7.5.0
   ```

   然后根据官方手册`spack compilers`查看可用编译器，发现没有load上gcc@7.5.0。使用`spack compiler find`之后也没也看到gcc-7.5.0在可用列表内。手册上说可以使用`spack compiler find /path/`来直接添加，但我并不知道gcc-7.5.0的安装位置。手册上也给出了这种情况的解决办法：

   ```bash
   spack load gcc@7.5.0
   spack compiler find
   ==> Added 1 new compiler to /home/fengguofeng/.spack/linux/compilers.yaml
       gcc@7.5.0
   ==> Compilers are defined in the following files:
       /home/fengguofeng/.spack/linux/compilers.yaml
   ```

   再检查时可以看到已经load上了gcc@7.5.0，也没有后续报错，问题暂时解决。

   另外，可以通过`spack find --paths`列出所有路径，也可以对于gcc，通过`spack find -p gcc`来查看指定包具体的路径。

   

3. Q: `spack install`时external packages相关报错，例如：

   ```bas
   Error: InstallError: cray-fftw is not installable, you need to specify it as an external package in packages.yaml
   ```

   A:  有时候会出现，有时候不出现，即使是对于info中相关的external信息为False的情况。有点奇怪。比如可以装netlib-scalapack但没法install fftw。

4. 怎么用spack+module啊，安装lmod提示失败。

## References

1. [Spack — Spack 0.16.3 documentation](https://spack.readthedocs.io/en/latest/)
2. [Spack - NERSC Documentation](https://docs.nersc.gov/development/build-tools/spack/)
3. [Spack 入门指南 – refraction-ray (re-ra.xyz)](https://re-ra.xyz/Spack-入门指南/)
4. [Installing HPCToolkit with Spack](http://hpctoolkit.org/software-instructions.html)



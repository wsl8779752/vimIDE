###### 第一步：源码安装vim 
a. 安装依赖库  
sudo apt install libncurses5-dev libgnome2-dev libgnomeui-dev \\\
libgtk2.0-dev libatk1.0-dev libbonoboui2-dev \\\
libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev \\\
python3-dev ruby-dev lua5.1 liblua5.1-dev libperl-dev git libgtk-3-dev

b. Remove vim if you have it already \
sudo apt remove vim vim-runtime gvim

c. Once everything is installed, getting the source is easy. \
cd vim_source \
./configure --with-features=huge \\\
--enable-multibyte \\\
 --enable-rubyinterp=yes \\\ 
--enable-pythoninterp=yes \\\
 --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu  \\\
--enable-perlinterp=yes \\\
 --enable-luainterp=yes \\\
--enable-gui=gtk2 --enable-cscope --prefix=/usr/local \\\
     (注意ubuntu下只能在python2.7 和3.5之间选一个，另外python-config-dir 变量值要改成自己机器上想匹配的)

make VIMRUNTIMEDIR=/usr/local/share/vim/vim81 \\\
sudo make install

###### 第二步：各种插件

a.将配置文件夹下的所有内容拷贝到 ~目录下 \
cp -r 配置文件 ~

b.需要安装的一些插件依赖程序或者库 \
sudo apt-get install ctags \
sudo apt-get install ack-grep


c.然后执行:PluginInstall即可\


###### 第三步:安装llvm 和clang（最蛋疼的一步，花了我两天时间）\

关于第三步骤，有两种方法
方法一：\
```
cd clang+llvm-6.0.0-x86_64-linux-gnu-ubuntu-16.04/
sudo cp -R * /usr/

```
玩 C/C++ 你肯定要用到标准库。概念上，GCC 配套的标准库涉及 libstdc++ 和 libsupc++ 两个子库，前者是接口层（即，上层的封装），后者是实
  
```
sudo apt install libstdc++-5-dev
```
位于 /usr/include/c++/5/，接着安装链接库
  
```
sudo apt-get install libstdc++6
```
位于 /usr/lib/libstdc++.so.6。呃，是滴，libstdc++、libsupc++ 两个子库的相关文件是合并一起安装的。\

方法二：\
下载 LLVM、clang 及辅助库源码： 
```
cd ~/downloads 
# Checkout LLVM
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm 
# Checkout Clang
cd llvm/tools 
svn co http://llvm.org/svn/llvm-project/cfe/trunk clang 
cd ../.. 
# Checkout extra Clang Tools
cd llvm/tools/clang/tools 
svn co http://llvm.org/svn/llvm-project/clang-tools-extra/trunk extra 
cd ../../../.. 
# Checkout Compiler-RT
cd llvm/projects 
svn co http://llvm.org/svn/llvm-project/compiler-rt/trunk compiler-rt 
cd ../..
```
关掉其他应用，尽量多的系统资源留给 GCC 编译 clang 源码：

```
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release ../llvm
make && make install
```
接下来，你先洗个澡，再约个会，回来应该编译好了（thinkpad T410I，CPU 奔腾双核 P6000，MEM 4G DDRIII，耗时 2 小时）。确认下：

```
clang --version 
```

玩 C/C++ 你肯定要用到标准库。概念上，GCC 配套的标准库涉及 libstdc++ 和 libsupc++ 两个子库，前者是接口层（即，上层的封装），后者是实现层（即，底层的具体实现），对应实物文件，你得需要两个子库的头文件和动态链接库（\*.so）。openSUSE 的安装源中有，直接安装头文件

```
sudo apt install libstdc++-5-dev
```
位于 /usr/include/c++/5/，接着安装链接库

```
sudo apt-get install libstdc++6
```
位于 /usr/lib/libstdc++.so.6。呃，是滴，libstdc++、libsupc++ 两个子库的相关文件是合并一起安装的。

对应到 clang 的标准库，libc++（接口层）和 libc++abi（实现层）也需要安装头文件和动态链接库（\*.so）。这里并不知道sudo apt-get install libc++可不可以，所以采用比较原始的方法


```
cd ~/Downloads/ 
svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx 
mkdir libcxx_build
cd libcxx_build 
cmake ~/Downloads/libcxx
make
```
头文件已经生成到(我觉得是svn下来的代码自带的） ~/Downloads/libcxx/include/，要让 clang 找到必须复制到 /usr/include/c++/v1/

```
cp -r ~/Downloads/libcxx/include/ /usr/include/c++/v1/
```

*.so 文件已生成 ~/Downloads/libcxx_build/lib/libc++.so.1.0，要让 clang 访问必须复制到 /usr/lib/，并创建软链接

```
cp ~/Downloads/libcxx_build/lib/libc++.so* /usr/lib/
```
类似，源码安装 libc++abi 的头文件和动态链接库：

```
cd ~/Downloads/ 
svn co http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi 
mkdir libcxxabi_build
cd libcxxabi_build
cmake ~/Downloads/libcxxabi
make
```
头文件已经生成到(系统自带） ~/Downloads/libcxxabi/include/，要让 clang 找到必须复制到 /usr/include/c++/v1/

```
cp -r ~/Downloads/libcxxabi_build/include/ /usr/include/c++/v1/
\*.so 文件已生成 ~/downloads/libcxx/lib/libc++abi.so.1.0，要让 clang 访问必须复制到 /usr/lib/，并创建软链接
sudo cp ~/Downloads/libcxxabi_build/lib/libc++abi.so* /usr/lib/
```
至此完成clang-llvm 的安装和配置，个人觉得还是非常麻烦的。我将所需要的源码保存在deb里，下次可以省去点下载时间。直接解压到Downloads

接着，执行 ctags 生成标准库的标签文件：
###### 第四步执行安装后配置
```
cd /usr/include/c++/5
ctags -R --c++-kinds=+l+x+p --fields=+iaSl --extra=+q --language-force=c++ -f stdcpp.tags

```
添加支持修改MARKDOWN 插件的文本

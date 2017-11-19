# YCM-on-arch
This are the instructions to install you complete me on arch linux

Download Clang / LLVM Arch Linux binaries from : http://releases.llvm.org/download.html. 

Say it is downloaded to ~/Downloads,

>cd ~/Downloads

>tar xf clang+llvm-4.0.0-aarch64-linux-gnu.tar.xz

>sudo mkdir /usr/share/ycm-clang-llvm

>sudo mv clang+llvm-4.0.0-aarch64-linux-gnu/* /usr/share/ycm-clang-llvm/

Now you need to find the CMakeLists.txt file for ycm_core. Go to the folder where your vim / neovim package manager downloaded YouCompleteMe. In my case, it is :

>cd ~/.config/nvim/bundle/YouCompleteMe/

The cmake file is located in

>/path/to/YouCompleteMe/third_party/ycmd/cpp/ycm/CMakeLists.txt

You will notice the following lines at the top of this file:

Change OFF to ON on lines 23 and 24 ( USE_CLANG_COMPLETER and USE_SYSTEM_LIBCLANG ).

Then on line 25, for 'PATH_TO_LLVM_ROOT' insert the path to where you coppied the clang / llvm binaries. In my case the line reads
set( PATH_TO_LLVM_ROOT "/usr/share/ycm-clang-llvm" CACHE PATH "Path to ... " )

And finally, set the path to libclang on line 26. In my case it reads
set( EXTERNAL_LIBCLANG_PATH "/usr/share/ycm-clang-llvm/lib/libclang.so" CACHE PATH "Path to ... " )

Finally, we can build and install YouCompleteMe by running ./install.py --clang-completer in the neovim / vim plugin directory, where it was downloaded.

Enjoy the YCM !!

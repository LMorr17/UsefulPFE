# UsefulPFE


## Setup your build environment

**Note:**

This document presents the inital setup to create an independant linux system within your machine with a version that supprots Rust integration.
Steps form 1 to 8 are made in the `~/src/linx` path while the rest in the `~/src/busybox` path.

---

1. Create your personal workspace folder: (Choose an appropriate path to set up the environment) <br>
`$ mkdir src` <br>
`$ cd src`

2. Clone the "Rust for linux" repo and make sure it's the "rust" branch <br>
`$ git clone -b rust --depth=1 https://github.com/Rust-for-Linux/linux.git`

3. Clone the busybox repo into your workspace folder: <br>
`$ git clone --depth=1 https://github.com/mirror/busybox`

After the previous operations you must have 2 git reposotories in the `src` folder: **linux** and **busybox**.

4. Go to the linux folder and execute the makefile: <br>
`$ cd linux` <br>
`$ make -j12` <br>

The makefile will fail in the beginning since there are some packages missing. To execute the makefile, you have :<br>
`$ make allnoconfig qemu-busybox-min.config`

Some errors that you fill find are that: flex and bison are missing, so you can install them from apt, and the makefile can be executed once all the packages are found.

5. You should build the kernel using LLVM: <br>
`$ make LLVM=1 allnoconfig qemu-busybox-min.config`

Again, some packages will be missing: such as clang and ld.lld
Make sure to sure to install version 11 of clang and ld.lld since it's the minimum version required to build the kernel. The packages on apt are "clang-11" and "lld-11"
To install the packages, run the command: <br>
`sudo apt install clang-11 lld-11`

If you already have clang and ld.lld installed from apt, don't remove them. Simply install clang-11 and lld-11 and use the update-alternatives command. <br>

`sudo update-alternatives --install /bin/clang clang /bin/clang-11 100` <br>
`sudo update-alternatives --install /bin/ld.lld ld.lld /bin/ld.lld-11 100` <br>

After installing the compiler and the linker, run the make command again to check if everything's set correctfully. <br>
`make LLVM=1 -j12`

6. Now, we need to setup the rust config in the kernel: <br>
`$ make LLVM=1 allnoconfig qemu-busybox-min.config rust.config`

But first, we have to setup Rust in our machine: <br>
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

If you don't have curl installed, install it with apt not with snap (`sudo apt install curl`)

Make sure to run this command after every install: <br>
`$ make LLVM=1 rustavailable`

After installing Rust, restart your terminal and run `rustc --version` to make sure it's properly installed

From the same folder (linux) re-update rustc to the version required with this command: <br>
`$ rustup override set $(scripts/min-tool-version.sh rustc)`

Now install bindgen with this command: <br>
`cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen`

Install the Rust standard library source
`$ rustup component add rust-src`

Now if you run: `$ make LLVM=1 rustavailable`, you should see that Rust is available

7. Rerun:<br>
`$ make LLVM=1 allnoconfig qemu-busybox-min.config rust.config`

8. Install necessary packages: <br>
```
$ sudo apt install libelf-dev
$ make LLVM=1 -j12
```

If you find issues with LLVM during the last command, you should update the links: <br>

```
$ sudo update-alternatives --install /bin/llvm-ar llvm-ar /bin/llvm-ar-11 100 
$ sudo update-alternatives --install /bin/llvm-nm llvm-nm /bin/llvm-nm-11 100 
$ sudo update-alternatives --install /bin/llvm-objcopy llvm-objcopy /bin/llvm-objcopy-11 100 
$ make LLVM=1 -j`nproc` 
```

This will build your first custom kernel

9. Setup busybox

```
~/src/busybox$ make defconfig
~/src/busybox$ make menuconfig
```
(You should see the menu: Go to settings and tick static building)
```
$ make -j`nproc`
$ make install
$ cd _install/
$ find . | cpio -H newc -o > ../ramdisk.img
```
10. `sudo apt install qemu-kvm`

11. Now, keep watching the video from 36:40 to setup qemu with your custom kernel and your custom busybox (mentioned bellow 5)

## Documentation
1. https://rust-for-linux.github.io/docs/kernel/index.html (Rust kernel modules documentation)
2. https://docs.kernel.org/ (kernel documentation)
3. https://elixir.bootlin.com/linux/latest/source (bootlin kernel source code with multiple versions)
4. https://docs.kernel.org/rust/coding-guidelines.html (Rust support in kernel documentation)
	       
	       
	       
5. Enviorement Setup: https://www.youtube.com/watch?v=tPs1uRqOnlk (video - PART 1)
6. Writing Linux Kernel in Rust: https://www.youtube.com/watch?v=-l-8WrGHEGI&list=PLbzoR-pLrL6o8cdq_JLTwsLfe2_DhNsDf&index=15 (video - PART 2)
7. Linux Kernel Map: https://makelinux.github.io/kernel/map/
8. Zulip channel for Rust-For-Linux: https://rust-for-linux.zulipchat.com/
9. Rust for Linux official page: https://rust-for-linux.com/
10. Presentation about network device driver: https://ufal.mff.cuni.cz/~jernej/2018/docs/predavanja15.pdf

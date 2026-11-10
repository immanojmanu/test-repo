shielded nodes: disabled
confidential nodes: disabled
OS: Ubuntu_Containerd

>>>> To determine whether your Ubuntu 24.04 GKE node (or underlying OS) supports 32-bit x86 (i386) binaries at Runtime Execution and Compile Time, you need to check kernel configurations, architecture runtime support, and build environments. >>>>> 
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uptime
 16:15:39 up 31 min,  1 user,  load average: 0.03, 0.04, 0.14
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ cat /etc/*release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.5 LTS"
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uname -r
6.8.0-1064-gke
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uname -m
x86_64
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ 



testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ zgrep CONFIG_IA32_EMULATION /proc/config.gz || grep CONFIG_IA32_EMULATION /boot/config-$(uname -r)
gzip: /proc/config.gz: No such file or directory
CONFIG_IA32_EMULATION=y
# CONFIG_IA32_EMULATION_DEFAULT_DISABLED is not set

> indicates the kernel allows 32-bit syscall execution.


* Check Kernel Boot Parameters & Sysctl Flags
Check if IA32 emulation is disabled via sysctl (0 = enabled, 1 = disabled)
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sysctl sys.kernel.ia32_emulation
sysctl: cannot stat /proc/sys/sys/kernel/ia32_emulation: No such file or directory
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-6.8.0-1064-gke root=PARTUUID=f1dad15e-aa31-48fc-abe7-74d6fd1b579a ro console=ttyS0,115200 panic=-1

> Starting with Linux 6.7+ (Ubuntu 24.04 defaults to Linux 6.8+), Linux supports disabling 32-bit emulation dynamically or via boot flags:


* Check 32-bit Userspace / Architecture Support

testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sudo dpkg --print-foreign-architectures
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ $?
-bash: 0: command not found

$ sudo ls -l /lib/ld-linux.so.2
ls: cannot access '/lib/ld-linux.so.2': No such file or directory



* Check Compile-Time Support (Can it COMPILE 32-bit binaries?)
gcc -m32 -v 2>&1 | grep -i "multilib"

testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sudo gcc -m32 -v 2>&1 | grep -i "multilib"
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ 


GCC is not there so installed:
sudo apt install build-essential -y

gcc -m32 -v 2>&1 | grep -i "multilib"

testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ gcc -m32 -v 2>&1 | grep -i "multilib"
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 13.3.0-6ubuntu2~24.04.1' --with-bugurl=file:///usr/share/doc/gcc-13/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-13 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/libexec --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-libstdcxx-backtrace --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-gcn/usr --enable-offload-defaulted --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=2


# vim test.c
#include <stdio.h>

int main() {
    printf("Pointer size: %zu bits\n", sizeof(void*) * 8);
    return 0;
}

Compile and run with the -m32 flag:

Compile as 32-bit binary
$ gcc -m32 test.c -o test32

testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ gcc -m32 test.c -o test32
In file included from test.c:1:
/usr/include/stdio.h:28:10: fatal error: bits/libc-header-start.h: No such file or directory
   28 | #include <bits/libc-header-start.h>
      |          ^~~~~~~~~~~~~~~~~~~~~~~~~~
compilation terminated.

$ sudo apt install gcc-multilib g++-multilib libc6-dev-i386
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sudo dpkg --add-architecture i386
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sudo apt update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Get:5 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/main i386 Packages [1041 kB]
Hit:6 http://ppa.launchpad.net/cloud-images/gke-1.35/ubuntu noble InRelease           
Get:7 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/universe i386 Packages [8514 kB]
Get:8 http://security.ubuntu.com/ubuntu noble-security/main i386 Packages [386 kB]
Get:9 http://ppa.launchpad.net/cloud-images/gke-1.35/ubuntu noble/main i386 Packages [1088 B]
Get:10 http://security.ubuntu.com/ubuntu noble-security/universe i386 Packages [841 kB]
Get:11 http://security.ubuntu.com/ubuntu noble-security/restricted i386 Packages [29.8 kB]
Get:12 http://security.ubuntu.com/ubuntu noble-security/multiverse i386 Packages [12.2 kB]   
Get:13 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/restricted i386 Packages [14.7 kB]
Get:14 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/multiverse i386 Packages [126 kB]
Get:15 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main i386 Packages [575 kB]
Get:16 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/universe i386 Packages [1153 kB]
Get:17 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/restricted i386 Packages [32.2 kB]
Get:18 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/multiverse i386 Packages [14.0 kB]
Get:19 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-backports/main i386 Packages [33.1 kB]
Get:20 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-backports/universe i386 Packages [17.1 kB]
Get:21 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-backports/multiverse i386 Packages [748 B]
Fetched 12.8 MB in 2s (5739 kB/s)               
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
72 packages can be upgraded. Run 'apt list --upgradable' to see them.
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ sudo apt install gcc-multilib g++-multilib libc6-dev-i386
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  g++-13-multilib gcc-13-multilib lib32asan8 lib32atomic1 lib32gcc-13-dev lib32gcc-s1 lib32gomp1 lib32itm1 lib32quadmath0 lib32stdc++-13-dev lib32stdc++6 lib32ubsan1 libc6-dev-x32
  libc6-i386 libc6-x32 libx32asan8 libx32atomic1 libx32gcc-13-dev libx32gcc-s1 libx32gomp1 libx32itm1 libx32quadmath0 libx32stdc++-13-dev libx32stdc++6 libx32ubsan1
Suggested packages:
  lib32stdc++6-13-dbg libx32stdc++6-13-dbg
The following NEW packages will be installed:
  g++-13-multilib g++-multilib gcc-13-multilib gcc-multilib lib32asan8 lib32atomic1 lib32gcc-13-dev lib32gcc-s1 lib32gomp1 lib32itm1 lib32quadmath0 lib32stdc++-13-dev lib32stdc++6
  lib32ubsan1 libc6-dev-i386 libc6-dev-x32 libc6-i386 libc6-x32 libx32asan8 libx32atomic1 libx32gcc-13-dev libx32gcc-s1 libx32gomp1 libx32itm1 libx32quadmath0 libx32stdc++-13-dev
  libx32stdc++6 libx32ubsan1
0 upgraded, 28 newly installed, 0 to remove and 72 not upgraded.
Need to get 26.2 MB of archives.
After this operation, 105 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libc6-i386 amd64 2.39-0ubuntu8.9 [2787 kB]
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libc6-dev-i386 amd64 2.39-0ubuntu8.9 [1447 kB]
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libc6-x32 amd64 2.39-0ubuntu8.9 [2917 kB]
Get:4 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libc6-dev-x32 amd64 2.39-0ubuntu8.9 [1637 kB]
Get:5 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32gcc-s1 amd64 14.2.0-4ubuntu2~24.04.1 [92.3 kB]
Get:6 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32gcc-s1 amd64 14.2.0-4ubuntu2~24.04.1 [78.5 kB]
Get:7 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32gomp1 amd64 14.2.0-4ubuntu2~24.04.1 [141 kB]
Get:8 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32gomp1 amd64 14.2.0-4ubuntu2~24.04.1 [144 kB]
Get:9 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32itm1 amd64 14.2.0-4ubuntu2~24.04.1 [29.5 kB]
Get:10 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32itm1 amd64 14.2.0-4ubuntu2~24.04.1 [29.8 kB]
Get:11 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32atomic1 amd64 14.2.0-4ubuntu2~24.04.1 [8594 B]
Get:12 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32atomic1 amd64 14.2.0-4ubuntu2~24.04.1 [10.3 kB]
Get:13 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32asan8 amd64 14.2.0-4ubuntu2~24.04.1 [2880 kB]
Get:14 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32asan8 amd64 14.2.0-4ubuntu2~24.04.1 [2893 kB]
Get:15 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32stdc++6 amd64 14.2.0-4ubuntu2~24.04.1 [814 kB]
Get:16 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32ubsan1 amd64 14.2.0-4ubuntu2~24.04.1 [1150 kB]
Get:17 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32stdc++6 amd64 14.2.0-4ubuntu2~24.04.1 [778 kB]
Get:18 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32ubsan1 amd64 14.2.0-4ubuntu2~24.04.1 [1169 kB]
Get:19 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32quadmath0 amd64 14.2.0-4ubuntu2~24.04.1 [227 kB]
Get:20 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32quadmath0 amd64 14.2.0-4ubuntu2~24.04.1 [157 kB]
Get:21 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32gcc-13-dev amd64 13.3.0-6ubuntu2~24.04.1 [2380 kB]
Get:22 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32gcc-13-dev amd64 13.3.0-6ubuntu2~24.04.1 [2190 kB]
Get:23 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 gcc-13-multilib amd64 13.3.0-6ubuntu2~24.04.1 [880 B]
Get:24 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 lib32stdc++-13-dev amd64 13.3.0-6ubuntu2~24.04.1 [1150 kB]
Get:25 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 libx32stdc++-13-dev amd64 13.3.0-6ubuntu2~24.04.1 [1086 kB]                                            
Get:26 http://us-central1.gce.archive.ubuntu.com/ubuntu noble-updates/main amd64 g++-13-multilib amd64 13.3.0-6ubuntu2~24.04.1 [892 B]                                                  
Get:27 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/main amd64 gcc-multilib amd64 4:13.2.0-7ubuntu1 [1474 B]                                                                  
Get:28 http://us-central1.gce.archive.ubuntu.com/ubuntu noble/main amd64 g++-multilib amd64 4:13.2.0-7ubuntu1 [884 B]                                                                   
Fetched 26.2 MB in 12s (2206 kB/s)                                                                                                                                                      
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libc6-i386.
(Reading database ... 70888 files and directories currently installed.)
Preparing to unpack .../00-libc6-i386_2.39-0ubuntu8.9_amd64.deb ...
Unpacking libc6-i386 (2.39-0ubuntu8.9) ...
Selecting previously unselected package libc6-dev-i386.
Preparing to unpack .../01-libc6-dev-i386_2.39-0ubuntu8.9_amd64.deb ...
Unpacking libc6-dev-i386 (2.39-0ubuntu8.9) ...
Selecting previously unselected package libc6-x32.
Preparing to unpack .../02-libc6-x32_2.39-0ubuntu8.9_amd64.deb ...
Unpacking libc6-x32 (2.39-0ubuntu8.9) ...
Selecting previously unselected package libc6-dev-x32.
Preparing to unpack .../03-libc6-dev-x32_2.39-0ubuntu8.9_amd64.deb ...
Unpacking libc6-dev-x32 (2.39-0ubuntu8.9) ...
Selecting previously unselected package lib32gcc-s1.
Preparing to unpack .../04-lib32gcc-s1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32gcc-s1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32gcc-s1.
Preparing to unpack .../05-libx32gcc-s1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32gcc-s1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32gomp1.
Preparing to unpack .../06-lib32gomp1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32gomp1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32gomp1.
Preparing to unpack .../07-libx32gomp1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32gomp1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32itm1.
Preparing to unpack .../08-lib32itm1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32itm1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32itm1.
Preparing to unpack .../09-libx32itm1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32itm1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32atomic1.
Preparing to unpack .../10-lib32atomic1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32atomic1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32atomic1.
Preparing to unpack .../11-libx32atomic1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32atomic1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32asan8.
Preparing to unpack .../12-lib32asan8_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32asan8 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32asan8.
Preparing to unpack .../13-libx32asan8_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32asan8 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32stdc++6.
Preparing to unpack .../14-lib32stdc++6_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32stdc++6 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32ubsan1.
Preparing to unpack .../15-lib32ubsan1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32ubsan1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32stdc++6.
Preparing to unpack .../16-libx32stdc++6_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32stdc++6 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32ubsan1.
Preparing to unpack .../17-libx32ubsan1_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32ubsan1 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32quadmath0.
Preparing to unpack .../18-lib32quadmath0_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32quadmath0 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package libx32quadmath0.
Preparing to unpack .../19-libx32quadmath0_14.2.0-4ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32quadmath0 (14.2.0-4ubuntu2~24.04.1) ...
Selecting previously unselected package lib32gcc-13-dev.
Preparing to unpack .../20-lib32gcc-13-dev_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32gcc-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package libx32gcc-13-dev.
Preparing to unpack .../21-libx32gcc-13-dev_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32gcc-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package gcc-13-multilib.
Preparing to unpack .../22-gcc-13-multilib_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking gcc-13-multilib (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package lib32stdc++-13-dev.
Preparing to unpack .../23-lib32stdc++-13-dev_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking lib32stdc++-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package libx32stdc++-13-dev.
Preparing to unpack .../24-libx32stdc++-13-dev_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking libx32stdc++-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package g++-13-multilib.
Preparing to unpack .../25-g++-13-multilib_13.3.0-6ubuntu2~24.04.1_amd64.deb ...
Unpacking g++-13-multilib (13.3.0-6ubuntu2~24.04.1) ...
Selecting previously unselected package gcc-multilib.
Preparing to unpack .../26-gcc-multilib_4%3a13.2.0-7ubuntu1_amd64.deb ...
Unpacking gcc-multilib (4:13.2.0-7ubuntu1) ...
Selecting previously unselected package g++-multilib.
Preparing to unpack .../27-g++-multilib_4%3a13.2.0-7ubuntu1_amd64.deb ...
Unpacking g++-multilib (4:13.2.0-7ubuntu1) ...
Setting up libc6-x32 (2.39-0ubuntu8.9) ...
Setting up libx32gomp1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libc6-i386 (2.39-0ubuntu8.9) ...
Setting up libx32quadmath0 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32atomic1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32atomic1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libc6-dev-i386 (2.39-0ubuntu8.9) ...
Setting up lib32itm1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32gcc-s1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32itm1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32asan8 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libc6-dev-x32 (2.39-0ubuntu8.9) ...
Setting up lib32gomp1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32gcc-s1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32stdc++6 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32asan8 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32quadmath0 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32stdc++6 (14.2.0-4ubuntu2~24.04.1) ...
Setting up libx32ubsan1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32ubsan1 (14.2.0-4ubuntu2~24.04.1) ...
Setting up lib32gcc-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Setting up libx32gcc-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Setting up gcc-13-multilib (13.3.0-6ubuntu2~24.04.1) ...
Setting up lib32stdc++-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Setting up gcc-multilib (4:13.2.0-7ubuntu1) ...
Setting up libx32stdc++-13-dev (13.3.0-6ubuntu2~24.04.1) ...
Setting up g++-13-multilib (13.3.0-6ubuntu2~24.04.1) ...
Setting up g++-multilib (4:13.2.0-7ubuntu1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.9) ...
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ $?
-bash: 0: command not found
















_________________________________________________________________________________________________________________________________________________________________


### Final Conclusion:
shielded nodes: disabled
confidential nodes: disabled
OS: Ubuntu_Containerd
GKE:  v1.35.8-gke.1975000


testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uptime
 16:15:39 up 31 min,  1 user,  load average: 0.03, 0.04, 0.14
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ cat /etc/*release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.5 LTS"
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uname -r
6.8.0-1064-gke
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$ uname -m
x86_64
# sudo dmesg
Based on the attached dmesg output from your GKE node, the logs confirm a clean and healthy kernel initialization that fully supports 32-bit execution.

Here is what the log reveals regarding 32-bit support:

	•	Clean Kernel Boot: The node boots using the standard GKE kernel (6.8.0-1064-gke) built on Ubuntu's x86_64 toolchain. 
	•	No Restrictive Boot Parameters: The kernel command line (Command line: BOOT_IMAGE=/vmlinuz-6.8.0-1064-gke root=... ro console=ttyS0,115200 panic=-1) contains no restrictions or overrides such as ia32_emulation=0. 
	•	Zero 32-Bit Errors: There are no warnings, faults, or rejection messages concerning ia32, emulation, or 32-bit system calls during the boot sequence or system startup. 
Conclusion
Because CONFIG_IA32_EMULATION=y is compiled into this kernel and the dmesg logs show no boot-time blocks or warnings, your GKE node kernel is fully active and ready to execute 32-bit binaries without any kernel-level interference. Combined with the libc6-i386 packages you installed via apt, your runtime and compile-time environments are completely set up for i386 support.

Top 3 Validations: ( On ubuntu2404 Node of the GKE)

1) Verifies that 32-bit system call emulation is enabled in the running kernel:
$ grep CONFIG_IA32_EMULATION /boot/config-$(uname -r)
CONFIG_IA32_EMULATION=y

> Confimrs Kernel Support (Runtime Foundation)
What it means: The running GKE kernel (6.8.0-1064-gke) has 32-bit system call emulation compiled directly into it. This allows the 64-bit CPU to process instructions and intercept system calls from 32-bit software.

2) Verifies that the required 32-bit dynamic libraries and runtime linkers are installed on the OS:
testing-1@gke-testing-gke-32bin-ub-np001-709d2037-qndz:~$  dpkg -l | grep -E "libc6-i386|lib32gcc"
ii  lib32gcc-13-dev                    13.3.0-6ubuntu2~24.04.1           amd64        GCC support library (32 bit development files)
ii  lib32gcc-s1                        14.2.0-4ubuntu2~24.04.1           amd64        GCC support library (32 bit Version)
ii  libc6-i386                         2.39-0ubuntu8.9                   amd64        GNU C Library: 32-bit shared libraries for AMD64

> Runtime Support (i386 Libraries & Linkers): Packages like libc6-i386 and lib32gcc-s1 are installed (ii).
The necessary 32-bit shared C libraries and dynamic linkers are present on the OS. This allows 32-bit ELF executables to successfully resolve their dependencies and execute at runtime without throwing missing library errors



3) Compile-Time Support
Verifies that GCC is configured with multilib support to compile 32-bit code (-m32):
$ gcc -m32 -v 2>&1 | grep -i "multilib"
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 13.3.0-6ubuntu2~24.04.1' --with-bugurl=file:///usr/share/doc/gcc-13/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-13 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/libexec --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-libstdcxx-backtrace --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-gcn/usr --enable-offload-defaulted --without-cuda-driver --enable-checking=release --build=x
86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-c
onfig=bootstrap-lto-lean --enable-link-serialization=2

> Compile-Time Support (Multilib Toolchain): GCC is configured with --enable-multilib and --with-multilib-list=m32,m64,mx32
Your compiler toolchain is fully equipped to build 32-bit code. You can use the -m32 flag with GCC to compile standard C/C++ source code into 32-bit binaries (i386).


% Confirmation of Your Conclusion:
Yes, you can say this with 100% confidence.
Based on all the validations you performed on your Ubuntu 24.04 GKE node:
	1	Kernel Runtime Support (CONFIG_IA32_EMULATION=y) proves the 64-bit kernel can handle 32-bit system calls.
	2	Library Support (libc6-i386, lib32gcc-s1) proves 32-bit apps can find the dynamic linkers and shared libraries they need to execute.
	3	Compile-Time Support (gcc-multilib) proves you can build and target i386 binaries using -m32.
As long as your Kubernetes pods contain the necessary 32-bit runtime libraries (either by installing them in your container image or utilizing a base image that includes them), your GKE nodes and pods can successfully compile and execute 32-bit binaries.

Also, dmesg output yielded no issues or reporters errors related to 32bit.

# sudo dmesg
Based on the attached dmesg output from your GKE node, the logs confirm a clean and healthy kernel initialization that fully supports 32-bit execution.

Here is what the log reveals regarding 32-bit support:

	•	Clean Kernel Boot: The node boots using the standard GKE kernel (6.8.0-1064-gke) built on Ubuntu's x86_64 toolchain. 
	•	No Restrictive Boot Parameters: The kernel command line (Command line: BOOT_IMAGE=/vmlinuz-6.8.0-1064-gke root=... ro console=ttyS0,115200 panic=-1) contains no restrictions or overrides such as ia32_emulation=0. 
	•	Zero 32-Bit Errors: There are no warnings, faults, or rejection messages concerning ia32, emulation, or 32-bit system calls during the boot sequence or system startup. 
Conclusion
Because CONFIG_IA32_EMULATION=y is compiled into this kernel and the dmesg logs show no boot-time blocks or warnings, your GKE node kernel is fully active and ready to execute 32-bit binaries without any kernel-level interference. Combined with the libc6-i386 packages you installed via apt, your runtime and compile-time environments are completely set up for i386 support.


_________________________________________________________________________________________________________________________________________________________________

Performing the same on brand-new cluster again:

shielded nodes: disabled
confidential nodes: disabled
OS: Ubuntu_Containerd
GKE:  v1.35.8-gke.1975000

testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ uptime
 17:57:51 up 10 min,  1 user,  load average: 0.05, 0.45, 0.37
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ cat /etc/*release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.5 LTS"
PRETTY_NAME="Ubuntu 24.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.5 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ uname -r
6.8.0-1064-gke
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ uname -m
x86_64


1) testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ grep CONFIG_IA32_EMULATION /boot/config-$(uname -r)
CONFIG_IA32_EMULATION=y
# CONFIG_IA32_EMULATION_DEFAULT_DISABLED is not set

2)  dpkg -l | grep -E "libc6-i386|lib32gcc"

dpkg not found
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ type gcc
-bash: type: gcc: not found
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ type dpkg
dpkg is /usr/bin/dpkg


testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ sudo  dpkg -l | grep -E "libc6-i386|lib32gcc"
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ $?
-bash: 1: command not found



3) ~$ sudo  gcc -32 -v | grep -i "multilib"
sudo: gcc: command not found

Installing gcc : 
apt-get update -y;  
sudo apt install gcc-multilib g++-multilib libc6-dev-i386

After installing gcc:
$ sudo  gcc -m32 -v | grep -i "multilib"
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-linux-gnu/13/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 13.3.0-6ubuntu2~24.04.1' --with-bugurl=file:///usr/share/doc/gcc-13/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,
objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-13 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/libexec --
without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-defaul
t-libstdcxx-abi=new --enable-libstdcxx-backtrace --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=r
elease --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enabl
e-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-13-EldibY/gcc-13-13.3.0/debian/tmp-gc
n/usr --enable-offload-defaulted --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap
-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 13.3.0 (Ubuntu 13.3.0-6ubuntu2~24.04.1) 



Now executed the step 2 which failed now after doing  "apt-get-update and the above packages"

$ sudo dpkg -l | grep -E "libc6-i386|lib32gcc"
ii  lib32gcc-13-dev                    13.3.0-6ubuntu2~24.04.1           amd64        GCC support library (32 bit development files)
ii  lib32gcc-s1                        14.2.0-4ubuntu2~24.04.1           amd64        GCC support library (32 bit Version)
ii  libc6-i386                         2.39-0ubuntu8.9                   amd64        GNU C Library: 32-bit shared libraries for AMD64


<< Testing the whole scenario by creating a CProgram, compiling and exectuing it >>
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("--- 32-bit Execution Test ---\n");
    printf("Pointer size: %zu bits (Expected: 32)\n", sizeof(void*) * 8);
    printf("Process ID: %d\n", getpid());
    printf("SUCCESS: 32-bit binary is running properly!\n");
    return 0;
}

2) Compile it as a 32-bit Binary
Compile the file using the -m32 flag to force 32-bit code generation:
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ gcc -m32 test32.c -o test32_bin
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ ls -ltr
total 20
-rw-rw-r-- 1 testing-1 testing-1   293 Nov 10 18:40 test32.c
-rwxrwxr-x 1 testing-1 testing-1 14972 Nov 10 18:41 test32_bin

3) Verify the Binary Type
Run the file command to inspect the executable format and confirm it built as an ELF 32-bit binary:
$ file test32_bin
testing-1@gke-testing-gke-002-np002-ub2404-cont-c0e37ff6-d4c6:~$ file test32_bin
test32_bin: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=a763ef1c6656b02fdb6dd09d2e4253b71c532146, for GNU/Linux 3.2.0, not stripped

4) Running the binary
$ ./test32_bin
--- 32-bit Execution Test ---
Pointer size: 32 bits (Expected: 32)
Process ID: 11241
SUCCESS: 32-bit binary is running properly!


The test output completely confirms everything end-to-end.

* **The Binary Type Verification** (`file test32_bin`) confirms it successfully built as an **`ELF 32-bit LSB pie executable, Intel 80386`** using the dynamic interpreter `/lib/ld-linux.so.2`.


* **The Execution Output** (`./test32_bin`) confirms that the GKE node kernel handled the 32-bit system calls successfully, resulting in the correct pointer size of **32 bits** and a successful execution run.

Your transition strategy to Ubuntu 24.04 nodes with confidential nodes disabled is fully verified and ready for production deployment with your pre-built 32-bit container images.

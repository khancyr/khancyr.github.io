---
title: "Speeding up compilation"
date:
  created: 2021-07-07
  updated: 2021-07-07
draft: false
categories: 
    - ArduPilot
    - ROS
tags:
    - ROS

authors:
  - khancyr
---

# Speeding up compilation time

![compiling|413x360](https://discuss.ardupilot.org/uploads/default/original/3X/6/f/6f534090abf7e3170318de96efc2b7b029a806a4.png)
*https://xkcd.com/303/*

Hello friends,

Some time ago, I left my job for new adventures. Doing so, I lost my company recent laptop and then my development workflow get really downgraded.
I am now using an old 2011 laptop with i5-2410M (2 cores at 2.3Ghz) and 4Go RAM DDR3. That is still decent but when you taste the speed you cannot get back ...

Well, my issue is the compile time for reviewing PR on ArduPilot.

# Standard setup

With the default installation a full build with `waf` takes **7m14s**.

Yep, you should use just `waf` or `waf copter` for example instead of `waf -j4`. The `-j` stand for `--jobs` and is the number of compilation jobs you want to use. The more you use the more CPU cores and computation power for compilation you will use. Generally, you scale the number of jobs with the number of threads your computer support. On my laptop, I got a 2 cores CPU with 4 threads. That means that I could do 4 compilation jobs in parallel !
On contrary of `make` that need the number of jobs explicitly passed, waf is already taking care of maximizing the number of jobs on your machine.

Fortunately, like make and other builds system, waf is smart enough to not do full rebuild each time we made some change. But this generally won't work when we switch the branch on git or, obviously, do a `waf clean`.

# Best standard setup

If you followed our installation instructions correctly, you should have seen that when you are using `waf configure` the output looks like :

```` python
Checking for 'g++' (C++ compiler) : /usr/lib/ccache/g++
Checking for 'gcc' (C compiler) : /usr/lib/ccache/gcc
````

instead of
```` python
Checking for 'g++' (C++ compiler) : /usr/bin/g++
Checking for 'gcc' (C compiler) : /usr/bin/gcc
````

What does it mean ? On the second case, waf is detecting GCC and G++ as the C and C++ compiler, that is the intended setup. On the first and rightful case, waf is detecting ccache as the compiler. Ccache is a compiler cache. It will put in cache previous compilation file to reuse them !

You can use `ccache -s` to get a summary of you cache usage. In my case :
```` python
cache directory /home/khancyr/.ccache
primary config /home/khancyr/.ccache/ccache.conf
secondary config  (readonly) /etc/ccache.conf
stats updated Wed Feb 19 13:31:20 2020
stats zeroed Mon Nov 11 19:20:57 2019
cache hit (direct) 3513
cache hit (preprocessed) 117
cache miss 12804
cache hit rate 22.09 %
called for link 483
called for preprocessing 78
compile failed 33
cleanups performed 208
files in cache 117
cache size 1.4 MB
max cache size 5.0 GB
````
It reuses 20% of the cache instead of compiling, and that is pretty interesting to speed up your builds !

After a small change on ArduPilot file, using `waf` but this time with ccache, I get a build time of **17.9s**. Well, mostly everything is in cache, so I don't have to recompile everything !

Sadly, that won't work in all case, and plenty of time in need the full and long build.
Then, how to speed up compilation ?

# Using another computer to speed up compilation

I got a gaming desktop computer with a 4 core i5 and 16Go DDR3 RAM. That is a also an old computer but that is a beast comparing to the laptop !
As I got Windows, games and most of my important file (picture, papers, etc.) on it, I don't want to risk myself to dual boot it to have a Linux on it. The simpler way was to instead a virtual machine on it. I used VirtualBox and setup a Ubuntu VM.

On Linux, it exists a utility called `distcc` that allow to distribute the compilation tasks across multiple computer ! That what I am going to use.

## Setup
On Ubuntu the installation is simple :
`sudo apt install distcc gcc g++ ccache`
Obviously, you need a compiler to make it works, and I also install ccache as it will serve.
You can use `systemctl enable distccd` to auto start it on your machine.

Now you get distcc on your VM waiting for compilation order. You need then to show waf how to use it.
I have create a new file called `distcc_config` with content :
```` bash
export CCACHE_PREFIX="distcc"
export CC="ccache gcc"
export CXX="ccache g++"
export DISTCC_HOSTS='localhost/4 10.42.0.79/5,lzo'
export DISTCC_JOBS=$(distcc -j)
echo "Building with $DISTCC_JOBS parallel jobs on following servers:"
for server in `distcc --show-hosts`; do
server=$(echo $server | sed 's/:.*//')
echo -e "\t$server"
done
````

Here is what it does :

- `CCACHE_PREFIX` allow us to use distcc on our computer in combination with ccache. I would be a same to not have it.
- `export CC` and `export CXX` explicitly set the compilers for distcc.
- `DISTCC_HOSTS` need, unfortunately to be set manually. It said to distcc what computer use and the number of jobs they can handle. In my case, `localhost/4` is for my laptop : 4 jobs. `10.42.0.79/8,lzo` for my desktop computer : 8 jobs and `lzo` to compress file to send.

Now you can invoke `waf -j $(distcc -j)` to ask waf to compile with distcc max number of jobs, in my case 12 jobs.
![distcc2|634x500](https://discuss.ardupilot.org/uploads/default/original/3X/4/2/42faf50f45dc680d26f84fb8458bb7a5fa10b453.png)

The result is a full compilation in **4m10s** with the drawback to use a lot my network, but as I am on a gigabyte Ethernet network with nobody watching 4k video, that isn't an issue for me !

My solution was to use VirtualBox, but if you are more used to Docker, you could just use a lightly machine with distcc only for the same purpose.

Edit: Funny things, I give another try on WSL and success make it work. It was even better that with VirtualBox as it achieved compilation in **3m36.658s**

### Limits

Limits to distcc usage :
- you need the same version of the compiler on each computer.
- it will use a lot your network to transfers file to compile and compilation results.
- using your dusted RPIs won't bring much help against using a decent CPU.


# Conclusion
I hope you learn something on how to speed up compilation on ArduPilot, unless you really need it, I won't recommend you to use distcc as the setup and maintenance can be tricky. The default ArduPilot environment with Waf and Ccache should be enough in most case to get the maximum performance out of your machine.

You can still help us on the project to make consecutive compilation faster by helping us to clean the inclusion dependency on ArduPilot. With clean inclusion, waf will be able, on its own, to recompile only the part that need to instead of compiling back everything.
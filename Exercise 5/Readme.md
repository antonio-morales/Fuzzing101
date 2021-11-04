# Exercise 5 - LibXML2

For this exercize we will fuzz **LibXML2** XML parsing library. The goal is to find a crash/PoC for [**CVE-2017-9048**](https://nvd.nist.gov/vuln/detail/CVE-2017-9048) in LibXML2 2.9.4. 

<details>
  <summary>For more information about CVE-2017-9048 vulnerability, click me!</summary>
  --------------------------------------------------------------------------------------------------------
  
  **CVE-2017-9048** is an stack buffer overflow vulnerability affecting the DTD validation functionality of LibXML2.
  
  A stack buffer overflow is a type of buffer overflow where the buffer being overwritten is allocated on the stack.
  
 As a result, a remote attacker can exploit this issue to execute arbitrary code within the context of an application using the affected library.

 You can find more information about stack buffer oveflow vulnerabilities at the following link: https://cwe.mitre.org/data/definitions/121.html
  
</details>

## What you will learn
Once you complete this exercise you will know how: 
- To use custom dictionaries for helping the fuzzer to find new execution paths
- To parallelize the fuzzing job accross multiple cores

## Read Before Start
- I suggest you to try to **solve the exercise by yourself** without checking the solution. Try as hard as you can, and only if you get stuck, check out the example solution below.
- AFL uses a non-deterministic testing algorithm, so two fuzzing sessions are never the same. That's why I highly recommend **to set a fixed seed (-s 123)**. This way your fuzzing results will be similar to those shown here and that will allow you to follow the exercises more easily.  
- If you find a new vulnerability, **please submit a security report** to the project. If you need help or have any doubt about the process, the [GitHub Security Lab](mailto:securitylab.github.com) can help you with it :)

## Contact
Are you stuck and looking for help? Do you have suggestions for making this course better or just positive feedback so that we create more fuzzing content?
Do you want to share your fuzzing experience with the community?
Join the GitHub Security Lab Slack and head to the `#fuzzing` channel. [Request an invite to the GitHub Security Lab Slack](mailto:securitylab-social@github.com?subject=Request%20an%20invite%20to%20the%20GitHub%20Security%20Lab%20Slack)

## Environment

All the exercises have been tested on **Ubuntu 20.04.2 LTS**. I highly recommend you to use **the same OS version** to avoid different fuzzing results and to run AFL++ **on bare-metal** hardware, and not virtualized machines, for best performance.

Otherwise, you can find an Ubuntu 20.04.2 LTS VMware image [here](https://drive.google.com/file/d/1_m1x-SHcm7Muov2mlmbbt8nkrMYp0Q3K/view?usp=sharing). You can also use VirtualBox instead of VMware.

The username / password for this VM are `fuzz` / `fuzz`.

## Dictionaries

When we want to fuzz complex text-based file formats (such as XML), it’s useful to provide the fuzzer with a dictionary containing a list of basic syntax tokens.

In the case of AFL, such a dictionary is simply a set of words or values which is used by AFL to apply changes to the current in-memory file. Specifically, AFL performs the following changes with the values provided in the dictionary:
- Override: Replaces a specific position with n number of bytes, where n is the length of a dictionary entry.
- Insert: Inserts the dictionary entry at the current file position, forcing all characters to move n positions down and increasing file size.

You can find a good bunch of examples [here](https://github.com/AFLplusplus/AFLplusplus/tree/stable/dictionaries)

## Paralellization

If you have a multi-core system is a good idea to parallelize your fuzzing job to make the most of your CPU resources.

### Independent instances

This is the simplest parallelization strategy. In this mode, we run fully separate instances of afl-fuzz.

It is important to remember that AFL uses a non-deterministic testing algorithm. So, if we run multiples AFL instances, We increase our chances of success.

For this, you only need to run multiple instances of "afl-fuzz" on multiple terminal windows, setting a different "output folder" for each one of them. A simple approach is to run as many fuzzing jobs as cores are on your system.

Note: If you're using the -s flag, you need to use a different seed for each instance

### Shared instances

The use of shared instances is a better approach to parallel fuzzing. In this case, each fuzzer instance gathers any test cases found by other fuzzers.

You will usually have only one master instance at a time:
```
./afl-fuzz -i afl_in -o afl_out -M Master -- ./program @@
```

and N-1 number of slaves:
```
./afl-fuzz -i afl_in -o afl_out -S Slave1 -- ./program @@
./afl-fuzz -i afl_in -o afl_out -S Slave2 -- ./program @@
...
./afl-fuzz -i afl_in -o afl_out -S SlaveN -- ./program @@
```

## Do it yourself!
In order to complete this exercise, you need to:
1) Find an interface application that makes use of the LibXML2 library
2) Copy the [SampleInput.xml](./SampleInput.xml) file to your AFL input folder
3) Create a custom dictionary for fuzzing XML
4) Fuzz LibXML2 until you have a few unique crashes. I recommend you to use as many AFL instances as posible (CPU cores)
5) Triage the crashes to find a PoC for the vulnerability
6) Fix the issues

**Estimated time = 3 hours**


<details>
  <summary>SPOILER ALERT! : Solution inside</summary>

### Download and build your target

Let's first get our fuzzing target. Create a new directory for the project you want to fuzz:
```
cd $HOME
mkdir Fuzzing_libxml2 && cd Fuzzing_libxml2
```

Download and uncompress libxml2-2.9.4.tar.gz
```
wget http://xmlsoft.org/download/libxml2-2.9.4.tar.gz
tar xvf libxml2-2.9.4.tar.gz && cd libxml2-2.9.4/
```

Build and install libxml2:
```
sudo apt-get install python-dev
CC=afl-clang-lto CXX=afl-clang-lto++ CFLAGS="-fsanitize=address" CXXFLAGS="-fsanitize=address" LDFLAGS="-fsanitize=address" ./configure --prefix="$HOME/Fuzzing_libxml2/libxml2-2.9.4/install" --disable-shared --without-debug --without-ftp --without-http --without-legacy --without-python LIBS='-ldl'
make -j$(nproc)
make install
```

Now, we can test that all is working OK with:
```
./xmllint --memory ./test/wml.xml
```

and you should see something like that

![](Images/Image1.png)
  
### Seed corpus creation

First of all, we need to get some XML samples. We're gonna use the **SampleInput.xml** provided in this repository:
```
mkdir afl_in && cd afl_in
wget asdf
cd ..
```
  
### Custom dictionary

Now, you need to create an XML dictionary. Alternatively, you can use the XML dictionary provided with AFL++: 
```
mkdir dictionaries && cd dictionaries
wget https://github.com/AFLplusplus/AFLplusplus/blob/stable/dictionaries/xml.dict
cd ..
```
### Fuzzing time
  
In order to catch the bug, is mandatory to enable the `--valid` parameter. I also set the dictionary path with the **-x flag** and enable the deterministic mutations with the **-D flag** (only for the master fuzzer):

For example, I ran the fuzzer with the following command 
```
afl-fuzz -m none -i ./afl_in -o afl_out -s 123 -x xml.dict -D -M master -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

You can run another slave instance with:
```
afl-fuzz -m none -i ./afl_in -o afl_out -s 234 -S slave1 -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

After a while, you should have multiple crashes:
  
![](Images/Image2.png)

### Triage
  
To debug a program built with ASan is so much easier than in the previous exercises. All you need to do is to feed the program with the crash file:
  
```
./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude './afl_out/default/crashes/id:000000,sig:06,src:003963,time:12456489,op:havoc,rep:4'
```

and you will get a nice summary of the crash, including the execution trace:
  
![](Images/Image3.png)  

### Fix the issue

The last step of the exercise is to fix the bug! Rebuild your target after the fix and check that your PoC don't crash the program anymore. This last part is left as exercise for the student.
  
  <details>
  <summary>Solution inside</summary>
   --------------------------------------------------------------------------------------------------
    
  Official fix:
  - https://github.com/GNOME/libxml2/commit/932cc9896ab41475d4aa429c27d9afd175959d74
    
   </details> 

Alternatively, you can download a newer version of LibXML, and check that the bug has been fixed.
  
</details>


# Exercise 4 - LibTIFF

This time we will fuzz **LibTIFF** image library. The goal is to find a crash/PoC for [**CVE-2016-9297**](https://www.cvedetails.com/cve/CVE-2016-9297/) in libtiff 4.0.4 and **to measure the code coverage data** of your crash/PoC. 

<details>
  <summary>For more information about CVE-2016-9297 vulnerability, click me!</summary>
  --------------------------------------------------------------------------------------------------------
  
**CVE-2016-9297** is an Out-of-bounds Read vulnerability that can be triggered via crafted TIFF_SETGET_C16ASCII or TIFF_SETGET_C32_ASCII tag values.
  
  An Out-of-bounds Read is a vulnerability that occurs when the program reads data past the end, or before the beginning, of the intended buffer.
  
  As a result, it allows remote attackers to cause a denial of service or possibly obtain potentially sensitive information from process memory.
  
  You can find more information about Out-of-bounds Read vulnerabilities at the following link: https://cwe.mitre.org/data/definitions/125.html
  
</details>

## What you will learn
Once you complete this exercise you will know:
- How to measure code coverage using LCOV
- How to use code coverage data to improve the effectiveness of fuzzing

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

## Do it yourself!
In order to complete this exercise, you need to:
1) Fuzz LibTiff (with ASan enabled) until you have a few unique crashes
2) Triage the crashes to find a PoC for the vulnerability
3) Measure the code coverage of this PoC 
4) Fix the issue

**Estimated time = 3 hours**

---------------------------------------------------------------------------------------------------------------------------------------------------

<details>
  <summary>SPOILER ALERT! : Solution inside</summary>

### Download and build your target

Let's first get our fuzzing target. Create a new directory for the project you want to fuzz:
```
cd $HOME
mkdir fuzzing_tiff && cd fuzzing_tiff/
```

Download and uncompress libtiff 4.0.4:
```
wget https://download.osgeo.org/libtiff/tiff-4.0.4.tar.gz
tar -xzvf tiff-4.0.4.tar.gz
```


Now, we can build and install libtiff:
```
cd tiff-4.0.4/
./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```
  
As target binary we can just fuzz the ``tiffinfo`` binary located in the ``/bin`` folder. As seed input corpus, we're gonna use the sample images from the ``/test/images/`` folder.
  
To test everything is working properly, just type:
```
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff
```
  
and you should see something like that
![](Images/Image1.png)
  
In the last command line you can see that I enabled all these flags: "-j -c -r -s -w". This is to improve the **code coverage** and increase the chances of finding the bug.

But how can we measure the code coverage of a given input case?
  
### Code coverage
  
Code coverage is a software metric that show the number of times each line of code is triggered. By using code coverage we will get to know which parts of the code have been reached by the fuzzer and visualizes the fuzzing process.
  
First of all, we need to install lcov. We can do it with the following command:
```
sudo apt install lcov
```

Now we need to rebuild libTIFF with the ``--coverage`` flag (compiler and linker):
```
rm -r $HOME/fuzzing_tiff/install
cd $HOME/fuzzing_tiff/tiff-4.0.4/
make clean
  
CFLAGS="--coverage" LDFLAGS="--coverage" ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```  
Then we can collect code coverage data by typing the following:

```
cd $HOME/fuzzing_tiff/tiff-4.0.4/
lcov --zerocounters --directory ./
lcov --capture --initial --directory ./ --output-file app.info
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff
lcov --no-checksum --directory ./ --capture --output-file app2.info
```
  
I will try to explain each of the commands:
  - ``lcov --zerocounters --directory ./`` : Reset previous counters
  - ``lcov --capture --initial --directory ./ --output-file app.info`` : Return the "baseline" coverage data file that contains zero coverage for every instrumented line
  - ``$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff`` : Run the application you want to analyze . You can run it multiple times with different inputs
  - ``lcov --no-checksum --directory ./ --capture --output-file app2.info``: Save the current coverage state into the app2.info file

Finally, we have to generate the HTML output:
```
genhtml --highlight --legend -output-directory ./html-coverage/ ./app2.info
```
  
If all went well, the code coverage report was created in the ``html-coverage`` folder. Just open the ``./html-coverage/index.html`` file and you should see something like this:
  
![](Images/Image2_2.png)
  
Now you can browse trough the different folders and files, and see the number of times each line was executed.
  
### Fuzzing time

Now we're going to compile libtiff with ASAN enabled.

First of all, we're going to clean all previously compiled object files and executables:
```
rm -r $HOME/fuzzing_tiff/install
cd $HOME/fuzzing_tiff/tiff-4.0.4/
make clean
```

Now, we set AFL_USE_ASAN=1 before calling make:
```
export LLVM_CONFIG="llvm-config-12"
CC=afl-clang-lto ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
AFL_USE_ASAN=1 make -j4
AFL_USE_ASAN=1 make install

```

Now, you can run the fuzzer with the following command:
```
afl-fuzz -m none -i $HOME/fuzzing_tiff/tiff-4.0.4/test/images/ -o $HOME/fuzzing_tiff/out/ -s 123 -- $HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w @@
```

After a few minutes you should see somethink like this:
![](Images/Image2.png)
  
### Triage
  
The ASan trace may looks like:
  
![](Images/Image3.png)
  
### Code coverage measure
  
Now, try to measure the code coverage of your PoC. In order to complete this part, **you need to obtain a coverage html report**, similar to the example above.
  
### Fix the issue

The last step of the exercise is to fix the bug! Rebuild your target after the fix and check that your PoC don't crash the program anymore. This last part is left as exercise for the student.
  
  <details>
  <summary>Solution inside</summary>
   --------------------------------------------------------------------------------------------------
    
  Official fix:
  - https://github.com/vadz/libtiff/commit/30c9234c7fd0dd5e8b1e83ad44370c875a0270ed
    
   </details> 

Alternatively, you can download a newer version of LibTIFF, and check that both bugs have been fixed.
  
  
</details>

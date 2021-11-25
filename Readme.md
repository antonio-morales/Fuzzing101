# Fuzzing-101

Do you want to learn how to fuzz like a real expert, but don't know how to start?

If so, this is the course for you!

**10 real targets, 10 exercises.** Are you able to solve all 10?

## Structure

| Exercise No.  | Target | CVEs to find | Time estimated | Main topics |
| ------------- | ------------- | ------------- |  ------------- | ------------- |
| [Exercise 1](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%201) | Xpdf  | CVE-2019-13288 | 120 mins | Afl-clang-fast, Afl-fuzz, GDB |
| [Exercise 2](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%202)  | libexif  |  CVE-2009-3895, CVE-2012-2836 | 6 hours | Afl-clang-lto, Fuzz libraries, Eclipse IDE|
| [Exercise 3](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%203)  | TCPdump  | CVE-2017-13028 | 4 hours | ASan |
| [Exercise 4](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%204)  | LibTIFF  | CVE-2016-9297 | 3 hours | Code coverage, LCOV |
| [Exercise 5](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%205)  | Libxml2  | CVE-2017-9048 | 3 hours | Dictionaries, Basic parallelization, Fuzzing command-line arguments  |
| [Exercise 6](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%206)  | GIMP  | CVE-2016-4994, Bonus bugs | 7 hours | Persistent fuzzing |
| [Exercise 7](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%207)  | VLC media player | CVE-2019-14776 | 6 hours | Partial instrumentation, Fuzzing harness |
| Exercise 8  | will be released soon  | | |
| Exercise 9  | will be released soon  | | |
| Exercise 10  | will be released soon  || |

## Changelog

- 11/25/2021: Exercise 3 updated with some fixes.


## Who is the course intended for?
- Anyone wishing to learn fuzzing basics
- Anyone who wants to learn how to find vulnerabilities in real software projects.

## Requirements
- All you need for this course is a running Linux system with an internet connection. You will find a suitable VMware image in the exercises.
- At least basic Linux skills are highly recommended.
- All the exercises have been tested on Ubuntu **20.04.2 LTS**. You can download it from [here](https://ubuntu.com/download/desktop/thank-you?version=20.04.2.0&architecture=amd64)
- In this course we're going to use [AFL++](https://github.com/AFLplusplus/AFLplusplus), a newer and superior fork of Michał "lcamtuf" Zalewski's AFL, for solving the fuzzing exercises.

## What is fuzzing?

**Fuzz testing (or fuzzing)** is an automated software testing technique that is based on feeding the program with random/mutated input values and monitoring it for exceptions/crashes.

[AFL](https://github.com/google/AFL), [libFuzzer](https://llvm.org/docs/LibFuzzer.html) and [HonggFuzz](https://github.com/google/honggfuzz) are three of the most successful fuzzers when it comes to real world applications. All three are examples of **Coverage-guided evolutionary** fuzzers.

###  Coverage-guided evolutionary fuzzer

- **Evolutionary**: is a metaheuristic approach inspired by evolutionary algorithms, which basically consists in the evolution and mutation of the initial subset (seeds) over time, by using a selection criteria (ex. coverage).

- **Coverage-guided**: To increase the chance of finding new crashes, coverage-guided fuzzers gather and compare code coverage data between different inputs (usually through instrumentation) and pick those inputs which lead to new execution paths.


<img src="./Diagram.png">

<p align="center">
  Simplification of the coverage gathering process of a coverage-guided evolutionary fuzzer
</p>

## Thanks

Thanks for their help:
- [Xavier RENE-CORAIL](https://github.com/xcorail)
- [Alan Vivona](https://github.com/alanvivona)
- [Jason White](https://github.com/misfir3)
- [Octavio Gianatiempo](https://github.com/ogianatiempo)
- [van Hauser](https://github.com/vanhauser-thc)
- [Marc Poulhiès](https://github.com/dkm)
- [Xu Hanyu](https://github.com/Mundi-Xu)
- [tclan126](https://github.com/tclan126)
- [epi052](https://github.com/epi052)

## Contact

Are you stuck and looking for help? Do you have suggestions for making this course better or just positive feedback so that we can create more fuzzing content?
Do you want to share your fuzzing experience with the community?
Join the GitHub Security Lab Slack and head to the `#fuzzing` channel. [Request an invite to the GitHub Security Lab Slack](mailto:securitylab-social@github.com?subject=Request%20an%20invite%20to%20the%20GitHub%20Security%20Lab%20Slack)


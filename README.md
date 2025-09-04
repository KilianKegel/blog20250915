# Engineering a UEFI C Library
### CdePkgBlog 2025-09-15

# ***Part 1: MATH.H***

[<img src="https://upload.wikimedia.org/wikipedia/commons/1/14/Intel_C8087.jpg" width="800">](https://upload.wikimedia.org/wikipedia/commons/a/aa/Intel_8087_die.JPG)<br><br>
[<img src="https://github.com/KilianKegel/pictures/blob/master/IEEEMilestone.png" width="800">](https://math.berkeley.edu/news/congratulations-professor-william-velvel-kahan)

### Table of content
* [Preface](README.md#Preface)
* [Abstract](README.md#Abstract)
* [Introduction](README.md#Introduction)
* [In a nutshell: **CdePkg**](README.md#in-a-nutshell-cdepkg)
    * [A short comparison with RedfishCrtLib](README.md#a-short-comparison-with-redfishcrtlib)
    * [RedfishCrtLib conflicts with CdePkg by design](README.md#redfishcrtlib-conflicts-with-cdepkg-by-design)
* [Transformation of a traditional UEFI DXE driver to CdePkg driver](README.md#transformation-of-a-traditional-uefi-dxe-driver-to-cdepkg-driver)
    * [Adjusting the .INF file to run CdePkg features](README.md#adjusting-the-inf-file-to-run-cdepkg-features)
    * [Adjusting the ENTRY_POINT-.C file to run CdePkg features](README.md#adjusting-the-entry_point-c-file-to-run-cdepkg-features)
    * [Adding a command line (EmulatorPkg)](README.md#adding-a-command-line-emulatorpkg)
    * [Adding a command line on a real platform](README.md#adding-a-command-line-on-a-real-platform)
* [Perform Standard C in RestJsonStructureDxe.c in the Emulator](README.md#perform-standard-c-in-restjsonstructuredxec-in-the-emulator)
    * [`wctype.h` demonstration](README.md#wctypeh-demonstration)
    * [`wchar.h` demonstration](README.md#wcharh-demonstration)
* [CDETRACE() -- An introduction of a new DEBUG/TRACE concept for UEFI platforms](README.md#cdetrace----an-introduction-of-a-new-debugtrace-concept-for-uefi-platforms)
    * [tianocore `DEBUG()`](README.md#cdetrace----an-introduction-of-a-new-debugtrace-concept-for-uefi-platforms)
    * [`CDETRACE()`](README.md#cdetrace)
* [Coming up soon](README.md#coming-up-soon)

## Preface
Lets celebrate the 45th anniversary of the [**Intel 8087 FPU**](https://en.wikipedia.org/wiki/X87#8087) (floating point unit) and the 40th anniversary of the
[**IEEE 754**](https://de.wikipedia.org/wiki/IEEE_754) floating point standard.


The successor to this revolutionary digital silicon circuitry design was the [**Intel 80387**](https://en.wikipedia.org/wiki/X87#80387) math coprocessor, 
that is still present in current x86 CPU processors.


## Abstract
This article introduces a high precision, high performance and low code size [**`math.h`**](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1256.pdf#page=224) implementation for UEFI drivers, UEFI shell applications and Windows applications on x86 platforms.
It discusses the  design decisions, trade-offs and  describes the validation concept.
Additionally a very short retrospective of floating point calculation is given.

## Introduction
### Retrospective
Until the 1970s a floating point standard did not exist. Each manufacturer of computing systems 
and each programming language implemented its own floating point representation and arithmetic.
That made it difficult to port software between different systems and to compare results of floating point calculations:
[YOUTUBE: William Kahan on the 8087 and designing Intel's floating point](https://www.youtube.com/watch?v=L-QVgbdt_qg)

The company [**Intel**](https://en.wikipedia.org/wiki/Intel), founded in 1968, decided to provide a floating point coprocessor for their new 16-bit microprocessor,
and to stop the chaos of incompatible floating point implementations.

The upcoming [**8087**]() would provide the **entire floating point library** in one chip:
* floating point base functions
  - addition
  - subtraction
  - multiplication
  - division
* floating point transcendental functions 
    * square root
    * logarithm
    * exponential function
    * tangent
    * arcus tangent<br>
      NOTE: sine and cosine and related functions are derived from the tangent/arcus tangent function
* 64 bit integer and packed BCD arithmetic

That time the semiconductor technology was able to produce chips with approximately 40.000 transistors.<br>
The limitation required a very *efficient design* of the FPU (floating point unit) interface and architecture, 
that was challanging for programmers and compiler writers  :<br>
* [**On the Advantages of the 8087’s Stack**](documents/87STACK.pdf)
* [**How Intel 80x87 Stack Over/Underflow Should Have Been Handled**](documents/STACK87.pdf)

NOTE: Concurrent FPU designs ([**Motorola 68881**](https://en.wikipedia.org/wiki/Motorola_68881)) appeared later on the
market with the availability of about 160.000 transistors and a more elegant architecture and interface design.<br>



#### The Intel 8087 and the IEEE 754 Standard
The Intel 8087, introduced in 1980, was the first floating-point coprocessor designed to work with the Intel 8086 and 8088 microprocessors. It was a groundbreaking piece of hardware that significantly enhanced the computational capabilities of personal computers by offloading complex floating-point arithmetic operations from the main CPU.



## Coming up soon...
<del>2021-11-28:<br>                                                </del>      
<del>* Using UEFI- and Standard-C-API in shell applications<br>     </del>
<del>* Creating MSDOS tools.<br>                                    </del>
<del>2021-12-12:<br>                                                </del>
<del>* Adopting open source Visual-Studio projects to UEFI<br>      </del>
<del>* Introduction of the ACPCIA port to UEFI<br>                  </del>

<ins>**2021-12-19:**</ins>
* Redfish on CdePkg<br>

<ins>**2022-01-16:**</ins>
* Adopting open source Visual-Studio projects to UEFI<br>
* Introduction of the ACPCIA port to UEFI<br>

<ins>**2022-01-30:**</ins>
* Introduction of how to calibrate a TSC-based software timer - on IBM-AT compatible UEFI platforms



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
Additionally a short retrospective of floating point calculation is given.

## Introduction
### Retrospective of floating point calculation over history
#### The Story of William Kahan, Robert Palmer, Jerome Coonen, and the IEEE 754 Standard

The IEEE 754 floating point standard, now fundamental to all modern computing, was shaped by the vision and collaboration of several key figures—most notably Professor William Kahan, Robert (Bob) Palmer from Intel, and Jerome (Jerry) Coonen.

**William Kahan**—a professor at the University of California, Berkeley—was the driving force behind the standard. Known as the "father of floating point," Kahan recognized the chaos and incompatibility in floating point arithmetic across different computer systems in the 1970s. He advocated for a universal, reliable, and mathematically sound approach to floating point computation, emphasizing the need for well-defined rounding, exception handling, and special values like NaN (Not a Number) and infinity.

**Robert Palmer** was a senior engineer at Intel and the chief architect of the Intel 8087 floating point coprocessor. Palmer worked closely with Kahan to ensure that the 8087 would implement the features and behaviors Kahan envisioned. Palmer’s technical leadership and willingness to adapt Intel’s hardware design to Kahan’s mathematical requirements were crucial in making the 8087 the first commercial implementation to closely follow what would become the IEEE 754 standard.

**Jerome Coonen** was a mathematician and computer scientist who worked with Kahan at Berkeley and later joined Intel. Coonen contributed significantly to the design and analysis of floating point algorithms and the formalization of the standard. He played a key role in bridging the gap between theoretical mathematics and practical hardware implementation, helping to translate Kahan’s ideas into specifications that could be realized in silicon.

The collaboration between Kahan, Palmer, and Coonen was instrumental in the creation of the IEEE 754 standard, first published in 1985. Their work established a common foundation for floating point arithmetic, enabling portability, predictability, and correctness in scientific and engineering software worldwide. The Intel 8087, guided by their efforts, became the first widely adopted hardware to implement these principles, setting the stage for all subsequent floating point hardware and software.

Today, the legacy of Kahan, Palmer, and Coonen lives on in every modern processor, ensuring that floating point calculations are consistent and reliable across platforms.

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



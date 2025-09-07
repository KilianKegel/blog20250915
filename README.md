# Engineering the UEFI C Library
### CdePkgBlog 2025-09-15

# ***Part 1: math.h***

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

## introductory email
From: Kilian Kegel <
Lets celebrate the 45th anniversary of the [**Intel 8087 FPU**](https://en.wikipedia.org/wiki/X87#8087) (floating point unit) and the 40th anniversary of the
[**IEEE 754**](https://de.wikipedia.org/wiki/IEEE_754) floating point standard.

The successor to this revolutionary digital circuitry design was the [**Intel 80387 FPU**](https://en.wikipedia.org/wiki/X87#80387) , 
that is still present in current x86 CPU processors.

# Preface
The **UEFI C Library**  discussed here is the [**toro C Library**](https://github.com/KilianKegel/toro-C-Library), [source code](https://github.com/KilianKegel/Visual-TORO-C-LIBRARY-for-UEFI). 

**toro C Library** is a *monolithic* Standard C Library for the UEFI x86-64 
target platform.

The **toro C Library** is implemented to target [**ANSI/ISO C Standard Library**]( https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=176) compatibility,
enabling the creation of applications and drivers for UEFI systems using the design and debugging 
infrastructure provided by current **Microsoft C tool chain** in **Visual Studio 2022**.

To implement the [**math.h** functions](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1256.pdf#page=224) the **80387** **FPU** is used.<br>
That allows compact and precise implementation also for **POST** (power on self test) usage.

#### Getting finished ...

# Abstract
This article introduces a high precision, high performance and low code size [**`math.h`**](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1256.pdf#page=224) implementation for UEFI drivers, UEFI shell applications and Windows applications on x86 platforms.
It discusses the  design decisions, tradeoffs and  describes the validation concept.
Additionally a short retrospective of floating point calculation is given.

# Introduction
The **FPU** is the foundation of this math library, providing a space-optimized, 
ROM-able implementation of C's MATH.H functions while maintaining the precision and correctness 
already established in earlier x87-based math libraries.<br>

Since the traditional **FPU** **80387** is still present in current x86 processors and is 
also ***not deprecated*** in the [**X86S specification**](https://www.intel.com/content/www/us/en/developer/articles/technical/envisioning-future-simplified-architecture.html), [**.PDF**](https://github.com/KilianKegel/4KPages-TechDocs/blob/main/x86s-eas-external-1.1.pdf),
it can be safely used here, now and in the future on x86-based platforms.<br>

The **80387** processor has various improvements over its **8087** predecessor, such as<br>
  * range extension for transcendental function:

    | Instruction  | function                      |
    |--------------|-------------------------------|
    |FPTAN         | Partial tangent               |
    |FPATAN        | Partial arctangent            |
    |F2XM1         | 2<sup>x</sup> - 1             |
    |FYL2X         | Y * log<sub>2</sub>X          |
    |FYL2XP1       | Y * log<sub>2</sub>(X + 1)    |
    
* new instructions, e.g.

    | Instruction  | function                      |
    |--------------|-------------------------------|
    |FSIN          | sine                          |
    |FCOS          | cosine                        |

This **FPU** is still today the most precise arithmetic unit in the x86 processor — because it uses 80-bit floating point arithmetic internally.<br>
Additionally it provides on current x86-64 processors a complete set of transcendental functions: **logarithm**, **exponential function**, **sine**, **cosine**, **tangent** and  the corresponding **arcus**-functions, .<br>

Instead the modern [**SSE instruction set**](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions) uses only 64-bit double precision floating point arithmetic
and is not spread internally to higher bit width.<br>
**SSE** is designed for vector processing of arithmetic operations on multiple data sets in parallel.<br>
Calculation of transcendental functions is not supported directly by the **SSE** instruction set.<br>

## Retrospective
Until the 1970s a floating point standard did not exist. Each manufacturer of computing systems 
and each programming language implemented its own floating point representation and arithmetic.
That made it difficult to port software between different systems and to compare results of floating point calculations:
[YOUTUBE: William Kahan on the 8087 and designing Intel's floating point](https://www.youtube.com/watch?v=L-QVgbdt_qg)

The company [**Intel**](https://en.wikipedia.org/wiki/Intel), founded in 1968, decided to provide a floating point coprocessor (**FPU**) for their new 16-bit microprocessor,
and to stop the chaos of incompatible floating point implementations.

The upcoming [**8087**]() would provide the **entire floating point library** in one chip:
* floating point algebraic functions: **addition, subtraction, multiplication, division, square root**
* floating point transcendental functions: **logarithm, exponential function, tangent, arcus tangent** <br>
    NOTE: sine and cosine and related functions are derived from the tangent/arcus tangent function
* floating point constants: **0.0, 1.0, log<sub>2</sub>(e), log<sub>2</sub>(10), log<sub>10</sub>(2), log<sub>e</sub>(2), π**
* 64 bit integer and packed BCD arithmetic

That time the semiconductor technology at **Intel** was able to produce chips with approximately 40,000 transistors.<br>
The limitation required a very *efficient design* of the **FPU** interface and architecture, 
that was challenging for programmers and compiler writers  :<br>
* [**On the Advantages of the 8087’s Stack**](documents/87STACK.pdf)
* [**How Intel 80x87 Stack Over/Underflow Should Have Been Handled**](documents/STACK87.pdf)

NOTE: Competing FPU designs, such as the [**Motorola 68881**](https://en.wikipedia.org/wiki/Motorola_68881), 
entered the market later, as process technology enabled approximately 160,000 transistors, offering an easier-to-use architecture and interface.

### The Intel 8087, the Intel 80387 and the IEEE 754 Standard
The Intel 8087, introduced in 1980, was the first **FPU** designed to work with the 
Intel 8086 and 8088 microprocessors. 

While the **8087** was not fully compliant with the later [**IEEE 754 standard**](https://de.wikipedia.org/wiki/IEEE_754),
it laid the groundwork for future floating-point units. 

In 1985, Intel released the **80387**, which was designed to be fully compliant 
with the [**IEEE 754 standard**](https://de.wikipedia.org/wiki/IEEE_754), that was finalized in the same year.<br>

## Hardware/Software conditions — Library requirements
### Library requirements
The UEFI C Library ([**toro C Library**](https://github.com/KilianKegel/toro-C-Library)) runs on x86-64 platforms
during **POST** in **PEI-**, **DXE-**, **SMM-drivers**, in **UEFI Shell apps** and in **Windows 64/32 console apps**.<br>

**ANSI-C/C90** **`math.h`** functions set requires to provide 22 functions:<br>
**`acos()`**, **`asin()`**, **`atan()`**, **`atan2()`**, **`ceil()`**, **`cos()`**,<br>
**`cosh()`**, **`exp()`**, **`fabs()`**, **`floor()`**, **`fmod()`**, **`frexp()`**, <br>
**`ldexp()`**, **`log()`**, **`log10()`**, **`modf()`**, **`pow()`**, **`sin()`**, <br>
**`sinh()`**, **`sqrt()`**, **`tan()`**, **`tanh()`**<br>


## Hardware conditions
All UEFI-enabled x86-64 platforms provide the **80387** **FPU** and
the **SSE2** instruction set as a minimum.<br>

Access to the **80387** and the **SSE** arithmetic unit is possible in all CPU modes and privilege levels.<br>

## Software conditions
### Getting finished . . .
The entire design and development of the [**toro C Library**](https://github.com/KilianKegel/toro-C-Library)
is done using the latest [**Visual Studio**](https://visualstudio.microsoft.com/vs/) [standard installation for C/C++](https://github.com/KilianKegel/Howto-setup-a-UEFI-Development-PC?tab=readme-ov-file#install-visual-studio-2022).<br>
**Visual Studio** provides a complete and robust C/C++ development environment that offers best build performance and debugging features . . .<br>
**Visual Studio** allows to achieve programmers intention instantly.<br>


The Microsoft C/C++ compiler offers 3 different floating point models in both the 32- and 64-bit codegenerator :<br>
* **precise**
* **fast**
* **strict**

To unveil the secrets beyond these models, a little test program was in charge, that simply invokes the sine **`sin()`** function:<br>
```c
#include <stdlib.h>
#include <math.h>

void main(int argc) 
{
    volatile double d = sin(argc);
}
```
#### 32Bit  machine code model `precise`:
```asm
_main:
  00000000: 55                 push        ebp
  00000001: 8B EC              mov         ebp,esp
  00000003: 83 EC 08           sub         esp,8
  00000006: 66 0F 6E 45 08     movd        xmm0,dword ptr [ebp+8]       ; load integer parameter argc into xmm0
  0000000B: F3 0F E6 C0        cvtdq2pd    xmm0,xmm0                    ; CVTDQ2PD — Convert Packed Doubleword Integers to Packed 
                                                                        ;            Double Precision Floating-PointValues
  0000000F: E8 00 00 00 00     call        __libm_sse2_sin_precise      ; invoke sin() with parameter in xmm0
                                                                        ; NOTE: __libm_sse2_sin_precise is the function name generated
                                                                        ; by the Microsoft C/C++ compiler in PRECISE mode
  00000014: F2 0F 11 45 F8     movsd       mmword ptr [ebp-8],xmm0      ; result is returned in xmm0 and stored in the local variable d
  00000019: 33 C0              xor         eax,eax                      ; return code 0 in eax
  0000001B: 8B E5              mov         esp,ebp
  0000001D: 5D                 pop         ebp
  0000001E: C3                 ret
```
#### 32Bit  machine code model `strict`:
```asm
_main:
  00000000: 55                 push        ebp
  00000001: 8B EC              mov         ebp,esp
  00000003: 83 EC 08           sub         esp,8
  00000006: 66 0F 6E 45 08     movd        xmm0,dword ptr [ebp+8]       ; load integer parameter argc into xmm0
  0000000B: 83 EC 08           sub         esp,8
  0000000B: F3 0F E6 C0        cvtdq2pd    xmm0,xmm0                    ; CVTDQ2PD — Convert Packed Doubleword Integers to Packed 
                                                                        ;            Double Precision Floating-PointValues
  00000012: F2 0F 11 04 24     movsd       mmword ptr [esp],xmm0        ; store parameter on stack
  00000017: E8 00 00 00 00     call        _sin                         ; invoke sin() with parameter on stack
                                                                        ; NOTE: _sin is the function name generated in STRICT mode
  0000001C: 83 C4 08           add         esp,8
  0000001F: 33 C0              xor         eax,eax                      ; return code 0 in eax
  00000021: DD 5D F8           fstp        qword ptr [ebp-8]            ; result is returned in ST(0) and stored in the local variable d
  00000024: 8B E5              mov         esp,ebp
  00000026: 5D                 pop         ebp
  00000027: C3                 ret
```

#### 32Bit  machine code model `fast`:
```asm
_main:
  00000000: 55                 push        ebp
  00000001: 8B EC              mov         ebp,esp
  00000003: 83 EC 08           sub         esp,8
  00000006: 66 0F 6E 45 08     movd        xmm0,dword ptr [ebp+8]       ; load integer parameter argc into xmm0
  0000000B: F3 0F E6 C0        cvtdq2pd    xmm0,xmm0                    ; CVTDQ2PD — Convert Packed Doubleword Integers to Packed
                                                                        ;            Double Precision Floating-PointValues
  0000000F: E8 00 00 00 00     call        ___libm_sse2_sin             ; invoke sin() with parameter in xmm0
                                                                        ; NOTE: ___libm_sse2_sin is the function name generated
                                                                        ; by the Microsoft C/C++ compiler in FAST mode
  00000014: F2 0F 11 45 F8     movsd       mmword ptr [ebp-8],xmm0      ; result is returned in xmm0 and stored in the local variable d
  00000019: 33 C0              xor         eax,eax                      ; return code 0 in eax
  0000001B: 8B E5              mov         esp,ebp
  0000001D: 5D                 pop         ebp
  0000001E: C3                 ret
```

#### 64Bit  machine code model `precise`, `strict`and `fast`:
NOTE: In 64Bit mode **`precise`**, **`strict`** and **`fast`** model the parameter is passed in the **XMM0** register and the result is returned in the **XMM0** register.<br>
```asm
main:
  00000000: 48 83 EC 28        sub         rsp,28h
  00000004: 66 0F 6E C1        movd        xmm0,ecx                     ; load integer parameter argc into xmm0
  00000008: f3 0f e6 c0        cvtdq2pd    xmm0,xmm0                    ; CVTDQ2PD — Convert Packed Doubleword Integers to Packed
                                                                        ;            Double Precision Floating-PointValues
  0000000C: E8 00 00 00 00     call        sin                          ; invoke sin() with parameter in xmm0
  00000011: F2 0F 11 44 24 38  movsd       mmword ptr [rsp+38h],xmm0    ; result is returned in xmm0 and stored in the local variable d
  00000017: 33 C0              xor         eax,eax                      ; return code 0 in eax
  00000019: 48 83 C4 28        add         rsp,28h
  0000001D: C3                 ret
```

To allow portability to other compilers too the true function name should be generated and not some compiler-specific alias.<br>
So it came out, that the **`strict`** model is the best choice for the **toro C Library**, because it generates the true function name
in 64Bit and 32Bit mode, e.g. **`sin`**.

### 8087 FPU LOAD/STORE instructions 
The 8087 I/O interface is based on a set of [**LOAD**](https://www.felixcloutier.com/x86/fst:fstp) and [**STORE**](https://www.felixcloutier.com/x86/fst:fstp)
instructions to transfer data between memory and the FPU register stack.<br>
There is no direct register-to-register transfer instruction.<br>

That means when parameters/results are passed/returned via XMM registers, the data must be transferred between XMM and FPU registers via memory.<br>

### Getting finished . . . The Precision Tradeoff / 
**SSE2** instructions are used to transfer data between XMM registers and memory.<br>
To simplify  program flow and use of built-in floating point compiler capabilities,
pre- and postprocessing of **8087** **FPU** parameters/results are done in C language.<br>
To implement math.h functions to run entirely inside **8087** (internally in 80Bit precision) would allow
higher precision and faster processing, but would require much more time to implement and validate.

With the **SSE/8087** mixed approach that task could be finished. . . 

### Demonstrating sin()
The implemented sine  **`sin()`** function is here:<br>
```c
[00]    double __cdecl sin(double x)
[01]    {
[02]        CDEDOUBLE d = { .dbl = x };
[03]        CDEDOUBLE dRet = { .uint64 = 0x7FF8042000000000LL };
[04]        
[05]        do 
[06]        {
[07]    
[08]            if (d.member.exp == 0x7FF)                                  // check for NaN or INF
[09]            {
[10]                dRet.uint64 = 0x0008000000000000ULL | d.uint64;
[11]    
[12]                if (0ULL == d.member.mant)
[13]                    dRet.uint64 = 0x8008000000000000ULL | d.uint64,
[14]                    errno = EDOM;                                       // set errno to EDOM for sin(NaN)=NaN and sin(INF)=NaN
[15]    
[16]                break;
[17]            }
[18]    
[19]            if ((63 + 1023) > d.member.exp)
[20]                dRet.dbl = __cde80387FSIN(x);                           // invoke 80387 FSIN instruction 
[21]            else
[22]                errno = ERANGE;
[23]    
[24]        } while (0);
[25]    
[26]        return dRet.dbl;
[27]    }
```
After testing parameters for number range and validity (NaN, INF) the actual sine calculation is done in line [20] by invoking the helper function **`__cde80387FSIN()`**:<br>

#### 32Bit machine code :
```asm
[00]    __cde80387FSIN proc C public float64:QWORD
[01]    
[02]        fld float64     ; load parameter x from stack into ST(0)
[03]    
[04]        FSIN            ; calculate sine of ST(0), result is returned in ST(0)
[05]    
[06]        ret
[07]    
[08]    __cde80387FSIN endp
```
#### 64Bit machine code :
```asm
[00]    __cde80387FSIN proc
[01]    
[02]        local float64:QWORD
[03]    
[04]        movsd float64,xmm0
[05]    
[06]        fld float64
[07]    
[08]        FSIN
[09]    
[10]        fstp float64
[11]    
[12]        movsd xmm0,float64
[13]    
[14]        ret
[15]    
[16]    __cde80387FSIN endp
```

//As [**toro C Library**](https://github.com/KilianKegel/toro-C-Library) is 





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



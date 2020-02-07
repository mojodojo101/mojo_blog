---
layout: post
title: Having A Look At Portable Executables
date: 2020-01-30
author: Mojo
categories: [""]
---

# **Examining Windows EXEs and DLLs**


#### This is the source code i used to create an example executable. 
#### I used Visual Studio 2019s default compiler and C++

#### When you dont understand something try to crossreference it with this link:
[Microsoft PE. docs.microsoft.com](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)

#### The other important Source i used is this PDF:
[Goppit Portable Executable. stondecoder.ord](http://www.stonedcoder.org/~kd/lib/CBJ-2005-74.pdf)

```cpp
#include <cstdio>

static void sayHello(char* name)
{
	printf("Hello %s",name);
}

int main(int argc, char* argv[])
{
	char* name = argv[1];
	sayHello(name);
}
```

This code will simply take the first argument we pass to our Exe and print "Hello "+ your argument.


### Portable Executable
    

#### DOS Header 64 bytes 

|:----------------+:---------------+:-----------------------------------------------------------|
|Offset           |Value           | Meaning                                                    |
|-----------------|----------------|------------------------------------------------------------|
|0x00             |0x4D5A or MZ    |E-magic: Stands for Mark Zbikowsky who created the PE format|
|0x3c             |0x0100          |E-lfanew: The offset to the start of the Pe Header          |

![Dos Header](/mojo_blog/assets/pictures/portable-executable/pe-hex-dos-header.PNG)



#### PE Header

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0100           |0x50450000      |Signature: "PE" folloewd by 2x 0x00|
|0x0104           |20 bytes        |Image File Header                  |
|0x0118           |224 bytes       |Optional Header                    |

#### File Header

The location of the Header will depend on the E-lfanew value in the Dos Header

|:----------------+:---------------+:--------------------------------------------------------|
|Offset           |Value           | Meaning                                                 |
|-----------------|----------------|---------------------------------------------------------|
|0x0104           |0x4c01          |Machine: For example i386                                |
|0x0106           |0x0500          |Number of sections: .text.rdata.data.rsrc.reloc          |
|0x0108           |0x20663C5E      |Time of creation: 2020/02/06 19:16:48                    |
|0x010c           |0x00000000      |Pointer to symbol table: 0 here because not debug Version|
|0x0110           |0x00000000      |Number of symbols                                        |
|0x0114           |0xE000          |Size of Optional Header                                  |
|0x0116           |0x0201          |Characteristics: see below                               |


![PE File Header](/mojo_blog/assets/pictures/portable-executable/pe-file-header.PNG)

[Characteristics](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#characteristics)

<details><summary>Characteristics 0x0201</summary>
   
   1. MAGE_FILE_RELOCS_STRIPPED 0x0001. Image only, Windows CE, and Microsoft Windows NT and later. This indicates that the file does not        contain base relocations and must therefore be loaded at its preferred base address. If the base address is not available, the reports     an error. The default behavior of the linker is to strip base relocations from executable (EXE) files.
   
    2. IMAGE_FILE_DEBUG_STRIPPED 0x0200.Debugging information is removed from the image file.

</details>



#### Optional Header 224 bytes

The Last 128 bytes contain the Data Directory

[Optional Header. docs.microsoft.com](https://docs.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_optional_header32)

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0118           |0x0b01          |Magic Number: either 0x10b or 0x20b|
|0x011a           |0x0E18          |Linker Version                     |
|0x011c           |0x000E0000      |Size of Code Section               |
|0x0120           |0x00140000      |Size of Initialized Data           |
|0x0124           |0x00000000      |Size of Unitialied Data            |
|0x0128           |0xB2120000      |Address of Entrypoint              |
|0x012c           |0x00100000      |Base of Code                       |
|0x0130           |0x00200000      |Base of Data                       |
|0x0134           |0x00004000      |Image Base   !!Important           |
|0x0138           |0x00100000      |Section Alignment !! >= File Align |
|0x013c           |0x00020000      |File Alignment                     |              
|0x0140+16 bytes  | versions       |A bunch of version Stuff           |
|0x0150           |0x00600000      |Size of Image                      |
|0x0154           |0x00400000      |Size of Headers                    |
|0x0158           |0x00000000      |CheckSum                           |
|0x015c           |0x0300          |Subsytem                           |
|0x015e           |0x4081          |DllCharacteristics  !!Important    |
|0x0160           |0x00001000      |Size of Stack Reserve              |
|0x0164           |0x00100000      |Size of Heap Reserve               |
|0x0168           |0x00001000      |Size of Heap Commit                |
|0x016c           |0x00100000      |Loader Flags [^1]                  |
|0x0170           |0x00000000      |Number of directory entries        |
|0x0174           |0x10000000      |Virtual address of Data directory  |
|0x0178           |0x00000000      |Size of Data directory             |


![Optional Header](/mojo_blog/assets/pictures/portable-executable/pe-optional-header.PNG)

















[^1]: Certain debuggers cant handle corrupted flags + number of RVA and sizes and will run the exe without debugging. Pretty interesting for Malware









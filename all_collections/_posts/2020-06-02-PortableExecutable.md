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
    

<details><summary>DOS Header 64 bytes </summary>

|:----------------+:---------------+:-----------------------------------------------------------|
|Offset           |Value           | Meaning                                                    |
|-----------------|----------------|------------------------------------------------------------|
|0x00             |0x4D5A or MZ    |E-magic: Stands for Mark Zbikowsky who created the PE format|
|0x3c             |0x0100          |E-lfanew: The offset to the start of the Pe Header          |

![Dos Header](/mojo_blog/assets/pictures/portable-executable/pe-hex-dos-header.PNG)

</details>

<details><summary>PE Header</summary>

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0100           |0x50450000      |Signature: "PE" folloewd by 2x 0x00|
|0x0104           |20 bytes        |Image File Header                  |
|0x0118           |224 bytes       |Optional Header                    |

<details><summary>File Header</summary>

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

<details><summary>Characteristics 0x0201</summary>

![Characteristics](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#characteristics)

1. MAGE_FILE_RELOCS_STRIPPED 0x0001. Image only, Windows CE, and Microsoft Windows NT and later. This indicates that the file does not contain base relocations and must therefore be loaded at its preferred base address. If the base address is not available, the reports an   error. The default behavior of the linker is to strip base relocations from executable (EXE) files.
2. IMAGE_FILE_DEBUG_STRIPPED 0x0200.Debugging information is removed from the image file.

![PE File Header](/mojo_blog/assets/pictures/portable-executable/pe-file-header.PNG)

</details>

<details><summary>Optional Header 224 bytes</summary>

The Last 128 bytes contain the Data Directory

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0118           |0x0b01          |Magic Number: either 0x10b or 0x20b|

</details>







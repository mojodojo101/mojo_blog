---
layout: post
title: Having A Look At Portable Executables
date: 2020-02-06
author: Mojo
categories: ["Portable Executables"]
---

# **Examining Windows EXEs and DLLs**
#### Tools:
* [HxD](https://mh-nexus.de/en/hxd/)
* [Visual Studio](https://visualstudio.microsoft.com/de/vs/?rr=https%3A%2F%2Fwww.google.com%2F)
* [Pe Explorer](http://www.heaventools.com/)
* [Immunity](https://www.immunityinc.com/products/debugger/)



#### This is the source code i used to create an example executable. 

#### I used Visual Studio 2019s default compiler and C++

#### When you don't understand something try to cross reference it with this link:
[Microsoft PE. docs.microsoft.com](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)

#### The other important Source i used is this PDF:
[Goppit Portable Executable. stondecoder.ord](http://www.stonedcoder.org/~kd/lib/CBJ-2005-74.pdf)

#### There are small differences between x86 and x64 i will focus on x86.

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

![Dos Header](/assets/pictures/portable-executable/pe-hex-dos-header.PNG)



#### PE Header

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0100           |0x50450000      |Signature: "PE" folloewd by 2x 0x00|
|0x0104           |20 bytes        |Image File Header                  |
|0x0118           |224 bytes       |Optional Header                    |

#### File Header

![PE File Header](/assets/pictures/portable-executable/pe-file-header.PNG)

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




[Characteristics](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#characteristics)

<details><summary>Characteristics 0x0201</summary>
   
   1. MAGE_FILE_RELOCS_STRIPPED 0x0001. Image only, Windows CE, and Microsoft Windows NT and later. This indicates that the file does not        contain base relocations and must therefore be loaded at its preferred base address. If the base address is not available, the reports     an error. The default behavior of the linker is to strip base relocations from executable (EXE) files.
   
    2. IMAGE_FILE_DEBUG_STRIPPED 0x0200.Debugging information is removed from the image file.

</details>



#### Optional Header 224 bytes

![Optional Header](/assets/pictures/portable-executable/pe-optional-header.PNG)

The Last 128 bytes contain the Data Directory

[Optional Header. docs.microsoft.com](https://docs.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_optional_header32)

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0118           |0x0b01          |Magic Number: either 0x10b or 0x20b|
|0x011a           |0x0E18          |Linker Version                     |
|0x011c           |0x000E0000      |Size of Code Section               |
|0x0120           |0x00140000      |Size of Initialized Data           |
|0x0124           |0x00000000      |Size of Unitialized Data            |
|0x0128           |0xB2120000      |Address of Entrypoint   [^1]       |
|0x012c           |0x00100000      |Base of Code                       |
|0x0130           |0x00200000      |Base of Data                       |
|0x0134           |0x00004000      |Image Base [^2]		               |
|0x0138           |0x00100000      |Section Alignment [^3]			   |
|0x013c           |0x00020000      |File Alignment                     |              
|0x0140+16 bytes  | versions       |A bunch of version Stuff           |
|0x0150           |0x00600000      |Size of Image 	                   |
|0x0154           |0x00400000      |Size of Headers                    |
|0x0158           |0x00000000      |CheckSum                           |
|0x015c           |0x0300          |Subsystem                           |
|0x015e           |0x4081          |DllCharacteristics  !!Important    |
|0x0160           |0x00001000      |Size of Stack Reserve              |
|0x0164           |0x00100000      |Size of Stack Commit               |
|0x0168           |0x00001000      |Size of Heap Reserve               |
|0x016c           |0x00100000      |Size of Heap Commit                |
|0x0170           |0x00000000      |Loader Flags [^6]                  |
|0x0174           |0x10000000      |Number of directory entries [^7]   |
|0x0178           |0x00000000      |Virtual address of Data Dir        |
|0x017c           |0x00000000      |Size of Data directory             |





An example how this looks mapped to memory in the debugger. [Immunity](https://www.immunityinc.com/products/debugger/) :

![Pe Header. Immunity](/assets/pictures/portable-executable/pe-debugger-peheader.PNG)

#### Data Directory 

![Pe Data Directory](/assets/pictures/portable-executable/pe-data-directory-entrys.PNG)

In our case we have 16 Data Directory Entrys

|:----------------+:---------------+:----------------------------------|
|Offset           |Value           | Meaning                           |
|-----------------|----------------|-----------------------------------|
|0x0178			  |0x00000000	   |Export Table Address=0 [^10]       |
|0x017c			  |0x00000000	   |Export Table Size=0				   |
|0x0180			  |0x4c250000	   |Import Table Address   [^11]	   |
|0x0184			  |0xA0000000	   |Import Table Size=10			   |
|0x0188			  |0x00400000	   |Resource Table Address 		  	   |
|0x018c			  |0xE0010000	   |Resource Table Size				   |
|0x0190			  |0x00000000	   |Exeption Table Address			   |
|0x0194			  |0x00000000	   |Exception Table Size			   |
|0x0198			  |0x00000000	   |Certificate Table Address 	       |
|0x019c			  |0x00000000	   |Certificate Table Size			   |
|0x01a0			  |0x00500000	   |Relocation Table Address 		   |
|0x01a4			  |0x54010000	   |Relocation Table Size			   |
|0x01a8			  |0x20210000	   |Debug Data Address				   |
|0x01ac			  |0x70000000	   |Debug Data Size					   |
|0x01b0			  |0x00000000	   |Architecture Data Address 		   |
|0x01b4			  |0x00000000	   |Architecture Data Size			   |
|0x01b8			  |0x00000000	   |Global Pointer Address			   |
|0x01bc		      |0x00000000	   |Must be 0						   |
|0x01c0		      |0x00000000	   |Thread Local Storage Address	   |
|0x01c4		      |0x00000000	   |Thread Local Storage Size		   |
|0x01c8		      |0x90210000	   |Load Config Table Address		   |
|0x01cc		      |0x40000000	   |Load Config Table Size			   |
|0x01d0		      |0x00000000	   |Bound Import Table Address		   |
|0x01d4		      |0x00000000	   |Bound Import Table Size			   |
|0x01d8		      |0x00200000	   |Import Address Table Adress		   |
|0x01dc		      |0xC4000000	   |Import Address Table Size		   |
|0x01e0		      |0x00000000	   |Delay-Load Import Table Addr	   |
|0x01e4		      |0x00000000 	   |Delay-Load Import Table Size	   |
|0x01e8		      |0x00000000	   |CLR Header Address				   |
|0x01f0			  |0x00000000	   |CLR Header Size					   |
|0x01f4			  |0x00000000      |Reserved						   |
|0x01f8			  |0x00000000      |Reserved						   |




#### Section Headers

![Pe Section Header](/assets/pictures/portable-executable/pe-header-section-text.PNG)

##### I will only list the .text section for this one.

##### The name of a Section are kept in the first 8 bytes of the Header 

|:----------------+:-----------------+:----------------------------------|
|Offset           |Value             | Meaning                           |
|-----------------|------------------|-----------------------------------|
|0x01F8		      |0x2E74657874000000|Name of Section (here .text)	     |
|0x0200			  |0xFC0C0000        |Virtual Size						 |				     			   			   
|0x0208			  |0x00100000	     |Virtual Address					 |
|0x020C			  |0x000E0000	     |Size of Raw Data				     |				   
|0x0210			  |0x00040000	     |Pointer to raw Data				 |			   
|0x0214			  |0x00000000	     |Pointer to Relocations			 |			   
|0x0218			  |0x00000000	     |Pointer to Linenumbers		     |			   
|0x0219			  |0x0000		     |Number of Relocations			     |
|0x021B			  |0x0000		     |Number of Linenumbers			     |
|0x021D			  |0x20000006	     |Characteristics /permissions      |




![Pe Sections. Immunity](/assets/pictures/portable-executable/pe-debugger-sections.PNG)

[Understanding imports. sandsprite.com](http://www.sandsprite.com/CodeStuff/Understanding_imports.html)

#### Footnotes:

[^1]: When you attach a debugger the Entry Point will be the location in Virtual Address Space where EIP will first be located.
[^2]: The Image Base is the point in the Virtual Address Space where you will find the first byte of your Image.
[^3]: The Section alignment has to be greater than  or equal to the File alignment. This variable determines how memory is mapped from your disk to random access memory.
[^6]: Certain debuggers cant handle corrupted flags + number of RVA and sizes and will run the exe without debugging. Pretty interesting for malware.
[^7]: Each one of the Data Directories has 2 fields. The first one is a pointer to the Directory and the second one determines the Size. In our example there are 16 Data Directories. These Directories contain stuff like import and export table, so they are definitely important for reverse engineering. 
[^10]: When a table has no entries the data directory will contain 0x00000000
[^11]: Might want to read up on the [Import directory table](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format#import-directory-table) 










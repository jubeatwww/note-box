
[ELF文件解析（一）：Segment和Section](https://www.cnblogs.com/jiqingwu/p/elf_format_research_01.html)
[elf 文件格式-1- 可执行文件](https://zhuanlan.zhihu.com/p/363488456)

![[Pasted image 20230930180316.png]]

[What's the difference of section and segment in ELF file format](https://stackoverflow.com/questions/14361248/whats-the-difference-of-section-and-segment-in-elf-file-format)
- 一個 segment 包含若干個 section
> A segment can contain 0 or more sections

- the segments contain information needed at ==runtime==
	- segment是可執行文件中的概念
	- 是讓loader
- the sections contain information needed during ==linking==
	- 針對linker
	- 例如: `.text` `.rodata` `.bss`
對於process的loading來說完全不關心數據屬於哪個section, 比較重要的是將操作權限一樣的部分組合在一起(segment)，不然過多的section會浪費內存

```shell
readelf -l /bin/date

Elf file type is EXEC (Executable file)
Entry point 0x402000
There are 9 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x00000000000001f8 0x00000000000001f8  R E    8
  INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238
                 0x000000000000001c 0x000000000000001c  R      1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x000000000000d5ac 0x000000000000d5ac  R E    200000
  LOAD           0x000000000000de10 0x000000000060de10 0x000000000060de10
                 0x0000000000000440 0x0000000000000610  RW     200000
  DYNAMIC        0x000000000000de38 0x000000000060de38 0x000000000060de38
                 0x00000000000001a0 0x00000000000001a0  RW     8
  NOTE           0x0000000000000254 0x0000000000400254 0x0000000000400254
                 0x0000000000000044 0x0000000000000044  R      4
  GNU_EH_FRAME   0x000000000000c700 0x000000000040c700 0x000000000040c700
                 0x00000000000002a4 0x00000000000002a4  R      4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     8
  GNU_RELRO      0x000000000000de10 0x000000000060de10 0x000000000060de10
                 0x00000000000001f0 0x00000000000001f0  R      1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame 
   03     .ctors .dtors .jcr .dynamic .got .got.plt .data .bss 
   04     .dynamic 
   05     .note.ABI-tag .note.gnu.build-id 
   06     .eh_frame_hdr 
   07     
   08     .ctors .dtors .jcr .dynamic .got 
```

![[Pasted image 20230930201735.png]]
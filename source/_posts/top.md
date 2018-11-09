---
title: top
date: 2018-10-28 22:19:48
tags:
---
linux下的top命令参数说明 （virt,res,shr,data 的意义）  
  
%mem 内存使用率  
virt 是虚拟内存  
res是常驻内存  
shr是共享内存  
  <!-- more --> 
top命令下按f键可以看到详细说明  
\* A: PID        = Process Id  
\* E: USER       = User Name  
\* H: PR         = Priority  
\* I: NI         = Nice value  
\* O: VIRT       = Virtual Image (kb)  
\* Q: RES        = Resident size (kb)  
\* T: SHR        = Shared Mem size (kb)  
\* W: S          = Process Status  
\* K: %CPU       = CPU usage  
\* N: %MEM       = Memory usage (RES)  
\* M: TIME+      = CPU Time, hundredths  
b: PPID       = Parent Process Pid  
c: RUSER      = Real user name  
d: UID        = User Id  
f: GROUP      = Group Name  
g: TTY        = Controlling Tty  
j: P          = Last used cpu (SMP)  
p: SWAP       = Swapped size (kb)  
l: TIME       = CPU Time  
r: CODE       = Code size (kb)  
s: DATA       = Data+Stack size (kb)  
u: nFLT       = Page Fault count  
v: nDRT       = Dirty Pages count  
y: WCHAN      = Sleeping in Function  
z: Flags      = Task Flags <sched.h>  
\* X: COMMAND    = Command name/line  
  
top命令下要查看某个用户启动的进程：先输入u，然后输入用户名，再回车  
  
VIRT：virtual memory usage。Virtual这个词很神，一般解释是：virtual adj.虚的, 实质的, \[物\]有效的, 事实上的。到底是虚的还是实的？让Google给Define之后，将就明白一点，就是这东西还是非物质的，但是有效果的，不发生在真实世界的，发生在软件世界的等等。这个内存使用就是一个应用占有的地址空间，只是要应用程序要求的，就全算在这里，而不管它真的用了没有。写程序怕出错，又不在乎占用的时候，多开点内存也是很正常的。  
RES：resident memory usage。常驻内存。这个值就是该应用程序真的使用的内存，但还有两个小问题，一是有些东西可能放在交换盘上了（SWAP），二是有些内存可能是共享的。  
SHR：shared memory。共享内存。就是说这一块内存空间有可能也被其他应用程序使用着；而Virt － Shr似乎就是这个程序所要求的并且没有共享的内存空间。  
DATA：数据占用的内存。如果top没有显示，按f键可以显示出来。这一块是真正的该程序要求的数据空间，是真正在运行中要使用的。  
  
============================================  
  
top里面描述进程内存使用量的数据来源于/proc/$pid/statm这个文件。通过观察kernel的代码就能弄清楚SHR,VIRT和RES这些数值的具体含义。  
  
Linux通过一个叫做 task_statm 的函数来返回进程的内存使用状况  
  
int task\_statm(struct mm\_struct \*mm, int \*shared, int *text,  
               int \*data, int \*resident)  
{  
        *shared = get\_mm\_counter(mm, file_rss);  
        *text = (PAGE\_ALIGN(mm->end\_code) - (mm->start\_code & PAGE\_MASK))  
                                                           >\> PAGE_SHIFT;  
        *data = mm->total\_vm - mm->shared\_vm;  
        \*resident = \*shared + get\_mm\_counter(mm, anon_rss);  
        return mm->total_vm;  
}  
上面的代码中shared就是page cache里面实际使用了的物理内存的页数，text是代码所占用的页  
数，data是总虚拟内存页数减去共享的虚拟内存页数，resident是所有在使用的物理内存的页  
数。最后返回的mm->total_vm是进程虚拟内存的寻址空间大小。  
  
函数get\_mm\_counter并不会做什么计算，它的功能是保证数值读取的原子性。  
  
上面的数值最后会通过 procfs输出 到/proc/$pid/statm中去。他们与top显示的数值对应关系如下。  
  
SHR: shared  
RES: resident  
VIRT: mm->total_vm  
CODE: code  
DATA: data  
  
=======================================  
  
o: VIRT (Virtual Image) - 进程使用的总虚拟内存 (virtual memory) 大小，包括进程的程序码、资料和共享程序库再加上被置换 (swap out) 的空间。VIRT = SWAP + RES  
p: SWAP (Swapped size) - 进程被置换的虚拟内存空间大小。  
q: RES (Resident size) - 进程非被置换的实体内存大小。RES = CODE + DATA  
r: CODE' (Code size) - 进程的程序码在实体内存佔用空间大小，亦叫作 text resident set (TRS)。  
s: DATA (Data+Stack size) - 进程佔用实体内存中的非程序码部份大小，亦叫作 data resident set (DRS)。  
t: SHR (Shared Mem size) - 进程使用的共享内存大小，即可以和其他进程共享的内存空间。  
n: %MEM (Memory usage) - 进程佔用实体内存大小对系统总实体内存大小的比例，以百分比显示。
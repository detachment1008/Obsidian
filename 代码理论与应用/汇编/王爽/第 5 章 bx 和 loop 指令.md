# 引言

## 1.\[bx] 和内存单元的描述

如果用立即数描述内存单元：`mov ax,[0]`
1. 内存单元的偏移地址(段地址在 ds 中)
2. 内存单元的长度(字节，用寄存器的长度指出)

如果用寄存器描述内存单元(注意：只能用寄存器 bx)：`mov al,[bx]`
1. 偏移地址由寄存器 bx 中的数值给出(段地址仍在 ds 中)
2. 取出的大小是 1 个字节，由 al 寄存器的长度指出

## 2.loop

即循环相关的指令

## 3.我们定义的描述性的符号："()"

如：(ax) 表示 ax 寄存器中的内容，(20000H) 表示 内存 20000H 单元的内容

() 中的元素有 2 种类型：
1. 寄存器名
2. 内存单元的物理地址

## 4.约定符号 idata 表示常量

如：`mov ax,[1]` 可以通用的表示为 `mov ax,[idata]`

# 5.1 \[BX]

使用方式：
1. `mov ax,[bx]`
2. `mov [bx],ax`

# 5.2 Loop 指令

loop 指令的格式：`loop 标号`

loop 指令的操作：
1. `(cx) = (cx) - 1`
2. 判断 cx 中的值，不为 0 则跳转到标号处执行，为 0 则继续向下执行

即 cx 寄存器中的值影响着 loop 指令的执行结果

如：计算 $2^{12}$ 的结果
```
assume cs:code
code segment
	mov ax,2
	mov cx,11
s:  add ax,ax
    loop s
	mov ax,4c00h
	int 21h
code ends
end
```

cx 中存放的是真正的循环次数，即 如果 `mov cx,11`，后面的就会执行 11 次

# 5.3 在 Debug 中跟踪用 loop 指令实现的循环程序

如：计算 ffff:0006 单元中的数乘以 3，结果存放在 dx 中

```
assume cs:code
code segment
	mov ax,0ffffh
	mov ds,ax
	mov bx,6

	mov al,[bx]
	mov ah,0

	mov dx,0
	
	mov cx,3
 s: add dx,ax
    loop s

	mov ax,4c00h
	int 21h
code ends
end
```
- 注意：在汇编源程序中，数据不能以字母开头，所以在写 ffffh 时，要写为 0ffffh

# 5.4 Debug 和汇编编译器 masm 对指令的不同处理

debug 中，我们可以通过：`mov ax,[0]` 来将 0 偏移地址的内存单元传送给 ax 寄存器，但是如果不通过 debug ，而是直接运行，那么 `[0]` 就会被当作为立即数 0 进行处理了

所以，只能通过寄存器 bx 进行处理，即 `mov ax,[bx]`，亦或这样处理：`mov ax,ds:[0]`

# 5.5 loop 和 \[bx] 的联合应用

有这样一个问题：计算 ffff:0~ffff:b 单元中的数据之和，结果存放到 dx 中

但是用单字节的 dh 或 dl 存储再相加，结果可能会越界

一种解决办法：使用一个大一点的寄存器，如：ax，作为中介，即赋值给 al，然后用 ax 和 dx 进行加法计算

可以利用 `inc bx`，让 bx 寄存器自加 1，然后通过循环寻址计算即可

# 5.6 段前缀

内存单元的段地址，可以由任何的段寄存器来表示，只是默认的为 ds：
1. ds
2. cs
3. ss
4. es

这些段寄存器被使用时，被称为段前缀，用于显示指明内存单元的段地址

# 5.7 一段安全的空间

当不确定一段内存空间中是否存放着重要的数据或代码时，不能随意向其中写入内容

在实模式时，如果操作覆盖了重要的数据或代码，可能引起死机；但是在保护模式下，不会，因为操作会被限制

但是我们尽量直接对硬件编程，即不想通过操作系统的管理从而安全的编程，在 DOS 系统中，0:200 ~ 0:2ff 这段空间一般不会被系统或其他程序的数据和代码使用

# 5.8 段前缀的使用

有一个问题：我们希望将内存 ffff:0 ~ ffff:b 单元中的数据复制到 0:200 ~ 0:20b 单元中

```
assume cs:code
code segment
	mov bx,0
	mov cx,12
 
s:  mov ax,0ffffh
	mov ds,ax
	mov dl,[bx]
 
	mov ax,0020h
	mov ds,ax
	mov [bx],dl

	inc bx
	loop s

	mov ax,4c00h
	int 21h
code ends
end
```

改进：
```
assume cs:code
code segment
	mov ax,0fffh
	mov ds,ax

	mov ax,0020h
	mov es,ax

	mov bx,0

	mov cx,12

s:  mov dl,[bx]
    mov es:[bx],dl
    inc bx
    loop s

	mov ax,4c00h
	int 21h
code ends
end
```

即我们可以用两个段寄存器来表示地址，从而简化程序，避免多次赋值

# 实验 4 \[bx] 和 loop 的使用
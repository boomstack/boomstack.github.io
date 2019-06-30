---

layout: post 

title: "从汇编角度看指针的意义" 

date: 2018-03-01 17:01:25 +0800 

categories: Android

---

如下MIPS汇编程序，新建一个数组，然后依次删除数组内容
```c
	.data
	.globl main
	.text
main:
	#mock data start
	li $a0, 268435456 #(1000 0000)16 静态数据区起始位置
	li $t1, 618
	sw $t1, 0($a0)
	addi $t1, $t1, 10
	sw $t1, 4($a0)
	addi $t1, $t1, 10
	sw $t1, 8($a0)
	li $a1, 3 #数组大小
	#mock data over
	
	#clear data
	add $t0, $zero, $a0 #t0即指针
	sll $t1, $a1, 2 #字节大小
	add $t2, $a0, $t1 #t2为数组最后一个位置
loop: 
	sw $zero, 0($t0)
	addi $t0, $t0, 4
	slt $t3, $t0, $t2
	beqz $t3, exit
	j loop
exit:
	li $v0, 1
	lw $a0, 8($a0)  #验证 输出0
	syscall
	
	li $v0, 10
	syscall
```
sw向内存写数据，lw从内存读数据；一开始以$a0为起始地址向内存连续写入3个数据（注：这里须从数据区开始sw，数据区从10000000（16进制）开始），随后将$a0赋值给$t0，于是$t0指向了数组开始地址$a0， $t0即为数组的指针（当然$a0也是，只不过这里将a0替换为t0，因为an寄存器一般存函数参数）。t0内容即内存中第10000000个存储单元编号，转为十进制为268435456，所以指针的本质意义即为存储单元的编号（中间可能有句柄管理，但是最终到底是指向存储单元）。

以下为内存结构的分配。另外，代码段被分配给Text区域，static data为全局静态变量，dynamic data实际上是堆，stack为栈，堆栈动态分配。从图中也可以发现一个现象，mips的J型跳转指令不能超过10000000即2的28次方，否则超出代码区 (图片来自《计算机组成与设计：硬件软件接口第五版》)。

!["dddd"](https://raw.githubusercontent.com/boomstack/boomstack.github.io/master/assets/all/20190407195131305.png)
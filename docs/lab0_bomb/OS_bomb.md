##OS-lab0-拆弹实验
<font size=2>牛天一519030910060 </font>

你需要通过阅读汇编代码以及使用调试工具来拆除一个二进制炸弹程序。本实验分为两个部分:第一部分介绍拆弹实验的基本知识,包括 ARM 汇编语言、QEMU 模拟器、GDB 调试器的使用;第二部分需要分析炸弹程序,推测正确的输入来使得炸弹程序能正常退出。实验链接：
https://github.com/SJTU-IPADS/OS-Course-Lab/tree/main

本次实验共6个phase，提供6次密码后可以解决问题。由于每人的密码是根据学号生成的，最后结果会有不同。其中前4次的代码可以在助教给出的指导视频中找到https://www.bilibili.com/video/BV1q94y1a7BF/?spm_id_from=333.337.search-card.all.click&vd_source=538fe4f145629b94b9fc7ac52322d384。

phase的代码,以phase_4为例：
```
input = read_line();
phase_4(input);
phase_defused();
```
反汇编之后得到的代码：
```
  4009e4:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
  4009e8:	910003fd 	mov	x29, sp
  4009ec:	a90153f3 	stp	x19, x20, [sp, #16]
  4009f0:	aa0003f3 	mov	x19, x0
  4009f4:	97fffe43 	bl	400300 <.plt+0x60>
  4009f8:	aa0003f4 	mov	x20, x0
  4009fc:	7100281f 	cmp	w0, #0xa
  400a00:	540001ec 	b.gt	400a3c <phase_4+0x58>
  400a04:	2a1403e1 	mov	w1, w20
  400a08:	aa1303e0 	mov	x0, x19
  400a0c:	97ffffaf 	bl	4008c8 <encrypt_method1>
  400a10:	2a1403e1 	mov	w1, w20
  400a14:	aa1303e0 	mov	x0, x19
  400a18:	97ffffd3 	bl	400964 <encrypt_method2>
  400a1c:	90000500 	adrp	x0, 4a0000 <_GLOBAL_OFFSET_TABLE_+0x470>
  400a20:	f9403401 	ldr	x1, [x0, #104]
  400a24:	aa1303e0 	mov	x0, x19
  400a28:	94008456 	bl	421b80 <strcmp>
  400a2c:	350000c0 	cbnz	w0, 400a44 <phase_4+0x60>
  400a30:	a94153f3 	ldp	x19, x20, [sp, #16]
  400a34:	a8c27bfd 	ldp	x29, x30, [sp], #32
  400a38:	d65f03c0 	ret
  400a3c:	9400002e 	bl	400af4 <explode>
  400a40:	17fffff1 	b	400a04 <phase_4+0x20>
  400a44:	9400002c 	bl	400af4 <explode>
  400a48:	17fffffa 	b	400a30 <phase_4+0x4c>
```

在phase_4中首先跳转到400300位置，这里经过gdb查看后发现是strlen函数，可以知道返回了字符串长度。而后进入encrypt_method1和encrypt_method2。最后经过与*(0x4a0000+104)所指字符串作比较，相同则通过。查看后发现目标字符串为"isggstsvkt"。
再查看encrypt_method1：
```
  4008c8:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
  4008cc:	910003fd 	mov	x29, sp
  4008d0:	910043e2 	add	x2, sp, #0x10
  4008d4:	3821c85f 	strb	wzr, [x2, w1, sxtw]
  4008d8:	0b417c23 	add	w3, w1, w1, lsr #31
  4008dc:	13017c63 	asr	w3, w3, #1
  4008e0:	7100043f 	cmp	w1, #0x1
  4008e4:	540003cd 	b.le	40095c <encrypt_method1+0x94>
  4008e8:	aa0203e4 	mov	x4, x2
  4008ec:	d2800002 	mov	x2, #0x0                   	// #0
  4008f0:	d37ff845 	lsl	x5, x2, #1
  4008f4:	38656805 	ldrb	w5, [x0, x5]
  4008f8:	38001485 	strb	w5, [x4], #1
  4008fc:	91000442 	add	x2, x2, #0x1
  400900:	6b02007f 	cmp	w3, w2
  400904:	54ffff6c 	b.gt	4008f0 <encrypt_method1+0x28>
  400908:	7100007f 	cmp	w3, #0x0
  40090c:	1a9fc465 	csinc	w5, w3, wzr, gt
  400910:	6b05003f 	cmp	w1, w5
  400914:	540001cd 	b.le	40094c <encrypt_method1+0x84>
  400918:	4b050024 	sub	w4, w1, w5
  40091c:	4b0300a2 	sub	w2, w5, w3
  400920:	8b22c402 	add	x2, x0, w2, sxtw #1
  400924:	91000442 	add	x2, x2, #0x1
  400928:	d2800001 	mov	x1, #0x0                   	// #0
  40092c:	910043e3 	add	x3, sp, #0x10
  400930:	8b25c065 	add	x5, x3, w5, sxtw
  400934:	d37ff823 	lsl	x3, x1, #1
  400938:	38636843 	ldrb	w3, [x2, x3]
  40093c:	382168a3 	strb	w3, [x5, x1]
  400940:	91000421 	add	x1, x1, #0x1
  400944:	eb04003f 	cmp	x1, x4
  400948:	54ffff61 	b.ne	400934 <encrypt_method1+0x6c>  // b.any
  40094c:	910043e1 	add	x1, sp, #0x10
  400950:	940084dc 	bl	421cc0 <strcpy>
  400954:	a8c27bfd 	ldp	x29, x30, [sp], #32
  400958:	d65f03c0 	ret
  40095c:	52800005 	mov	w5, #0x0                   	// #0
  400960:	17ffffec 	b	400910 <encrypt_method1+0x48>
```
查看反汇编后的代码，经过了两次循环，分别取了偶数位的字符和奇数位的字符重排，例如"ababababab".重排后为"aaaaabbbbb".
再查看encrypt_method2:
```
  400964:	7100003f 	cmp	w1, #0x0
  400968:	540003cd 	b.le	4009e0 <encrypt_method2+0x7c>
  40096c:	a9bd7bfd 	stp	x29, x30, [sp, #-48]!
  400970:	910003fd 	mov	x29, sp
  400974:	a90153f3 	stp	x19, x20, [sp, #16]
  400978:	a9025bf5 	stp	x21, x22, [sp, #32]
  40097c:	aa0003f3 	mov	x19, x0
  400980:	8b21c015 	add	x21, x0, w1, sxtw
  400984:	90000516 	adrp	x22, 4a0000 <_GLOBAL_OFFSET_TABLE_+0x470>
  400988:	910162d6 	add	x22, x22, #0x58
  40098c:	14000009 	b	4009b0 <encrypt_method2+0x4c>
  400990:	39400281 	ldrb	w1, [x20]
  400994:	f94006c0 	ldr	x0, [x22, #8]
  400998:	8b010000 	add	x0, x0, x1
  40099c:	3859f000 	ldurb	w0, [x0, #-97]
  4009a0:	39000280 	strb	w0, [x20]
  4009a4:	91000673 	add	x19, x19, #0x1
  4009a8:	eb15027f 	cmp	x19, x21
  4009ac:	54000120 	b.eq	4009d0 <encrypt_method2+0x6c>  // b.none
  4009b0:	aa1303f4 	mov	x20, x19
  4009b4:	39400260 	ldrb	w0, [x19]
  4009b8:	51018400 	sub	w0, w0, #0x61
  4009bc:	12001c00 	and	w0, w0, #0xff
  4009c0:	7100641f 	cmp	w0, #0x19
  4009c4:	54fffe69 	b.ls	400990 <encrypt_method2+0x2c>  // b.plast
  4009c8:	9400004b 	bl	400af4 <explode>
  4009cc:	17fffff1 	b	400990 <encrypt_method2+0x2c>
  4009d0:	a94153f3 	ldp	x19, x20, [sp, #16]
  4009d4:	a9425bf5 	ldp	x21, x22, [sp, #32]
  4009d8:	a8c37bfd 	ldp	x29, x30, [sp], #48
  4009dc:	d65f03c0 	ret
  4009e0:	d65f03c0 	ret
```
这里做的操作是先取出一段字符串(0x4a0058+8)，这段字符串是"qwertyuiopasdfghjklzxcvbnm"然后依次取method1得到的字符串，ldrb每次只取一个字符，该字符的ASCII码-97得到一个数，这个数就是在"qwertyuiopasdfghjklzxcvbnm"取到字符的位置。同样十次之后返回。
因此我们可以根据目标字符串"isggstsvkt"各个字符在"qwertyuiopasdfghjklzxcvbnm"中的位置反推我们需要输入的字符串。例如i位于第8位，则我们的第一个字符为104也就是h；而第二个s位于12位，则我们的第三个字符为108也就是l。最后可以得到结果helloworle。

下面来看phase_5:
```
  400ac0:	a9bf7bfd 	stp	x29, x30, [sp, #-16]!
  400ac4:	910003fd 	mov	x29, sp
  400ac8:	94000043 	bl	400bd4 <read_int>
  400acc:	90000501 	adrp	x1, 4a0000 <_GLOBAL_OFFSET_TABLE_+0x470>
  400ad0:	91016021 	add	x1, x1, #0x58
  400ad4:	91006021 	add	x1, x1, #0x18
  400ad8:	97ffffdd 	bl	400a4c <func_5>
  400adc:	71000c1f 	cmp	w0, #0x3
  400ae0:	54000061 	b.ne	400aec <phase_5+0x2c>  // b.any
  400ae4:	a8c17bfd 	ldp	x29, x30, [sp], #16
  400ae8:	d65f03c0 	ret
  400aec:	94000002 	bl	400af4 <explode>
  400af0:	17fffffd 	b	400ae4 <phase_5+0x24>
```
调用read_int读取了一个int数据，之后取全局变量再进入func_5.
func_5:
```
  400a4c:	b4000361 	cbz	x1, 400ab8 <func_5+0x6c>
  400a50:	a9be7bfd 	stp	x29, x30, [sp, #-32]!
  400a54:	910003fd 	mov	x29, sp
  400a58:	a90153f3 	stp	x19, x20, [sp, #16]
  400a5c:	2a0003f4 	mov	w20, w0
  400a60:	aa0103f3 	mov	x19, x1
  400a64:	b9400020 	ldr	w0, [x1]
  400a68:	6b14001f 	cmp	w0, w20
  400a6c:	54000160 	b.eq	400a98 <func_5+0x4c>  // b.none
  400a70:	b9400260 	ldr	w0, [x19]
  400a74:	6b14001f 	cmp	w0, w20
  400a78:	5400014d 	b.le	400aa0 <func_5+0x54>
  400a7c:	f9400661 	ldr	x1, [x19, #8]
  400a80:	2a1403e0 	mov	w0, w20
  400a84:	97fffff2 	bl	400a4c <func_5>
  400a88:	531f7800 	lsl	w0, w0, #1
  400a8c:	a94153f3 	ldp	x19, x20, [sp, #16]
  400a90:	a8c27bfd 	ldp	x29, x30, [sp], #32
  400a94:	d65f03c0 	ret
  400a98:	94000017 	bl	400af4 <explode>
  400a9c:	17fffff5 	b	400a70 <func_5+0x24>
  400aa0:	f9400a61 	ldr	x1, [x19, #16]
  400aa4:	2a1403e0 	mov	w0, w20
  400aa8:	97ffffe9 	bl	400a4c <func_5>
  400aac:	531f7800 	lsl	w0, w0, #1
  400ab0:	11000400 	add	w0, w0, #0x1
  400ab4:	17fffff6 	b	400a8c <func_5+0x40>
  400ab8:	52800000 	mov	w0, #0x0                   	// #0
  400abc:	d65f03c0 	ret
```
观察后可以看到这是一个带有条件的递归函数，如果read_int读入的int值不小于一开始的*（int *)(0x4a0070)=49的值就将0x4a0070+16再递归；如果小于就0x4a0070加8再递归。观察phase_5可以看到func_5的返回值应该为3，也就是经过递归后w0变为3.
假设一开始不小于49，则得到后移16后的指针为0x4a00a0，解引用后为88假设还是不小于，后移后为0x4a0100,解引用后为91，假设为小于则得到0x0.通过，故结果应为89或90.（相等时会直接触发爆炸）
这里之所以这样假设是观察到两种条件下对w0从0开始操作可以得到3的一种可能。故phase_5最后的密码是89或90








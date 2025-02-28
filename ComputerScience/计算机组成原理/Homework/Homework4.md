## 4.1
4.1.1 信号的值如下:

| 寄存器写入 | ALUsrc | ALU操作 | MemWrite | MemRead | MemToReg |
| ---------- | ------ | ------- | -------- | ------- | -------- |
| 真         | 0      | 与      | false    | false   | 0        |

4.1.2 寄存器、ALU src_mux、ALU 和 MemToReg_mux。
4.1.3 所有块都会产生一些输出。DataMemory 和 Imm Gen 的输出不被使用。

## 4.6

4.6.1 不需要添加

4.6.2

| Branch | MemRead | MemToReg | ALUop | MemWrite | ALUsrc | RegWrite |
| ------ | ------- | -------- | ----- | -------- | ------ | -------- |
| false  | false   | 0        | 10    | false    | 1      | 1        |

## 4.7

4.7.1 R-type: 30+250+150+25+200+25+20=700 ps
4.7.2 1d: 30+250+150+25+200+250+25+20=950 ps
4.7.3 sd: 30+250+150+200+25+250=905
4.7.4 beq: 30+250+150+25+200+5+25+20=705
4.7.5 I-type: 30+250+150+25+200+25+20=700 ps

4.7.6 950 ps

## 4.13

4.13.1 需要一些额外的多路复用器

4.13.2 不需要修改任何功能模块。

4.13.3 需要有一条路径从ALU输出到数据存储器的写数据端口。还需要有一条路径从读数据2直接到数据存储器的地址输入。

4.13.4 这些新的数据路径将需要由多路复用器驱动。这些多路复用器将需要控制线来选择。

## 4.19

x15 = 54

## 4.22

4.22.1 停顿用**标记：

| 1d 1d   | x29,8(x16)        |      | IF   | ID   | EX   | ME WB |      |      |      |          |
| ------- | ----------------- | ---- | ---- | ---- | ---- | ----- | ---- | ---- | ---- | -------- |
| sub sub | sub x17, x15, x14 |      |      | IF   | ID   | EX ME | WB   |      |      |          |
| bez     | x17, label        |      |      |      | **   | ** IF | ID   | EX   | ME   | WB       |
| add add | x15,x11,x14       |      |      |      |      |       | IF   | ID   | EX   | ME WB    |
| sub sub | sub x15,x30,x14   |      |      |      |      |       |      | IF   | ID   | EX ME WB |

4.22.2 重新排序代码不会有帮助。每条指令都必须被获取；因此，每次数据访问都会导致停顿。重新排序代码只会改变发生冲突的指令对。

4.22.3 你不能用NOP解决这个结构性危险，因为NOP也必须从指令存储器中获取。

4.22.4 35%。每次数据访问都会导致停顿。

## 4.27

4.27.1

```assembly
add	x15, x12, x11
nop 
nop
ld	x13,  4(x15)
ld	x12,  0(x2)
nop
or	x13, x15, x13
nop
nop
sd	x13, 0(x15)
```

4.27.2

无法减少NOP的数量

4.27.3 

代码执行正确。我们只需要进行危险检测，以在加载后面的指令使用加载结果时插入停顿。在这种情况下不会发生。

4.27.4 

| Cycle | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    |      |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| add   | IF   | ID   | EX   | ME   | WB   |      |      |      |      |
| ld    |      | IF   | ID   | EX   | ME   | WB   |      |      |      |
| ld    |      |      | IF   | ID   | EX   | ME   | WB   |      |      |
| or    |      |      |      | IF   | ID   | EX   | ME   | WB   |      |
| sd    |      |      |      |      | IF   | ID   | EX   | ME   | WB   |

由于这段代码中没有停顿，PCWrite 和 IF/IDWrite 总是 1，ID/EX 之前的多路复用器总是设置为传递控制值。 

(1) ForwardA = X; ForwardB = X（EX 阶段还没有指令） 

(2) ForwardA = X; ForwardB = X（EX 阶段还没有指令） 

(3) ForwardA = 0; ForwardB = 0（没有转发；从寄存器中获取值） 

(4) ForwardA = 2; ForwardB = 0（基寄存器从上一条指令的结果中获取） 

(5) ForwardA = 1; ForwardB = 1（基寄存器从前两条指令的结果中获取） 

(6) ForwardA = 0; ForwardB = 2（rsl = x15 从寄存器中获取；rs2 = x13 从前一条 ld 的结果中获取——两条指令之前） 

(7) ForwardA = 0; ForwardB = 2（基寄存器从寄存器文件中获取。要写入的数据从前一条指令中获取）

4.27.5 

危险检测单元还需要来自 MEM/WB 寄存器的 rd 值。\

4.27.6

| Cycle | 1    | 2    | 3    | 4    | 5    | 6    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| add   | IF   | ID   | EX   | ME   | WB   |      |
| ld    |      | IF   | ID   | -    | -    | Ex   |
| ld    |      |      | IF   | -    | -    | ID   |

(1) PCWrite = 1; IF/IDWrite = 1; control mux = 0 

(2) PCWrite = 1; IF/IDWrite = 1; control mux = 0 

(3) PCWrite = 1; IF/IDWrite = 1; control mux = 0 

(4) PCWrite = 0; IF/IDWrite = 0; control mux = 1 

(5) PCWrite = 0; IF/IDWrite = 0; control mux = 1

## 4.28

4.28.1  CPI由1增加到1.4215

4.28.2  1.3375

4.28.3  1.1125

4.28.4  性能提升为1.019

4.28.5  性能提升为91

4.28.6  为25%

---
title: '单周期CPU实验'
date: 2025-12-04
permalink: /posts/2025/12/single-cycle-CPU/
tags:
  - Laboratory_of _Computer_Organization
  - Verilog
---


# 实验目的

1\. 掌握单周期CPU数据通路图的构成、原理及其设计方法；

2\. 掌握单周期CPU的实现方法，代码实现方法；

3\. 认识和掌握指令与CPU的关系；

4\. 掌握测试单周期CPU的方法。

# 实验内容

设计一个单周期CPU，该CPU至少能实现以下指令功能操作。指令与格式如下：\
==\> 算术运算指令\
（1）add rd, rs, rt

::: tabular
\|c\|c\|c\|c\|c\|c\| &rs(5位)&rt(5位)&rd(5位)&00000&100000&
:::

[]{#add label="add"}

\
功能：GPR\[rd\] ← GPR\[rs\] + GPR\[rt\]。\
（2）sub rd, rs, rt

::: tabular
\|c\|c\|c\|c\|c\|c\| &rs(5位)&rt(5位)&rd(5位)&00000&100010&
:::

[]{#sub label="sub"}

\
功能：GPR\[rd\] ← GPR\[rs\] - GPR\[rt\]。\
（3）addiu rt, rs, immediate

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&immediate(16位)&
:::

[]{#addiu label="addiu"}

\
功能：GPR\[rt\] ← GPR\[rs\] + sign_extend(immediate)；
immediate做符号扩展再参加"与"运算。\
==\> 逻辑运算指令\
（4）andi rt, rs, immediate

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&immediate(16位)&
:::

[]{#addi label="addi"}

\
功能：GPR\[rt\] ← GPR\[rs\] and
zero_extend(immediate)；immediate做0扩展再参加"与"运算。\
（5）and rd, rs, rt

::: tabular
\|c\|c\|c\|c\|c\|c\| &rs(5位)&rt(5位)&rd(5位)&00000&100100&
:::

[]{#and label="and"}

\
功能：GPR\[rd\] ← GPR\[rs\] and GPR\[rt\]。 （6）ori rt, rs, immediate

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&immediate(16位)&
:::

[]{#ori label="ori"}

\
功能：GPR\[rt\] ← GPR\[rs\] or zero_extend(immediate)。\
（7）or rd, rs, rt

::: tabular
\|c\|c\|c\|c\|c\|c\| &rs(5位)&rt(5位)&rd(5位)&00000&100101&
:::

[]{#or label="or"}

\
功能：GPR\[rd\] ← GPR\[rs\] or GPR\[rt\]。\
==\>移位指令\
（8）sll rd, rt,sa

::: tabular
\|c\|c\|c\|c\|c\|c\| &00000&rt(5位)&rd(5位)&sa(5位)&000000&
:::

[]{#sll label="sll"}

\
功能：GPR\[rd\] ← GPR\[rt\] \<\< sa。\
\
==\>比较指令\
（9） slti rt, rs, immediate(带符号数)

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&immediate(16位)&
:::

[]{#slti label="slti"}

\
功能：if GPR\[rs\] \< sign_extend(immediate) GPR\[rt\] =1 else GPR\[rt\]
= 0。\
==\> 存储器读/写指令\
（10）sw rt, offset(rs(写存储器))

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&offset(16位)&
:::

[]{#sw label="sw"}

\
功能：memory\[GPR\[base\] + sign_extend(offset)\] ← GPR\[rt\]。\
（11） lw rt, offset(rs(读存储器))

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&offset(16位)&
:::

[]{#lw label="lw"}

\
功能：GPR\[rt\] ← memory\[GPR\[base\] + sign_extend(offset)\]。\
\
==\> 分支指令\
（12）beq rs,rt, offset

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&offset(16位)&
:::

[]{#beq label="beq"}

\
功能：if(GPR\[rs\] = GPR\[rt\]) pc←pc + 4 + sign_extend(offset)\<\<2
else pc ←pc + 4\
特别说明：offset是从PC+4地址开始和转移到的指令之间指令条数。offset符号扩展之后左移2位再相加。为什么要左移2位？由于跳转到的指令地址肯定是4的倍数（每条指令占4个字节），最低两位是"00"，因此将offset放进指令码中的时候，是右移了2位的，也就是以上说的"指令之间指令条数"。\
（13）bne rs, rt, offset

::: tabular
\|c\|c\|c\|c\| &rs(5位)&rt(5位)&offset(16位)&
:::

[]{#bne label="bne"}

\
功能：if(GPR\[rs\] != GPR\[rt\]) pc←pc + 4 + sign_extend(offset) \<\<2
else pc ←pc + 4 （14）blez rs, offset

::: tabular
\|c\|c\|c\|c\| &rs(5位)&00000&offset(16位)&
:::

[]{#blez label="blez"}

\
功能：if(GPR\[rs\] ≤ 0) pc←pc + 4 + sign_extend (offset) \<\<2 else pc
←pc + 4。\
\
==\>跳转指令\
（15）j addr

::: tabular
\|c\|c\| &addr(26位)&
:::

[]{#bne label="bne"}

\
功能：PC ← PC\[31:28\] , addr , 2'b0，无条件跳转。\
说明：由于MIPS32的指令代码长度占4个字节，所以指令地址二进制数最低2位均为0，将指令地址放进指令代码中时，可省掉！这样，除了最高6位操作码外，还有26位可用于存放地址，事实上，可存放28位地址，剩下最高4位由pc+4最高4位拼接上。\
==\> 停机指令\
（16）halt

::: tabular
\|c\|c\| &00000000000000000000000000(26位)&
:::

[]{#halt label="halt"}

\
功能：停机；不改变PC的值，PC保持不变。

# 实验原理

## MIPS指令下的单周期CPU

单周期CPU指的是一条指令的执行在一个时钟周期内完成，然后开始下一条指令的执行，即一条指令用一个时钟周期完成。电平从低到高变化的瞬间称为时钟上升沿，两个相邻时钟上升沿之间的时间间隔称为一个时钟周期。时钟周期一般也称振荡周期（如果晶振的输出没有经过分频就直接作为CPU的工作时钟，则时钟周期就等于振荡周期。若振荡周期经二分频后形成时钟脉冲信号作为CPU的工作时钟，这样，时钟周期就是振荡周期的两倍。）\
CPU在处理指令时，一般需要经过以下几个步骤：

\(1\)
取指令(IF)：根据程序计数器PC中的指令地址，从存储器中取出一条指令，同时，PC根据指令字长度自动递增产生下一条指令所需要的指令地址，但遇到"地址转移"指令时，则控制器把"转移地址"送入PC，当然得到的"地址"需要做些变换才送入PC。

\(2\)
指令译码(ID)：对取指令操作中得到的指令进行分析并译码，确定这条指令需要完成的操作，从而产生相应的操作控制信号，用于驱动执行状态中的各种操作。

\(3\)
指令执行(EXE)：根据指令译码得到的操作控制信号，具体地执行指令动作，然后转移到结果写回状态。

\(4\)
存储器访问(MEM)：所有需要访问存储器的操作都将在这个步骤中执行，该步骤给出存储器的数据地址，把数据写入到存储器中数据地址所指定的存储单元或者从存储器中得到数据地址单元中的数据。

\(5\)
结果写回(WB)：指令执行的结果或者访问存储器中得到的数据写回相应的目的寄存器中。

单周期CPU，是在一个时钟周期内完成这五个阶段的处理，流程如图[1](#CPU phase){reference-type="ref"
reference="CPU phase"}所示。

![CPU 处理指令周期](figures/CPU处理阶段.png){#CPU phase
width="1\\linewidth"}

MIPS指令有以下3种格式：

![MIPS指令格式](figures/三种指令机器码格式.png){#MIPS instruction
width="0.9\\linewidth"}

\
其中:

op：为操作码；

rs：只读。为第1个源操作数寄存器，寄存器地址（编号）是00000 11111，00 1F；

rt：可读可写。为第2个源操作数寄存器，或目的操作数寄存器，寄存器地址（同上）；

rd：只写。为目的操作数寄存器，寄存器地址（同上）；

sa：为位移量（shift amt），移位指令用于指定移多少位；

funct：为功能码，在寄存器类型指令中（R类型）用来指定指令的功能与操作码配合使用；

immediate：为16位立即数，用作无符号的逻辑操作数、有符号的算术操作数、数据加载（Laod）/数据保存（Store）指令的数据地址字节偏移量和分支指令中相对程序计数器（PC）的有符号偏移量；

address：为地址。

如图[3](#single cycle CPU){reference-type="ref"
reference="single cycle CPU"}是一个简单的基本上能够在单周期CPU上完成所要求设计的指令功能的数据通路和必要的控制线路图。其中指令和数据各存储在不同存储器中，即有指令存储器和数据存储器。访问存储器时，先给出内存地址，然后由读或写信号控制操作。对于寄存器组，先给出寄存器地址，读操作时不需要时钟信号，输出端就直接输出相应数据；而在写操作时，在
WE使能信号为1时，在时钟边沿触发将数据写入寄存器。如图[4](#control signal){reference-type="ref"
reference="control signal"}所示表格是Control
Unit输出的各类控制信号含义。

![单周期CPU数据通路](figures/CPU数据通路.png){#single cycle CPU
width="0.8\\linewidth"}

![控制信号表](figures/控制信号表.png){#control signal
width="1\\linewidth"}

指令执行的结果总是在时钟下降沿保存到寄存器和存储器中，PC的改变是在时钟上升沿进行的，这样稳定性较好。另外，值得注意的问题，设计时，用模块化的思想方法设计，关于ALU设计、存储器设计、寄存器组设计等等，也是必须认真考虑的问题。

## 模块设计

我们将采用模块化的思想，将单周期CPU的各个部件分开实现。具体情况如下：

### PC计数器

PC计数器用于给出下一条指令。PC计数器受时钟周期上升沿触发，受PC写使能信号控制，当PCWre为1时改变当前指令地址。同时也要基于Reset信号
的下降沿将指令地址重置为0。

![pc计数器](figures/pc计数器.png){#programe counter
width="0.2\\linewidth"}

pc计数器代码如下：

``` {#programe\\_counter .verilog language="Verilog" label="programe\\_counter"}
module programe_counter(
    input clk,                  // 时钟信号
    input reset,                // 清零置位
    input PCWre,                // 是否更改PC
    input [31:0] new_addr,       // 新指令地址
    output reg [31:0] PC         // 下一个指令地址
);
    
    always @(posedge clk or negedge reset) begin
        if (!reset)
            // 清零
            PC <= 32'b0;
        else if (PCWre)
            // 加载新地址
            PC <= new_addr;
    end
    
endmodule
```

### 指令存储器

**Instruction Memory：指令存储器**

Iaddr，指令存储器地址输入端口

IDataIn，指令存储器数据输入端口（指令代码输入端口）

IDataOut，指令存储器数据输出端口（指令代码输出端口）

RW，指令存储器读写控制信号，为0写，为1读

![指令存储器](figures/指令存储器.png){#Instruction Memory
width="0.4\\linewidth"}

该部件实现代码如下：

``` {#instruction\\_memory .verilog language="Verilog" label="instruction\\_memory"}
module InsMem(
    input [31:0] Iaddr,              // 指令地址
    input InsMemRW,                  // 读写模式，1为读
    input [31:0] IDataIn,            // 写入指令
    output reg [31:0] IDataOut       // 输出指令
);
    reg [7:0] ins_file [255:0];     // 256个8位指令存储器
    
    initial begin
    ins_file[0]  <= 8'h24; // addiu $1, $0, 8
    ins_file[1]  <= 8'h01;
    ins_file[2]  <= 8'h00;
    ins_file[3]  <= 8'h08;
    ......
    省略后续指令
    ......
    
    end
    
    always @(Iaddr or InsMemRW) begin
        if(!InsMemRW) begin
            // 写指令
            ins_file[Iaddr[7:0]] <= IDataIn[31:24];
            ins_file[Iaddr[7:0]+1] <= IDataIn[23:16];
            ins_file[Iaddr[7:0]+2] <= IDataIn[15:8];
            ins_file[Iaddr[7:0]+3] <= IDataIn[7:0];
        end else begin
            // 读指令
            IDataOut[31:24] <= ins_file[Iaddr[7:0]];
            IDataOut[23:16] <= ins_file[Iaddr[7:0]+1];
            IDataOut[15:8] <= ins_file[Iaddr[7:0]+2];
            IDataOut[7:0] <= ins_file[Iaddr[7:0]+3];
        end
    end
endmodule
```

### 数据存储器

**Data Memory：数据存储器**

Daddr，数据存储器地址输入端口

DataIn，数据存储器数据输入端口

DataOut，数据存储器数据输出端口

RD，数据存储器读控制信号，为0读

WR，数据存储器写控制信号，为0写

![数据存储器](figures/数据存储器.png){#Data Memory
width="0.2\\linewidth"}

该部件实现代码如下：

``` {#data\\_memory .verilog language="Verilog" label="data\\_memory"}
module DataMem(
    input clk,                      // 时钟信号
    input [31:0]Daddr,               // 输入地址
    input [31:0]DataIn,             // 输入数据
    input mRD,                      // 读模式
    input mWR,                      // 写模式
    output reg [31:0]DataOut        // 输出数据
);
    reg [7:0] data_file [255:0];   // 256个8位数据存储器
    integer i;
    initial begin
        for(i=0;i<256;i=i+1)
            data_file[i]<=0;
            DataOut = 32'b0;
    end
    
    always @(Daddr or mRD) begin    // 读数据
        if (mRD) begin
            // 大端储存
            DataOut[31:24] = data_file[Daddr];
            DataOut[23:16] = data_file[Daddr+1];
            DataOut[15:8] = data_file[Daddr+2];
            DataOut[7:0] = data_file[Daddr+3];
        end
    end
    
    always @(negedge clk) begin    // 写数据
        if (mWR) begin
            data_file[Daddr] <= DataIn[31:24];
            data_file[Daddr+1] <= DataIn[23:16];
            data_file[Daddr+2] <= DataIn[15:8];
            data_file[Daddr+3] <= DataIn[7:0];
        end
    end
endmodule
```

### 寄存器堆

**Register File：寄存器组**

Read Reg1，rs寄存器地址输入端口

Read Reg2，rt寄存器地址输入端口

Write Reg，将数据写入的寄存器端口，其地址来源rt或rd字段

Write Data，写入寄存器的数据输入端口

Read Data1，rs寄存器数据输出端口

Read Data2，rt寄存器数据输出端口

WE，写使能信号，为1时，在时钟边沿触发写入

![寄存器堆](figures/寄存器堆.png){#Register width="0.3\\linewidth"}

寄存器堆实现代码如下：

``` {#registers .verilog language="Verilog" label="registers"}
module register(
    input clk,                       // 时钟信号
    input WE,                        // 操作模式，0为读，1为写
    input [4:0] Read_Reg1,           // 输入地址1
    input [4:0] Read_Reg2,           // 输入地址2
    input [4:0] Write_Reg,           // 写入地址
    input [31:0] Write_Data,         // 写入数据
    output reg [31:0] Read_Data1,        // 输出数据1
    output reg [31:0] Read_Data2         // 输出数据2
    );

    reg [31:0] reg_file [256:0];     // 256个32位寄存器
    integer i;
    initial begin
        for (i = 0; i < 32; i = i+1)
            reg_file[i] <= 32'b0;
    end
    
    always@(*) begin
        Read_Data1 <= Read_Reg1?reg_file[Read_Reg1]:32'b0;
        Read_Data2 <= Read_Reg2?reg_file[Read_Reg2]:32'b0;
    end
    
    always@(posedge clk or Write_Data) begin
            if(WE) begin
            reg_file[{3'b0,Write_Reg}] = Write_Data;
            end
    end
endmodule
```

### 算数逻辑单元

**ALU：算术逻辑单元**

result，ALU运算结果

zero，运算结果标志，结果为0，则zero=1；否则zero=0

sign，运算结果标志，结果最高位为0，则sign=0，正数；否则，sign=1，负数

![算术逻辑单元](figures/ALU.png){#alu width="0.3\\linewidth"}

![image](figures/ALU指令.png){width="0.9\\linewidth"} []{#alu_signal
label="alu_signal"}

根据我们要实现的指令确定alu的功能指令如图[\[alu_signal\]](#alu_signal){reference-type="ref"
reference="alu_signal"}所示,算数逻辑单元实现代码如下：

``` {#alu .verilog language="Verilog" label="alu"}
module alu (
    input clk,
    input [31:0] input1,      // 输入1
    input [31:0] input2,      // 输入2
    input [2:0] mode,        // 运算模式
    output reg SF,               // 符号标志
    output reg ZF,               // 零标志
    output reg [31:0] result  // 结果
);

    always @(negedge clk) begin
        // 根据 mode 选择操作数
        case (mode)
            3'b000: begin
                // 加法
                result = input1 + input2;
            end
            3'b001: begin
                // 减法
                result = input1 - input2;
            end
            3'b010: begin
                // 左移
                result = input2 << input1;
            end
            3'b011: begin
                // OR 操作
                result = input1 | input2;
            end
            3'b100: begin
                // AND 操作
                result = input1 & input2;
            end
            3'b101: begin
                // 小于比较
                result = (input1 < input2) ? 32'b1 : 32'b0;
            end
            3'b110: begin
                // 有符号小于比较
                result = (((input1 < input2) && (input1[31] == input2[31])) 
                    || (input1[31] == 1 && input2[31] == 0)) ? 32'b1 : 32'b0;
            end
            3'b111: begin
                // 异或操作
                result = input1 ^ input2;
            end
            default: begin
                result = 32'b0;
            end
        endcase
        SF <= result[31] == 1;
        ZF <= (result == 32'b0);
    end
endmodule
```

### 控制单元

控制单元根据指令进行信号输出控制整个单周期CPU的运作。其信号输出对应如图[4](#control signal){reference-type="ref"
reference="control signal"}所示（在Page6）。

![控制单元](figures/控制单元.png){#Control unit width="0.5\\linewidth"}

控制单元实现代码如下：

``` {#control\\_unit .verilog language="Verilog" label="control\\_unit"}
module control_unit(
    input clk,
    input [5:0] opcode,       // 操作码
    input [5:0] funct,        // 功能码
    input zero,               // 零标志位
    input sign,               // 符号标志位
    output reg PCWre,         // PC 写使能信号
    output reg [1:0] PCSrc,   // PC 源选择信号
    output reg ExtSel,        // 扩展选择信号
    output reg RegDst,        // 寄存器写入目标选择信号
    output reg RegWre,        // 寄存器写使能信号
    output reg [2:0] ALUop,   // ALU 操作码
    output reg ALUSrcA,       // ALU 源 A 选择信号
    output reg ALUSrcB,       // ALU 源 B 选择信号
    output reg mRD,           // 数据存储器读使能信号
    output reg mWR,           // 数据存储器写使能信号
    output reg DBDataSrc      // 寄存器写源选择信号
    );
    
    always @(posedge clk)begin
        PCWre = 1;
        PCSrc = 2'b00;
        RegWre = 0;
        ExtSel = 0;
        DBDataSrc = 0;
        RegDst = 0;
        ALUSrcA = 0;
        ALUSrcB = 0;
        PCSrc = 2'b00;
        mRD = 0;
        mWR = 0;
        ALUop = 3'b111;
        case (opcode)
            6'b000000: begin        // R型指令
                RegDst = 1;
                RegWre = 1;
                case (funct)
                    6'b100000: begin    // 加法
                        ALUop <= 3'b000;
                    end
                    6'b100010: begin    // 减法
                        ALUop <= 3'b001;
                    end
                    6'b100100: begin    // and运算
                        ALUop <= 3'b100;
                    end
                    6'b100101: begin    // or运算
                        ALUop <= 3'b011;
                    end
                    6'b000000: begin    // 左移sll运算
                        ALUop <= 3'b010;
                        ALUSrcA = 1;
                    end
                endcase
            end
            6'b001001: begin        // addiu
                ALUSrcB = 1;
                ExtSel = 1;
                RegWre = 1;
                ALUop <= 3'b000;
            end
            6'b001100: begin        // andi
                ALUSrcB = 1;
                RegWre = 1;
                ALUop <= 3'b100;
            end
            6'b001101: begin        // ori
                ALUSrcB = 1;
                RegWre = 1;
                ALUop <= 3'b011;
            end
            6'b001010: begin        // slti
                ALUSrcB = 1;
                RegWre = 1;
                ExtSel = 1;
                ALUop <= 3'b110;
            end
            6'b101011: begin        // sw
                ALUSrcB = 1;
                ExtSel = 1;
                mWR = 1;
            end
            6'b100011: begin        // lw
                ALUSrcB = 1;
                ExtSel = 1;
                mRD = 1;
                RegWre = 1;
                DBDataSrc = 1;
            end
            6'b000100: begin        // beq
                ALUop = 3'b001;
                ExtSel = 1;
                PCSrc[0] = (zero == 1);
            end
            6'b000101: begin        // bne
                ALUop = 3'b001;
                ExtSel = 1;
                PCSrc[0] = (zero == 0);
            end
            6'b000110: begin        // blez
                PCSrc[0] = (sign == 1) || (zero == 1);
                ExtSel = 1;
                ALUop = 3'b001;
            end
            6'b000010: begin        // jump
                PCSrc[1] = 1;
            end
            6'b111111: PCWre = 0;   // halt
            default: PCWre = 0;
        endcase
    end
endmodule
```

### 扩展部件及选择部件

将16位数据扩展为32位的Extension代码如下：

``` {#extension .verilog language="Verilog" label="extension"}
module Extension(
    input ExtSel, // 0做0扩展，1做符号扩展
    input [15:0]immediate,
    output reg [31:0] res
    );
    
    always @(*) begin
        if(!ExtSel) begin
            res <= {16'b0,immediate};
        end else begin
            res = {immediate[15] == 1 ? 16'hffff : 16'h0000, immediate};
        end
    end
endmodule
```

选择部件分别有5位2选1，32位2选1，32位4选1三种情况。\
5位2选1实现代码如下：

``` {#mux_5bits .verilog language="Verilog" label="mux_5bits"}
module mux_5bits(
    input [4:0] A,             // 输入A
    input [4:0] B,             // 输入B
    input Src,                 // 输入选择信号
    output reg [4:0] out       // 输出
    );
    
    always @(*) begin
        if(!Src)
            out <= A;
        else
            out <= B;
    end
endmodule
```

\
32位2选1实现代码如下：

``` {#mux_32bits .verilog language="Verilog" label="mux_32bits"}
module mux_32bits(
    input [31:0] A,             // 输入A
    input [31:0] B,             // 输入B
    input Src,                 // 输入选择信号
    output reg [31:0] out       // 输出
    );
    
    always @(*) begin
        if(!Src)
            out <= A;
        else
            out <= B;
    end
endmodule
```

\
32位4选1实现代码如下：

``` {#mux_32bits_4in1 .verilog language="Verilog" label="mux_32bits_4in1"}
module mux_32bits_4in1(
    input [31:0] A,             // 输入A
    input [31:0] B,             // 输入B
    input [31:0] C,             // 输入C
    input [31:0] D,             // 输入D
    input [1:0] Src,            // 输入选择信号
    output reg [31:0] out       // 输出
    );
    
    always @(*) begin
        case (Src)
            2'b00: out <= A;
            2'b01: out <= B;
            2'b10: out <= C;
            2'b11: out <= D;
        endcase
    end
endmodule
```

### 顶层代码

通过single_cycle_CPU的顶层代码将以上部件串联在一起，使之可以正常发挥作用。其代码如下：

``` {#single\\_cycle\\_CPU .verilog language="Verilog" label="single\\_cycle\\_CPU"}
module single_cycle_CPU(
    input clk,
    input data_clk,
    input reset,
    input InsMemRW,
    input [31:0] IDataIn,
    output [31:0] PC,
    output [31:0] new_PC,
    output [31:0] instruction,
    output [31:0] rs_data,
    output [31:0] rt_data,
    output [31:0] ALU_reg,
    output [31:0] write_data
    );
      
      wire [4:0] write_reg;
      wire [31:0] DataOut;
      wire [31:0] Ext_Immediate;
      wire PCWre;
      wire RegWre;
      wire ExtSel;
      wire DBDataSrc;
      wire RegDst;
      wire ALUSrcA;
      wire ALUSrcB;
      wire mRD;
      wire mWR;
      wire [2:0] ALUop; 
      wire [1:0] PCSrc;
      wire zero;
      wire sign;
      wire [31:0] ALU_input1;
      wire [31:0] ALU_input2;
      wire [31:0] next_PC = PC + 4;
      
      mux_32bits_4in1 MUX32_4in1_inst(
        .A(PC+4),
        .B(PC+(Ext_Immediate<<2)+4),
        .C({PC[31:28],instruction[25:0],2'b00}),
        .D(32'b0),
        .Src(PCSrc),
        .out(new_PC)
      );
      
      programe_counter PC_inst(
        .clk(data_clk),
        .PCWre(PCWre),
        .reset(reset),
        .new_addr(new_PC),
        .PC(PC)
      );
      
      InsMem IM_ins(
        .Iaddr(PC),
        .InsMemRW(InsMemRW),
        .IDataOut(instruction),
        .IDataIn(IDataIn)
      );
      
      Extension Ext_inst(
        .ExtSel(ExtSel),
        .immediate(instruction[15:0]),
        .res(Ext_Immediate)
      );
      
      mux_5bits MUX5_inst(
        .A(instruction[20:16]),
        .B(instruction[15:11]),
        .Src(RegDst),
        .out(write_reg)
      );
      
      register register_inst(
        .clk(data_clk),
        .WE(RegWre),
        .Read_Reg1(instruction[25:21]),
        .Read_Reg2(instruction[20:16]),
        .Write_Reg(write_reg),
        .Write_Data(write_data),
        .Read_Data1(rs_data),
        .Read_Data2(rt_data)
      );
      
      mux_32bits MUX32A_inst(
        .A(rs_data),
        .B({27'b0, instruction[10:6]}),
        .Src(ALUSrcA),
        .out(ALU_input1)
      );
      
      mux_32bits MUX32B_inst(
        .A(rt_data),
        .B(Ext_Immediate),
        .Src(ALUSrcB),
        .out(ALU_input2)
      );
      
      alu ALU_ins(
        .clk(clk),
        .input1(ALU_input1),
        .input2(ALU_input2),
        .result(ALU_reg),
        .mode(ALUop),
        .ZF(zero),
        .SF(sign)
      );
      
      DataMem DM_ins(
        .clk(clk),
        .Daddr(ALU_reg),
        .DataIn(rt_data),
        .mRD(mRD),
        .mWR(mWR),
        .DataOut(DataOut)
      );
      
      mux_32bits MUX32_inst(
        .A(ALU_reg),
        .B(DataOut),
        .Src(DBDataSrc),
        .out(write_data)
      );
      
      control_unit control_inst(
        .clk(clk),
        .opcode(instruction[31:26]),
        .funct(instruction[5:0]),
        .zero(zero),
        .sign(sign),
        .PCWre(PCWre),
        .PCSrc(PCSrc),
        .ExtSel(ExtSel),
        .RegDst(RegDst),
        .RegWre(RegWre),
        .ALUop(ALUop),
        .ALUSrcA(ALUSrcA),
        .ALUSrcB(ALUSrcB),
        .mRD(mRD),
        .mWR(mWR),
        .DBDataSrc(DBDataSrc)
      );
      
endmodule
```

### 烧板

烧板时需将单周期CPU与按键防抖器、数码管和时钟分频器接在一起，使数码管按照烧板要求显示内容验证单周期CPU的逻辑正确。\
按键防抖器代码如下：

``` {#clock\\_div .verilog language="Verilog" label="clock\\_div"}
module debounce(
    input clk,
    input button,
    output reg debounced_button
    );
    
    reg [3:0] count;
    always @(posedge clk)begin
        if(button == debounced_button)begin
            count <= 0;
        end else begin
            if (count < 18'd20000) begin  //烧板用
          //  if (count < 10'd1) begin     //仿真用
                count <= count + 1;            
            end else begin
                debounced_button <= button;
                count <= 0;
            end
        end
    end
endmodule
```

数码管代码如下：

``` {#seg7\\_LED .verilog language="Verilog" label="seg7\\_LED"}
module seg7_LED(
    input clk,           // 时钟输入
    input data_clk,      // data输入时钟
    input rst_n,         // 复位信号
    input [15:0] display_data,  // 输入要显示的4位16进制数字，每4位对应一位数码管
    output reg [7:0] dispcode,  // 数码管段选信号
    output reg [3:0] sel       // 数码管位选信号
    );
    
    reg [1:0] scan_cnt;         // 扫描计数器，用于选择数码管位
    reg [3:0] num;              // 当前数码管显示的数值

    // 扫描计数器+位选信号+段选信号
    always @(posedge clk) begin
        if(scan_cnt < 4)begin
            scan_cnt <= scan_cnt + 1;
        end else begin
            scan_cnt <= 2'd0;
        end
        case(scan_cnt[1:0])
            2'b00:begin
                sel = 4'b1110;
                num <= display_data[7:4];
            end
            2'b01:begin
                sel = 4'b1101;
                num <= display_data[11:8];
            end
            2'b10:begin
                sel = 4'b1011;
                num <= display_data[15:12];
            end
            2'b11:begin
                sel = 4'b0111;
                num <= display_data[3:0];
            end
            default: begin
                sel = 4'b1111;
            end
        endcase
        case (num)
                4'b0000: dispcode <= 8'b1100_0000; // 显示0
                4'b0001: dispcode <= 8'b1111_1001; // 显示1
                4'b0010: dispcode <= 8'b1010_0100; // 显示2
                4'b0011: dispcode <= 8'b1011_0000; // 显示3
                4'b0100: dispcode <= 8'b1001_1001; // 显示4
                4'b0101: dispcode <= 8'b1001_0010; // 显示5
                4'b0110: dispcode <= 8'b1000_0010; // 显示6
                4'b0111: dispcode <= 8'b1101_1000; // 显示7
                4'b1000: dispcode <= 8'b1000_0000; // 显示8
                4'b1001: dispcode <= 8'b1001_0000; // 显示9
                4'b1010: dispcode <= 8'b1000_1000; // 显示A
                4'b1011: dispcode <= 8'b1000_0011; // 显示B
                4'b1100: dispcode <= 8'b1100_0110; // 显示C
                4'b1101: dispcode <= 8'b1010_0001; // 显示D
                4'b1110: dispcode <= 8'b1000_0110; // 显示E
                4'b1111: dispcode <= 8'b1000_1110; // 显示F
                default: dispcode <= 8'b0000_0000; // 默认
        endcase
    end
endmodule
```

时钟分频器代码如下：

``` {#clock\\_div .verilog language="Verilog" label="clock\\_div"}
module clock_div(
    input clk,
    output reg clk_sys = 0
    );
    
    reg[25:0] div_counter = 0;
    
    always@(posedge clk) begin
//     if(div_counter >= 5) begin          // 用于仿真
       if(div_counter >= 50000) begin     // 用于芯片
            clk_sys <= ~clk_sys;         //电平反向
            div_counter <= 0;
        end else begin
            div_counter <= div_counter + 1;
        end
        $display("div_counter:",div_counter," clk_sys: ",clk_sys);
    end
endmodule
```

烧板时要求如下：

开关SW_in
(SW15、SW14)状态情况如下。显示格式：左边两位数码管BB:右边两位数码管BB。以下是数码管的显示内容。\
SW_in = 00：显示 当前 PC值:下条指令PC值\
SW_in = 01：显示 RS寄存器地址:RS寄存器数据\
SW_in = 10：显示 RT寄存器地址:RT寄存器数据\
SW_in = 11：显示 ALU结果输出 :DB总线数据。\
复位信号（reset）接开关SW0，按键（单脉冲）接按键BTNR。

烧板顶层代码如下：

``` {#Basys3 .verilog language="Verilog" label="Basys3"}
module Basys3(
    input clk,
    input button,
    input reset,
    input [1:0] SW,
    input InsMemRW,
    output [7:0] dispcode,     // 数码管段选信号
    output [3:0] sel           // 数码管位选信号
    );
    
    wire clk_sys;
    wire data_clk;
    reg [15:0] display_data;       // 输入要显示的4位16进制数字，每4位对应一位数码管
    wire [31:0] IDataIn;
    wire [31:0] PC;
    wire [31:0] new_PC;
    wire [31:0] instruction;
    wire [31:0] rs_data;
    wire [31:0] rt_data;
    wire [31:0] ALU_reg;
    wire [31:0] write_data;
    
    // 实例化烧板模块
    single_cycle_CPU CPU(
        .clk(clk_sys),
        .data_clk(data_clk),
        .reset(reset),
        .IDataIn(32'b0),
        .InsMemRW(1'b1),
        .PC(PC),
        .new_PC(new_PC),
        .instruction(instruction),
        .rs_data(rs_data),
        .rt_data(rt_data),
        .ALU_reg(ALU_reg),
        .write_data(write_data)
    );
    
    debounce debounce_inst(
        .clk(clk_sys),
        .button(button),
        .debounced_button(data_clk)
    );
    
    clock_div clock_div_inst(
        .clk(clk),
        .clk_sys(clk_sys)
    );
    
    seg7_LED LED_inst(
        .clk(clk_sys),
        .display_data(display_data),
        .dispcode(dispcode),
        .sel(sel)
    );
    
    // 根据SW确定数码管显示内容
    always @(posedge clk_sys) begin
        if(!reset)
            display_data <= 16'b0;
        else case (SW)
            2'b00: begin
                display_data[15:8] <= PC[7:0];
                display_data[7:0] <= new_PC[7:0];
            end
            2'b01: begin
                display_data[15:8] <= {3'b0, instruction[25:21]};
                display_data[7:0] <= rs_data[7:0];
            end
            2'b10: begin
                display_data[15:8] <= {3'b0, instruction[20:16]};
                display_data[7:0] <= rt_data[7:0];
            end
            2'b11: begin
                display_data[15:8] <= ALU_reg[7:0];
                display_data[7:0] <= write_data[7:0];
            end
            default: begin
                display_data <= 16'hFFFF;
            end
        endcase
    end
endmodule
```

# 实验器材

电脑一台， Xilinx Vivado 软件一套， Basys3板一块。

# 实验过程与结果

## 仿真实验

我们将以下指令存入指令存储器并通过以下指令检验所设计的单周期CPU是否按照预期流程进行。

![image](figures/CPU测试.png){width="1\\linewidth"} []{#CPUtest
label="CPUtest"}

仿真结果如图[11](#仿真1){reference-type="ref"
reference="仿真1"}、[12](#仿真2){reference-type="ref"
reference="仿真2"}、[13](#仿真3){reference-type="ref"
reference="仿真3"}和[14](#仿真4){reference-type="ref"
reference="仿真4"}所示。

![仿真1](figures/仿真1.png){#仿真1 width="0.8\\linewidth"}

图[11](#仿真1){reference-type="ref"
reference="仿真1"}是从指令地址0x00000000至0x00000018的运行过程，其中programe
counter始终每周期自增4。这个过程后寄存器\$1=8, \$2=2, \$3=10, \$4=0,
\$5=8, \$8=2。

![仿真2](figures/仿真2.png){#仿真2 width="0.8\\linewidth"}

图[12](#仿真2){reference-type="ref"
reference="仿真2"}是从指令0x00000018至0x0000002c的运行过程，其中第一次到0x0000001c时，由于寄存器\$8=4\<8，故返回0x00000018，再次左移一位后\$8=8
==
8，指令计数器自增。之后顺序执行到指令0x00000028，此时寄存器\$6=1,\$7=8,
\$8=8。

![仿真3](figures/仿真3.png){#仿真3 width="0.8\\linewidth"}

图[13](#仿真3){reference-type="ref"
reference="仿真3"}是从指令0x0000002c至0x00000040的运行过程，其中0x0000002c至0x0000003c是顺序执行，第一次到0x0000003c时寄存器\$9=0,
\$10=-1。之后执行0x00000040的blez指令，-1≤0，故跳转执行0x0000003c，寄存器\$10=0。顺序执行0x00000040，仍然有\$10=0≤0，再次跳转执行0x0000003c，寄存器\$10=1。顺序执行0x00000040后，\$10=1\>0，故顺序执行0x00000044，使\$11=2。

![仿真4](figures/仿真4.png){#仿真4 width="0.8\\linewidth"}

图[14](#仿真4){reference-type="ref"
reference="仿真4"}是从指令0x00000040运行至0x00000050的过程。部分在上段已经分析，0x00000048时运行jump指令直接跳转0x00000050，在0x000050处运行halt指令，不再向下执行。

## 实验结果

将以上项目进行引脚分配后烧录到Basys3板上。引脚设置时仍旧记得将I/O
std改为LVCMO33，sel作为位选信号sel\[0\]-sel\[3\]连U2,U4,V4和W4，dispcode作为段选信号dispcode\[0\]-dispcode\[7\]分别连W7,W6,U8,V8,U5,V5,U7和V7；SW\[0\]和SW\[1\]分别连T1和R2，clk连W5，button连T17生成data_clk信号，reset连V17。
最后通过拨弄SW0和SW1设置数码管输出数以及按T17对应按钮产生data_clk信号进行测试。

烧板结果如预期一致。

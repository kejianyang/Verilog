# Verilog学习

## 语法篇

### 一、基础语法

#### 1逻辑值

逻辑0 ：低电平

逻辑1 ：高电平

逻辑X：未知 有可能是0 也有可能是1

 逻辑Z：高阻态 外部没有激励信号 是一个悬空状态

#### 2进制格式

二进制 ：‘b     4'b0101   八进制 ：’o  十进制 ：‘d	 4'd2   十六进制：‘h   4'ha

默认位宽32位 默认数据格式十进制

16‘b1001_1010_1001_1010=16'h9a9a

#### 3标识符

标识符可以是字母 数字 $ _ 的组合

第一个字符必须是字母或者下划线

标识符区别大小写

###### 推荐写法：

​	不建议大小写混用

​	普通内部信号建议全部小写

​	信号命名最好体现信号含义 ，简洁清晰易懂

#### 4数据类型

##### 寄存器 

​	抽象的数据存储单元 通过赋值语句改变寄存器存储的值

​	关键字 reg 默认初始值x

```verilog
reg[31:0] delay_cnt;
reg key_reg;
```

reg类型只能在always语句和initial语句中被赋值

如果该过程语句描述的时序逻辑，即always带有时钟信号，则该寄存器变量对应为触发器

如果该过程语句描述的组合逻辑，即always不带有时钟信号，则该寄存器变量对应为硬件连线

##### 线网

​	线网数据类型表嗾使结构实体（例如门）之间的物理连线

​	线网类型不能存储值，它的值有驱动他的元件所决定

​	驱动线网类型变量的元件有门，连续赋值语句，assign等

​	没有驱动元件连接到线网类型的变量上，则该变量就是高阻，即其值为z

​	线网数据类型包括wire和tri，常用的为wire

​	

```verilog
wire[2:0] key_flag
```

##### 参数  

​	参数就是一个常量 在Verilog中用parameter定义常量

​	可以一次定义多个参数，参数与参数之间需要用逗号隔开

​	每个参数定义的右边必须是一个常数表达式

```verilog
parameter N=8;
```

​	参数型数据常用于定义状态机的状态、数据位宽和延迟大小

​	采用标识符来代表一个常量可以提高程序的可读性和可维护性

​	在模块调用时，可通过参数传递来改变被调用模块中已定义的参数

#### 5运算符

##### 1算术运算符  +-*/%

##### 2关系预算符

```
> < >= <= == !=
```

##### 3逻辑运算符 

！ &&  ||   

##### 4条件运算符

a?b:c

##### 5位运算符

~ （取反）  &  |   ^ (异或)

位宽不足时高位补0

##### 6移位运算符

<<    >>

用0填补移除的空位

左移增加位宽  右移位宽不加边

##### 7拼接运算符

{，}  如 c={A,B[3:0]} 将A和B的第四位拼接

##### 8优先级

![1558681516048](D:\GitHub\Verilog\assets\1558681516048.png)

### 二、程序框架

#### 1注释 

//      /*  */

#### 2 关键字

![1558683148039](D:\GitHub\Verilog\assets\1558683148039.png)

#### 3程序框架

##### 1模块的结构

Verilog的基本设计单元是模块“block”

一个模块是由两部分组成，一部门描述接口 一部分描述逻辑

```verilog
module block(a,b,c,d);
    input a,b;
    output c,d;
    assign c=a|b;
    assign d=a&b;
endmodule
```

每个Verilog程序包括4个主要部分：端口定义 io说明 内部信号声明 功能定义

```verilog
module flow_led(
	input sys_clk,//系统时钟
	input sys_rst,//系统复位
    output reg [3:0] led //4个led灯
);
    reg [23:0] counter;
//******************************************
//******      main code               ******
//******************************************
    always@(posedge sys_clk or negedge sys_rst)
    begin
        if(!sys_rst)
            counter<=0;
        else if(counter <24'd100_0000)
            counter<=counter+1;
        else
            counter<=0;
    end
    //通过移位寄存器控制IO电平高低 从而改变LED显示状态
    always@(posedge sys_clk or negedge sys_rst)
        begin
        if(!sys_rst)
            led<=4'b0001;
        else if(counter==24'd100_0000)
            led[3:0]<={led[2:0],led[3]};
        else
            led<=led;
    end
endmodule
                
```

功能定义部分有三种方法：

1 assign语句 描述组合逻辑

2 always语句 描述组合、时序逻辑

3例化实例元件 如 and#2 u1(q,a,b)

上述三种逻辑功能是**并行**的

**注意：在always语句中，逻辑是顺序执行的 而多个always块之间是并行的**

##### 2模块的调用

在模块调用时，模块通过模块端口在模块之间传递

```verilog
module seg_led_statio_top(
	input sys_clk,//系统时钟
    input sys_rst，//系统复位
    output [5:0] sel,//数码管位选
    output [7:0] seg_led //数码管段选
);
//parameter define 
parameter TIME_SHOW=25'd25000_000
//wire define
wire add_flag;//数码管变化的通知信号
//******************************************
//******      main code               ******
//******************************************
time_count #（
    .MAX_NUM (TIME_SHOW))
  	u_time_count（
    .clk (sys_clk),
    .rst (sys_rst),
    .flag (add_flag)
    );
endmodule
module time_count(
	input clk,
    input rst,
    output reg flag
);
parameter MAX_NUM=50000_000;
//reg define
    reg [24:0] cnt;
```

另一种端口连接方式：

```verilog
time_count #（
    .MAX_NUM (TIME_SHOW))
  	u_time_count（
    sys_clk,
    sys_rst,
    add_flag
    );
```

**模块输入端 可以连接到wire/reg  模块输出端必须连接到wire**

### 三 语句

#### 1结构语句

##### initial

initial语句它在模块中只执行一次

它常用于测试文件的编写 用来产生仿真测试信号（激励信号），或者用于对存储器变量赋初值

```verilog
initial begin
	sys_clk <=1'b0;
	sys_rst <=1'b0;
	touch_key <=1'b0;
    //#表示延时xx个单位的延时 每个#后都代表再原延时的基础上再延时
	#20 sys_rst <=1'b1;
	#10 touch_key=1'b0;
	#30 touch_key=1'b1;
	#110 touch_key=1'b1;
	#30 touch_key=1'b0;
end
```

##### always

always语句一直活在不断的重复活动

但是只有和一定的时间控制结合在一起才有作用

```verilog
always #10 sys_clk<=~sys_clk;
```

always的时间控制可以是沿触发，也可以是电平触发 ；可以是单个信号，也可以是多个信号，多个信号中间要用关键字or连接。

always语句紧跟的过程块是否运行，要看它的触发条件是否满足。

```verilog
    always@(posedge sys_clk or negedge sys_rst)
    begin
        if(!sys_rst)
            counter<=0;
        else if(counter <24'd100_0000)
            counter<=counter+1;
        else
            counter<=0;
    end
```

沿触发的always块常常描述为时序逻辑行为

由关键词or连接的多个事件名或者信号名组成的列表常称为“敏感列表”。

电平触发的always块常用来描述组合逻辑行为

```verilog
always @(a or b or c or d or e or e or f or q or h or m )begin
	out1=a?(b+c):(d+e);
	out2=f?(g+h):(p+m);
end
```

如果组合逻辑块语句的输入变量很多，那么编写敏感列表会很繁琐且容易出错

```verilog
always @(*) begin
	out1=a?(b+c):(d+e);
	out2=f?(g+h):(p+m);
end
```

#### 2赋值语句

##### 阻塞赋值

阻塞赋值可以认为只有一个步骤的操作。计算等号右边并更新等号左边（一个功能）

所谓的阻塞概念是指，在同一个always块中，后面的赋值语句是在前一句赋值语句结束后才开始赋值的。

##### 非阻塞赋值

非阻塞赋值的操作可以看做两个步骤：

​	赋值开始的时候，计算等号右边表达式

​	赋值结束的时候，更新等号左边的值

所谓非阻塞赋值的概念是指，在计算非阻塞赋值等号右边表达式的值以及更新等号左边变量值的时候，允许娶她非阻塞赋值语句同时计算等号右边表达式的值和更新等号左边的值。

非阻塞赋值只能用于对寄存器类型的变量进行赋值，因此只能用于initial和always等过程块中。

**在描述组合逻辑的always块中用阻塞赋值=，综合成组合逻辑的电路结构。**这种电路结构只与输入电平的变化有关系。

**在描述时序逻辑的always块中用非阻塞赋值<=,综合成时序逻辑的电路结构。**这种结构往往与触发沿有关系，只有在触发沿是才可能发生赋值的变化。

**注意：在同一个always块中不要既用非阻塞赋值又用阻塞赋值。**

​			**不允许在多个always块中对同一个变量进行赋值。**
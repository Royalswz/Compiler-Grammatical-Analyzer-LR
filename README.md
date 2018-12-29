# 语法分析器的设计与实现-LR

## 要求（摘自课本

**实验内容** - 编写语法分析程序，实现对算术表达式的语法分析。要求所分析算术表达式由如下文法产生：

```
E -> E+T | E-T | T
T -> T*F | T/F | F
F -> (E) | num
```

**实验要求**
- 在对输入的算术表达式进行分析的过程中，依次输出所采用的产生式。
- 构造识别该文法所有活前缀的DFA
- 构造文法的LR分析表
- 编程实现算法4.3，构造LR分析程序

## 算法4.3（摘自课本

- 输入 - 文法G的一张分析表和一个输入符号串ω
- 输出 - 如果ω∈L(G)，得到ω自底向上的分析，否则报错

方法：

```c++
初始化 {
	初始状态S0压栈
	ω$存入输入缓冲区
	置ip指向ω$的第一个符号
}
do {
	令S是栈顶状态，a是ip指向的符号;
	if (action[S, a] = shift S'){
		把a和S'分别压入符号栈和状态栈的栈顶;
		推进ip，使它进入下一个输入符号;
	} else if (action[S, a] = reduce by A -> β){
		从栈顶弹出|β|个符号;
		令S'是栈顶状态，把A和goto[S', A]分别压入符号栈和状态栈的栈顶;
		输出产生式A -> β;
	} else if (action[S, a] = accept) return;
	else error();
} while (1);
```

## 设计

### 输入与存储

使用之前写的[语法分析器](https://github.com/DiscreteTom/Compiler-Grammatical-Analyzer)的符号表数据结构，定义Symbol类区分非终结符和终结符，定义SymbolTable类保存符号表

输入限制：因为要给此文法设计翻译程序，所以不支持输入自定义文法。此程序只对此文法进行翻译。可以输入待解析符号串。毕竟不是词法分析器，输入数字时限制只能输入正整数。

### 翻译方案设计

```
E' -> E { print(E.v) }
E -> E1 + T { E.v = E1.v + T.v }
E -> E1 - T { E.v = E1.v - T.v }
E -> T { E.v = T.v }
T -> T1 * F { T.v = T1.v * F.v}
T -> T1 / F { T.v = T1.v / F.v}
T -> F { T.v = F.v }
F -> (E) { F.v = E.v }
F -> num { F.v = num.v }
```

### 工作流程设计

1. 计算FIRST集
2. 计算FOLLOW集
3. 构造LR(0)有效项目集及其DFA
4. 由于此文法为SLR(1)文法，仅构造SLR(1)分析表
5. 输入文法符号串
6. 使用预测分析表分析

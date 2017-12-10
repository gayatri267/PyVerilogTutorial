Pyverilog
==============================

Python-based Hardware Design Processing Toolkit for Verilog HDL
Pyverilog is an open-source hardware design processing toolkit for Verilog HDL.

This software includes various tools for Verilog HDL design.
* vparser: Code parser to generate AST (Abstract Syntax Tree) from source codes of Verilog HDL.
* dataflow: Dataflow analyzer with an optimizer to remove redundant expressions and some dataflow handling tools.
* controlflow: Control-flow analyzer with condition analyzer that identify when a signal is activated.
* ast\_code\_generator: Verilog HDL code generator from AST(Abstract Syntax Tree).

You can create your own design analyzer, code translator and code generator of Verilog HDL based on this toolkit.

Prerequisites
==============================

* Python (2.7, 3.3 or later)
* Icarus Verilog (0.9.6 or later)
   - pyverilog.vparser.preprocessor.py uses 'iverilog -E' command as the preprocessor.
* Graphviz and Pygraphviz (Python3 does not support Pygraphviz)
   - pyverilog.dataflow.graphgen and pyverilog.controlflow.controlflow (without --nograph option) use Pygraphviz (on Python 2.7).
* Jinja2 (2.7 or later)
   - ast\_code\_generator requires jinja2 module.

Hands on Tutorial
==============================

Git Clone Our Tutorial
------------------------------

    git clone git@github.com:gayatri267/PyverilogTutorial.git
    

Do the following Installations
------------------------------------
If you are using Linux, you can directly follow the instructions. In case you are using Windows, please install Bash for Windows using the link https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/. It will make the process easier.
Please make sure you have python2.7 Installed before continuing with this tutorial.
Some of the commands in this tutorial can only be run using Python 2.7, while others can be run using Python 2.7 or Python 3
(We will specify during the commands if python2.7 is to be explicitly used)

1. Go into the pyverilog-0.9.1 directory in the cloned repository
    cd pyverilog-0.9.1
     
2. Install iverilog as below 
    ```
    sudo apt-get install iverilog
    ```
    
3. Install Pygraphviz as below (Note that Python3 does not support Pygraphviz and hence only Python2.7 is to be installed)
    ```
    sudo apt-get install -y python-pygraphviz
    ```

4. Install Jinja 2 as below
    ```
    sudo apt install python-pip
    pip install jinja2
    ```
5. Install the pyverilog library using setup.py.
    * If Python 2.7 is used,
    ```
    python setup.py install
    ```

    * If Python 3.x is used,
    ```
    python3 setup.py install
    ```
    

    
Example 1 (test.v)
-----------------------------
Let's try to use pyverilog tools on the verilog module test.v(already present in the pyverilog-0.9.1 directory)
This sample design adds the input value internally whtn the enable signal is asserted. Then it outputs its partial value to the LED.

### Code parser
Code parer is the syntax analysis. Please type the command as below.

    python pyverilog/vparser/parser.py test.v

The result of syntax analysis is displayed.

```
Source: 
  Description: 
    ModuleDef: top
      Paramlist: 
      Portlist: 
        Ioport: 
          Input: CLK, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: RST, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: enable, False
            Width: 
              IntConst: 0
              IntConst: 0
        Ioport: 
          Input: value, False
            Width: 
              IntConst: 31
              IntConst: 0
        Ioport: 
          Output: led, False
            Width: 
              IntConst: 7
              IntConst: 0
      Decl: 
        Reg: count, False
          Width: 
            IntConst: 31
            IntConst: 0
      Decl: 
        Reg: state, False
          Width: 
            IntConst: 7
            IntConst: 0
      Assign: 
        Lvalue: 
          Identifier: led
        Rvalue: 
          Partselect: 
            Identifier: count
            IntConst: 23
            IntConst: 16
      Always: 
        SensList: 
          Sens: posedge
            Identifier: CLK
        Block: None
          IfStatement: 
            Identifier: RST
            Block: None
              NonblockingSubstitution: 
                Lvalue: 
                  Identifier: count
                Rvalue: 
                  IntConst: 0
              NonblockingSubstitution: 
                Lvalue: 
                  Identifier: state
                Rvalue: 
                  IntConst: 0
            Block: None
              IfStatement: 
                Eq: 
                  Identifier: state
                  IntConst: 0
                Block: None
                  IfStatement: 
                    Identifier: enable
                    NonblockingSubstitution: 
                      Lvalue: 
                        Identifier: state
                      Rvalue: 
                        IntConst: 1
                IfStatement: 
                  Eq: 
                    Identifier: state
                    IntConst: 1
                  Block: None
                    NonblockingSubstitution: 
                      Lvalue: 
                        Identifier: state
                      Rvalue: 
                        IntConst: 2
                  IfStatement: 
                    Eq: 
                      Identifier: state
                      IntConst: 2
                    Block: None
                      NonblockingSubstitution: 
                        Lvalue: 
                          Identifier: count
                        Rvalue: 
                          Plus: 
                            Identifier: count
                            Identifier: value
                      NonblockingSubstitution: 
                        Lvalue: 
                          Identifier: state
                        Rvalue: 
                          IntConst: 0
```

### Dataflow analyzer
Let's try dataflow analysis. It is used to establish the relationship between outputs with inputs and states

    python3 pyverilog/dataflow/dataflow_analyzer.py -t top test.v 

The result of each signal definition and each signal assignment are displayed.

```
Directive:
Instance:
(top, 'top')
Term:
(Term name:top.led type:{'Output'} msb:(IntConst 7) lsb:(IntConst 0))
(Term name:top.enable type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.CLK type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.count type:{'Reg'} msb:(IntConst 31) lsb:(IntConst 0))
(Term name:top.state type:{'Reg'} msb:(IntConst 7) lsb:(IntConst 0))
(Term name:top.RST type:{'Input'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:top.value type:{'Input'} msb:(IntConst 31) lsb:(IntConst 0))
Bind:
(Bind dest:top.count tree:(Branch Cond:(Terminal top.RST) True:(IntConst 0) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 0)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 1)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 2)) True:(Operator Plus Next:(Terminal top.count),(Terminal top.value)))))))
(Bind dest:top.state tree:(Branch Cond:(Terminal top.RST) True:(IntConst 0) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 0)) True:(Branch Cond:(Terminal top.enable) True:(IntConst 1)) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 1)) True:(IntConst 2) False:(Branch Cond:(Operator Eq Next:(Terminal top.state),(IntConst 2)) True:(IntConst 0))))))
(Bind dest:top.led tree:(Partselect Var:(Terminal top.count) MSB:(IntConst 23) LSB:(IntConst 16)))
```

To view the result of dataflow analysis as a picture file, need to run the command as below (we select output port 'led' as the target for example)

    python3 pyverilog/dataflow/graphgen.py -t top -s top.led test.v 

out.png file will now be generated which has the definition of 'led' is a part-selection of 'count' from 23-bit to 16-bit.

![out.png](http://cdn-ak.f.st-hatena.com/images/fotolife/s/sxhxtxa/20140101/20140101045641.png)

### Control-flow analyzer
Control-flow analysis can be used to picturize how the state diagram of the RTL module look like

    python2.7 pyverilog/controlflow/controlflow_analyzer.py -t top test.v 
(Note that only python2.7 can be used to this command as it internally uses Pygraphviz)

We get the output as below, which shows the state machine structure and transition conditions to the next state in the state machine.

```
FSM signal: top.count, Condition list length: 4
FSM signal: top.state, Condition list length: 5
Condition: (Ulnot, Eq), Inferring transition condition
Condition: (Eq, top.enable), Inferring transition condition
Condition: (Ulnot, Ulnot, Eq), Inferring transition condition
# SIGNAL NAME: top.state
# DELAY CNT: 0
0 --(top_enable>'d0)--> 1
1 --None--> 2
2 --None--> 0
Loop
(0, 1, 2)
```

top_state.png is also generated which is the graphical representation of the state machine.

![top_state.png](http://cdn-ak.f.st-hatena.com/images/fotolife/s/sxhxtxa/20140101/20140101045835.png)

### Code Generator
Finally, pyverilog can be used to generate verilog code from the AST representation of RTL. We will be using 'test.py' for demonstrate
A Verilog HDL code is represented by using the AST classes defined in 'vparser.ast'.
Run the below command to see how AST representation in test.py gets translated to verilog code
```
python test.py
```

Verilog code generated from the AST instances is as below:

```verilog

module top
 (
  input [0:0] CLK, 
input [0:0] RST, 
output [7:0] led

 );
  assign led = 8;
endmodule

```

Example 2 (fsm.v)
-----------------------------
Let's try to use pyverilog tools on the verilog module fsm.v(already present in the pyverilog-0.9.1 directory)


### Code parser
Code parer is the syntax analysis. Please type the command as below.

    python pyverilog/vparser/parser.py fsm.v

The result of syntax analysis is displayed.

```
Source:
  Description:
    ModuleDef: fsm2
      Paramlist:
      Portlist:
        Port: clock, None
        Port: reset, None
        Port: req_0, None
        Port: req_1, None
        Port: gnt_0, None
        Port: gnt_1, None
      Decl:
        Input: clock, False
          Width:
            IntConst: 0
            IntConst: 0
        Input: reset, False
          Width:
            IntConst: 0
            IntConst: 0
        Input: req_0, False
          Width:
            IntConst: 0
            IntConst: 0
        Input: req_1, False
          Width:
            IntConst: 0
            IntConst: 0
      Decl:
        Output: gnt_0, False
          Width:
            IntConst: 0
            IntConst: 0
        Output: gnt_1, False
          Width:
            IntConst: 0
            IntConst: 0
      Decl:
        Wire: clock, False
          Width:
            IntConst: 0
            IntConst: 0
        Wire: reset, False
          Width:
            IntConst: 0
            IntConst: 0
        Wire: req_0, False
          Width:
            IntConst: 0
            IntConst: 0
        Wire: req_1, False
          Width:
            IntConst: 0
            IntConst: 0
      Decl:
        Reg: gnt_0, False
          Width:
            IntConst: 0
            IntConst: 0
        Reg: gnt_1, False
          Width:
            IntConst: 0
            IntConst: 0
      Decl:
        Parameter: SIZE, False
          Rvalue:
            IntConst: 3
      Decl:
        Parameter: IDLE, False
          Rvalue:
            IntConst: 3'b001
        Parameter: GNT0, False
          Rvalue:
            IntConst: 3'b010
        Parameter: GNT1, False
          Rvalue:
            IntConst: 3'b100
      Decl:
        Reg: state, False
          Width:
            Minus:
              Identifier: SIZE
              IntConst: 1
            IntConst: 0
      Decl:
        Reg: next_state, False
          Width:
            Minus:
              Identifier: SIZE
              IntConst: 1
            IntConst: 0
      Always:
        SensList:
          Sens: level
            Identifier: state
          Sens: level
            Identifier: req_0
          Sens: level
            Identifier: req_1
        Block: FSM_COMBO
          BlockingSubstitution:
            Lvalue:
              Identifier: next_state
            Rvalue:
              IntConst: 3'b000
          CaseStatement:
            Identifier: state
            Case:
              Identifier: IDLE
              IfStatement:
                Eq:
                  Identifier: req_0
                  IntConst: 1'b1
                Block: None
                  BlockingSubstitution:
                    Lvalue:
                      Identifier: next_state
                    Rvalue:
                      Identifier: GNT0
                IfStatement:
                  Eq:
                    Identifier: req_1
                    IntConst: 1'b1
                  Block: None
                    BlockingSubstitution:
                      Lvalue:
                        Identifier: next_state
                      Rvalue:
                        Identifier: GNT1
                  Block: None
                    BlockingSubstitution:
                      Lvalue:
                        Identifier: next_state
                      Rvalue:
                        Identifier: IDLE
            Case:
              Identifier: GNT0
              IfStatement:
                Eq:
                  Identifier: req_0
                  IntConst: 1'b1
                Block: None
                  BlockingSubstitution:
                    Lvalue:
                      Identifier: next_state
                    Rvalue:
                      Identifier: GNT0
                Block: None
                  BlockingSubstitution:
                    Lvalue:
                      Identifier: next_state
                    Rvalue:
                      Identifier: IDLE
            Case:
              Identifier: GNT1
              IfStatement:
                Eq:
                  Identifier: req_1
                  IntConst: 1'b1
                Block: None
                  BlockingSubstitution:
                    Lvalue:
                      Identifier: next_state
                    Rvalue:
                      Identifier: GNT1
                Block: None
                  BlockingSubstitution:
                    Lvalue:
                      Identifier: next_state
                    Rvalue:
                      Identifier: IDLE
            Case:
              BlockingSubstitution:
                Lvalue:
                  Identifier: next_state
                Rvalue:
                  Identifier: IDLE
      Always:
        SensList:
          Sens: posedge
            Identifier: clock
        Block: FSM_SEQ
          IfStatement:
            Eq:
              Identifier: reset
              IntConst: 1'b1
            Block: None
              NonblockingSubstitution:
                Lvalue:
                  Identifier: state
                Rvalue:
                  Identifier: IDLE
                DelayStatement:
                  IntConst: 1
            Block: None
              NonblockingSubstitution:
                Lvalue:
                  Identifier: state
                Rvalue:
                  Identifier: next_state
                DelayStatement:
                  IntConst: 1
      Always:
        SensList:
          Sens: posedge
            Identifier: clock
        Block: OUTPUT_LOGIC
          IfStatement:
            Eq:
              Identifier: reset
              IntConst: 1'b1
            Block: None
              NonblockingSubstitution:
                Lvalue:
                  Identifier: gnt_0
                Rvalue:
                  IntConst: 1'b0
                DelayStatement:
                  IntConst: 1
              NonblockingSubstitution:
                Lvalue:
                  Identifier: gnt_1
                Rvalue:
                  IntConst: 1'b0
                DelayStatement:
                  IntConst: 1
            Block: None
              CaseStatement:
                Identifier: state
                Case:
                  Identifier: IDLE
                  Block: None
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_0
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_1
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
                Case:
                  Identifier: GNT0
                  Block: None
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_0
                      Rvalue:
                        IntConst: 1'b1
                      DelayStatement:
                        IntConst: 1
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_1
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
                Case:
                  Identifier: GNT1
                  Block: None
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_0
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_1
                      Rvalue:
                        IntConst: 1'b1
                      DelayStatement:
                        IntConst: 1
                Case:
                  Block: None
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_0
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
                    NonblockingSubstitution:
                      Lvalue:
                        Identifier: gnt_1
                      Rvalue:
                        IntConst: 1'b0
                      DelayStatement:
                        IntConst: 1
```

### Dataflow analyzer
Let's try dataflow analysis. It is used to establish the relationship between outputs with inputs and states

    python3 pyverilog/dataflow/dataflow_analyzer.py -t fsm fsm.v 

The result of each signal definition and each signal assignment are displayed.

```
Directive:
Instance:
(fsm2, 'fsm2')
Term:
(Term name:fsm2.next_state type:{'Reg'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2._rn0_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.gnt_0 type:{'Reg', 'Output'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2._rn2_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2._rn3_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2._rn6_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.req_0 type:{'Input', 'Wire'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2.req_1 type:{'Input', 'Wire'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2.IDLE type:{'Parameter'} msb:'d31 lsb:'d0)
(Term name:fsm2.GNT0 type:{'Parameter'} msb:'d31 lsb:'d0)
(Term name:fsm2.SIZE type:{'Parameter'} msb:'d31 lsb:'d0)
(Term name:fsm2._rn1_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.clock type:{'Input', 'Wire'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2.reset type:{'Input', 'Wire'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2._rn8_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.gnt_1 type:{'Reg', 'Output'} msb:(IntConst 0) lsb:(IntConst 0))
(Term name:fsm2._rn5_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.GNT1 type:{'Parameter'} msb:'d31 lsb:'d0)
(Term name:fsm2._rn4_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2._rn7_next_state type:{'Rename'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
(Term name:fsm2.state type:{'Reg'} msb:(Operator Minus Next:(Terminal fsm2.SIZE),(IntConst 1)) lsb:(IntConst 0))
Bind:
(Bind dest:fsm2.next_state tree:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.IDLE)) True:(Branch Cond:(Operator Eq Next:(Terminal fsm2.req_0),(IntConst 1'b1)) True:(Terminal fsm2._rn1_next_state) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.req_1),(IntConst 1'b1)) True:(Terminal fsm2._rn2_next_state) False:(Terminal fsm2._rn3_next_state))) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT0)) True:(Branch Cond:(Operator Eq Next:(Terminal fsm2.req_0),(IntConst 1'b1)) True:(Terminal fsm2._rn4_next_state) False:(Terminal fsm2._rn5_next_state)) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT1)) True:(Branch Cond:(Operator Eq Next:(Terminal fsm2.req_1),(IntConst 1'b1)) True:(Terminal fsm2._rn6_next_state) False:(Terminal fsm2._rn7_next_state)) False:(Branch Cond:(IntConst 1) True:(Terminal fsm2._rn8_next_state))))))
(Bind dest:fsm2._rn0_next_state tree:(IntConst 3'b000))
(Bind dest:fsm2.gnt_0 tree:(Branch Cond:(Operator Eq Next:(Terminal fsm2.reset),(IntConst 1'b1)) True:(IntConst 1'b0) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.IDLE)) True:(IntConst 1'b0) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT0)) True:(IntConst 1'b1) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT1)) True:(IntConst 1'b0) False:(Branch Cond:(IntConst 1) True:(IntConst 1'b0)))))))
(Bind dest:fsm2._rn2_next_state tree:(Terminal fsm2.GNT1))
(Bind dest:fsm2._rn3_next_state tree:(Terminal fsm2.IDLE))
(Bind dest:fsm2.state tree:(Branch Cond:(Operator Eq Next:(Terminal fsm2.reset),(IntConst 1'b1)) True:(Terminal fsm2.IDLE) False:(Terminal fsm2.next_state)))
(Bind dest:fsm2._rn6_next_state tree:(Terminal fsm2.GNT1))
(Bind dest:fsm2.IDLE tree:(IntConst 3'b001))
(Bind dest:fsm2.GNT0 tree:(IntConst 3'b010))
(Bind dest:fsm2.SIZE tree:(IntConst 3))
(Bind dest:fsm2._rn1_next_state tree:(Terminal fsm2.GNT0))
(Bind dest:fsm2._rn8_next_state tree:(Terminal fsm2.IDLE))
(Bind dest:fsm2.gnt_1 tree:(Branch Cond:(Operator Eq Next:(Terminal fsm2.reset),(IntConst 1'b1)) True:(IntConst 1'b0) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.IDLE)) True:(IntConst 1'b0) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT0)) True:(IntConst 1'b0) False:(Branch Cond:(Operator Eq Next:(Terminal fsm2.state),(Terminal fsm2.GNT1)) True:(IntConst 1'b1) False:(Branch Cond:(IntConst 1) True:(IntConst 1'b0)))))))
(Bind dest:fsm2.GNT1 tree:(IntConst 3'b100))
(Bind dest:fsm2._rn4_next_state tree:(Terminal fsm2.GNT0))
(Bind dest:fsm2._rn7_next_state tree:(Terminal fsm2.IDLE))
(Bind dest:fsm2._rn5_next_state tree:(Terminal fsm2.IDLE))
```

To view the result of dataflow analysis as a picture file, need to run the command as below (we select output port 'fsm.gnt_0' as the target for example)

    python pyverilog/dataflow/graphgen.py -t fsm -s fsm.gnt_0 fsm.v 

out.png file will now be generated which has the definition of 'gnt_0'.
![alt text](https://drive.google.com/uc?id=1YCJZ198a4jnjtMBxkNEB159pFs3HHlhh)

### Control-flow analyzer
Control-flow analysis can be used to picturize how the state diagram of the RTL module look like

    python pyverilog/controlflow/controlflow_analyzer.py -t fsm fsm.v 
(Note that only python2.7 can be used to this command as it internally uses Pygraphviz)

We get the output as below, which shows the state machine structure and transition conditions to the next state in the state machine.

```
FSM signal: fsm2.gnt_1, Condition list length: 4
Condition: (Eq,), Inferring transition condition
Condition: (Ulnot, Eq), Inferring transition condition
Condition: (Ulnot, Ulnot, Eq), Inferring transition condition
Condition: (Ulnot, Ulnot, Ulnot), Inferring transition condition
FSM signal: fsm2.gnt_0, Condition list length: 3
Condition: (Eq,), Inferring transition condition
Condition: (Ulnot, Eq), Inferring transition condition
Condition: (Ulnot, Ulnot), Inferring transition condition
FSM signal: fsm2.state, Condition list length: 8
Condition: (Eq, Eq), Inferring transition condition
Condition: (Eq, Ulnot, Ulnot), Inferring transition condition
Condition: (Ulnot, Eq, Eq), Inferring transition condition
Condition: (Ulnot, Ulnot, Ulnot), Inferring transition condition
Condition: (Eq, Ulnot, Eq), Inferring transition condition
Condition: (Ulnot, Eq, Ulnot), Inferring transition condition
Condition: (Ulnot, Ulnot, Eq, Ulnot), Inferring transition condition
Condition: (Ulnot, Ulnot, Eq, Eq), Inferring transition condition
# SIGNAL NAME: fsm2.state
# DELAY CNT: 0
0 --None--> 1
1 --((!(fsm2_req_0==1'd1))&&(!(fsm2_req_1==1'd1)))--> 1
1 --(fsm2_req_0==1'd1)--> 2
1 --((!(fsm2_req_0==1'd1))&&(fsm2_req_1==1'd1))--> 4
2 --(fsm2_req_0==1'd1)--> 2
2 --(!(fsm2_req_0==1'd1))--> 1
3 --None--> 1
4 --(!(fsm2_req_1==1'd1))--> 1
4 --(fsm2_req_1==1'd1)--> 4
5 --None--> 1
6 --None--> 1
7 --None--> 1
Loop
(1, 2)
(2,)
(1,)
(1, 4)
(4,)
```

fsm_state.png is also generated which is the graphical representation of the state machine.
![alt text](https://drive.google.com/uc?id=1D9hBez8kQRp5SKjboTjBAuvRO0QNQbah)


# RTL Design and Simulation and Coverage using vcs and verdi 
## 1. Create the Project Folder
## 2. Open the Folder in Terminal And Type To Activate The Tools

    source /home/Admin/mjcet

## 3. Create the Standard Directory Structure In The Folder Which You Created Which is full_adder And Inside That Make Three Folders

    rtl
    tb
    sim

# RTL DESIGN (fa.v)
In rtl folder create fa.v file and paste the below code

    module fa(a,b,cin,sum,cout);
    input a,b,cin;
    output sum,cout;

    assign sum = a ^ b ^ cin;
    assign cout =  (a&b) | (b&cin) | (a&cin);

    endmodule

# TESTBENCH FOR SIMULATION (tb_fa.v)
In tb folder create tb_fa.v file and paste the below code

    module tb_fa;

    reg a, b, cin;
    wire sum,cout;

    fa dut (a,b,cin,sum,cout);

        $fsdbDumpfile("tb_fa.fsdb");
    initial begin
        $fsdbDumpvars(0,tb_fa);
    end

    initial begin
        $monitor("%0t    %b %b %b | %b   %b", $time, a, b, cin, sum, cout);
        a=0;b=1;cin=1; #5;
        a=1;b=1;cin=0; #5;
        $finish;
    end

    endmodule

# SIMULATION

```

    vcs ../rtl/fa.v ../tb/tb_fa.v -lca -kdb -debug_access+all -full64
```

```
    ./simv

```

```
    ./simv - verdi &
```
# SYNTHESIS (Synopsys DC)



## Run DC
To invoke the tool type the below command 
```
    dc_shell
```
```
    source -echo -verbose ./rm_setup/dc_setup.tcl
```
```
    set RTL_SOURCE_FILES ./../rtl/fa.v
```
```
    define_design_lib WORK -path ./WORK
```
```
    analyze -format verilog ${RTL_SOURCE_FILES}
```
```
    start_gui
```
```
    elaborate ${DESIGN_NAME}
```
```
    read_sdc ./../CONSTRAINTS/fa.sdc
```
```
    get_ports
```
```
    change_selection [get_ports]
```
```
    get_lib
```
```
compile
```
you can either use compile or compile_ultra command 
```
    compile_ultra
```
```
    report_cells
```
```
    report_timing
```
```
    write -format verilog -hierarchy -output ${RESULTS_DIR}/${DCRM_FINAL_VERILOG_OUTPUT_FILE}
```

# COVERAGE 
To do coverage we dont use verilog we use SystemVerilog so remove tb_fa.v from tb folder and create a file called tb_fa.sv then paste the below code 

    `timescale 1ns/1ps

    module tb_fa;

        reg a, b, cin;
        wire sum, cout;

        fa dut (a, b, cin, sum, cout);

        covergroup cg_fa;
            coverpoint a { bins zero = {0}; bins one = {1}; }
            coverpoint b { bins zero = {0}; bins one = {1}; }
            coverpoint cin { bins zero = {0}; bins one = {1}; }
            a_b_cin_cross: cross a, b, cin;
            coverpoint sum { bins zero = {0}; bins one = {1}; }
            coverpoint cout { bins zero = {0}; bins one = {1}; }
        endgroup

        cg_fa fa_cg = new();

        initial begin
            $display("Time\t a b cin | sum cout");
            $monitor("%0t\t %b %b %b | %b   %b", $time, a, b, cin, sum, cout);

            for (int i=0; i<8; i++) begin
                {a,b,cin} = i;
                #10;
                fa_cg.sample();
            end

            fa_cg.display();
            $finish;
        end

    endmodule

# COVERAGE COMMANDS
```
    vcs ../rtl/fa.v ../tb/tb_fa.sv -cm line+tgl+cond
```
if your have fsm and branch type below command 
```
    vcs ../rtl/fa.v ../tb/tb_fa.sv -cm line+tgl+cond+fsm+branch
```
```
    ./simv -cm line+tgl+cond
```
if your have fsm and branch type below command 

```
    ./simv -cm line+tgl+cond+fsm+branch
```

# POST-PROCESSING (URG)

```
urg -dir simv.vdb
```
```
verdi -cov -covdir simv.vdb
```
if you want report in both text and html type below command
```
urg -dir simv.vdb -format both
```
if you want report with custom folder name  type below command

```
urg -dir simv.vdb -report customReportFolder
```
custom folder and prallel then type below command
```
urg -dir simv.vdb -report customReportFolder -parallel
```

to do only line and toggle type below command
```
urg -dir simv.vdb -metric line+tgl
```
if you want to open in verdi then type 
```
verdi -cov -covdir simv.vdb
```

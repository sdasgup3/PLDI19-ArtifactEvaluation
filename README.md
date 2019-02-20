# [Paper 195] PLDI 2019 Artifact Evaluation


## Artifact Submission
- Accepted Paper [pdf](https://github.com/sdasgup3/PLDI19-ArtifactEvaluation/blob/master/pldi2019-paper195.pdf)
- VM Details
  - VM Player: [VirtualBox 5.1](https://www.virtualbox.org/wiki/Download_Old_Builds_5_1)
  - Ubuntu Image: [ova](https://drive.google.com/file/d/1CS-jqQdtFPzS58lTXOspazDdfSG9hT0_/view?usp=sharing)
    - md5 hash: a2aa3a7f74b9b1668d8f5b6125c8aef1
    - login: sdasgup3
    - password: aecadmin123
  - Guest Machine requirements
    - Minimum 8 GB of RAM.
    - Recommended number of processors is 4 to allow parallel experiments.
  - Host machine requirements
    - Architecture family of host processor must be Haswell or beyond.
    - Enable the processor flag `avx2` in the guest Ubuntu, if not enabled by default (which can be checked in `/proc/cpuinfo`), using the following command in the host machine, where vm_name is the name used for the VM. According to the [link](https://askubuntu.com/questions/699077/how-to-enable-avx2-extensions-on-a-ubuntu-guest-in-virtualbox-5), such flags are exposed in the guest machine by default since VirtualBox 5.0 Beta 3.
      ```bash
      $ VBoxManage setextradata "vm_name" VBoxInternal/CPUM/IsaExts/AVX2 1
      ```



## Getting Started Guide
In this section, we provide the instructions to (1) Compile the semantics definition, and (2) Interpret a simple program using the "executable" semantics.

### Compile the x86-64 semantics
In this section, we demonstrate how to compile the semantics of instructions along with the semantics of execution environment. Below, we have included semantics of all the instructions for compilation (using the `cp` command ) and the compilation will take up-to 15 mins. The reviewer can choose to include few instructions as well (for example, by including the instruction semantics in `systemInstructions` directory only).

```bash
$ cd /home/sdasgup3/Github/binary-decompilation/x86-semantics/semantics
$ cp registerInstructions/* memoryInstructions/* immediateInstructions/* systemInstructions/* underTestInstructions/
$ ../scripts/kompile.pl --backend java
```

### A simple test run
We demonstrate how to interpret a X64-64 assembly language program using the semantics compiled above. For demonstration, we use the [bubble-sort program](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/tests/Programs/bubblesort/test.c), which not only does the sorting but pretty-prints its results. The example, was chosen to show the c-library support provided (line 799-800) over and above the executability of the semantics.

The following command converts a C file to assembly program and interprets it.
```bash
$ cd /home/sdasgup3/Github/binary-decompilation/x86-semantics/semantics
$ ../scripts/run-single-c-file.sh ../tests/Programs/bubblesort/test.c
```
The expected output must looks like
```C
Before Sort
4 3 2 1 0
After Sort
0 1 2 3 4
```
The reviewer is encouraged to chose & run any other program from `x86-semantics/tests/Programs`. For example,
```bash
$ cd /home/sdasgup3/Github/binary-decompilation/x86-semantics/tests/Programs/stdio_fprintf
$ make all
```
The expected output is: A file named file.txt is created in the current working directory with contents as "We are in 2019"

To run all the programs in the directory use (**take ~30 mins**)
```bash
$ cd tests/program-tests
$ ./run-tests.sh --cleankstate
$ ./run-tests.sh --kstate -jobs 4
```

## Step-by-Step Instructions

### Artifacts for "Semantics of Instruction & Execution environment" (Refer sections 3.3, 3.4 & 3.5)

| Semantics                | Modeled by                                                                                                  |
|-------------------------|--------------------------------------------------------------------------------------------------------------|
| Instructions  | [register-instructions/\*.k](https://github.com/sdasgup3/binary-decompilation/tree/pldi19_AE_ConcreteExec/x86-semantics/semantics/registerInstructions) [immediate-instructions/\*.k](https://github.com/sdasgup3/binary-decompilation/tree/pldi19_AE_ConcreteExec/x86-semantics/semantics/immediateInstructions)  [memory-instructions/\*.k](https://github.com/sdasgup3/binary-decompilation/tree/pldi19_AE_ConcreteExec/x86-semantics/semantics/memoryInstructions)  [control-flow-instructions/\*.k](https://github.com/sdasgup3/binary-decompilation/tree/pldi19_AE_ConcreteExec/x86-semantics/semantics/systemInstructions) |
| Execution environment   | [x86-env-init.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-env-init.k) [x86-fetch-execute.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-fetch-execute.k) [x86-loader.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-loader.k)                                                              |
| Memory model            | [x86-memory.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-memory.k)                                                                                                 |
| C-library support       | [x86-builtin.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-builtin.k) [x86-c-library.k](https://github.com/sdasgup3/binary-decompilation/blob/pldi19_AE_ConcreteExec/x86-semantics/semantics/x86-c-library.k)                                                                                |

### Artifacts for "Testing"

#### Instruction Level Testing (Refer Section 4.1 --> Instruction Level Validation)
The instruction level testing is done using (1) [Stoke](https://github.com/StanfordPL/stoke)'s testing infrastructure, and (2) [K](https://github.com/kframework/k) interpreter generated using our semantic definition.

##### testing using Stoke's testing infrastructure
All the test logs and commands are available at [link](https://github.com/sdasgup3/x86-64-instruction-summary/tree/master/nightlyruns). **( We do not provide the repository in VM as it weighs 33 GB)**

Lets fist try to interpret some of the files present in the above link.
 - [job.04](https://github.com/sdasgup3/x86-64-instruction-summary/blob/master/nightlyruns/job.04): A collection of instructions to be tested as a batch (or job).
 - [info.04](https://github.com/sdasgup3/x86-64-instruction-summary/blob/master/nightlyruns/info.04): Information about the job. Like the date on which the test was fired, output file name, etc.
 - [runlog.04](https://github.com/sdasgup3/x86-64-instruction-summary/blob/master/nightlyruns/runlog.04): The output of the job run.

Note that the number `04` represents an ID that corresponds to `Test Chart` [tables](https://github.com/sdasgup3/x86-64-instruction-summary/tree/master/nightlyruns#test-charts-2) provided at [link](https://github.com/sdasgup3/x86-64-instruction-summary/blob/master/nightlyruns/README.md). These tables provide the information about how individual instructions are tested using 3 broad categories: Registers instructions, Immediate instructions and Memory instructions. The distribution is made because of different challenges need to address while testing each category.

As the entire testing took several days to complete, so we will provide instructions about testings sample instructions from each category in order to reproduce the results.

##### Testing a register instruction ( ~4 mins runtime )
The idea is to first create an instance of the assembly instruction under test and then to test that instance using a set of input CPU states. In the `Test charts` link above, we might find variants of the the command mentioned below, for example, one with an extra `--samereg` switch. This is to ensure that instructions like `xchg`, `xadd`, `cmpxchg` are tested with both the source and destination operands as same registers. This is important as the semantics rules of such instructions are different when the source and destination are the same and hence we test them separately.
  ```bash
  $ cd ~/TestArena
  $ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --prepare_concrete --opcode psrlq_xmm_xmm --workdir concrete_instances/register-variants/psrlq_xmm_xmm
  $ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --check_stoke --file concrete_instances/register-variants/psrlq_xmm_xmm/check_stoke.txt --instructions_path concrete_instances/register-variants/psrlq_xmm_xmm/instructions  --testid 00
  ```
  Expected output:
  ```C
    Info:Firing Last 1
    Thread 1 start: psrlq_xmm_xmm
    timeout 30m     /home/sdasgup3/Github/strata/stoke/./bin/stoke_check_circuit --strata_path /Github/strata-data/circuits --target concrete_instances/register-variants/psrlq_xmm_xmm/instructions/psrlq_xmm_xmm/psrlq_xmm_xmm.s --functions ~/Github/strata-data/data-regs/functions --testcases ~/Github/strata-data/data-regs/testcases.tc --def_in "{ %xmm1 %xmm2 }" --live_out "{ %xmm1 }" --maybe_undef_out "{ }"
    Reading the Testsuit
    Preparing Sandbox
    Run 6630 tests
    Collect Results
    Check Equivalence
    Completed 1000cases
    Completed 2000cases
    Completed 3000cases
    Completed 4000cases
    Completed 5000cases
    Completed 6000cases
    Execution time popcntq_r64_r64: 208 s
    Thread 1 done!: popcntq_r64_r64
  ```
Actual runlog: [Log](https://github.com/sdasgup3/x86-64-instruction-summary/tree/master/concrete_instances/register-variants/psrlq_xmm_xmm)

Similar logs for other instructions can also be found using similar paths as above.


##### Testing a immediate instruction ( ~4 mins runtime )
The idea is same as above except the fact that for each immediate instruction of immediate operand width as 8, we create 256 variants of instance of assembly instruction each corresponding to 256 immeidate values and test all of them. We spawn 256 software threads to accommodate all the runs for each instruction.

In the example below, we will be testing the instruction psrlq_xmm_imm8 for just 4 immediate operand values (0-3).
```bash
$ cd ~/TestArena
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --prepare_concrete_imm --opcode psrlq_xmm_imm8 --workdir concrete_instances/immediate-variants/psrlq_xmm_imm8
$ sed -i '5,$ d' concrete_instances/immediate-variants/psrlq_xmm_imm8/check_stoke.txt
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --check_stoke --file concrete_instances/immediate-variants/psrlq_xmm_imm8/check_stoke.txt --instructions_path concrete_instances/immediate-variants/psrlq_xmm_imm8/instructions  --testid 00
```
Actual runlog: [Log](https://github.com/sdasgup3/x86-64-instruction-summary/tree/master/concrete_instances/immediate-variants/psrlq_xmm_imm8). Similar logs for other instructions can also be found using similar paths as above.


##### Testing a memory instruction ( ~4 mins runtime )
The idea is same as above ideas (when the memory instruction has an immediate or register operand) except the fact the [Strata testcases](https://raw.githubusercontent.com/sdasgup3/strata-data-private/master/data-regs/testcases.tc) are not meant to test the memory instructions (In fact the Strata project do not test or synthesize the memory instructions). Hence, the testcases need to be modified slightly to accommodate testing memory-variants. For example, it we want to test `addq (%rbx), %rax`, we need to make sure that the register `%rbx` points to a valid memory address with some value to read from. We accomplish this using the switch `--update_tc` mentioned below.

```bash
$ cd ~/TestArena
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --prepare_concrete --opcode psrlq_xmm_m128 --workdir concrete_instances/memory-variants/psrlq_xmm_m128
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --opcode  psrlq_xmm_m128  --instructions_path concrete_instances/memory-variants/psrlq_xmm_m128/instructions/ --update_tc --testid 00
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --check_stoke --file concrete_instances/memory-variants/psrlq_xmm_m128/check_stoke.txt --instructions_path concrete_instances/memory-variants/psrlq_xmm_m128/instructions --use_updated_tc --testid 00
```
Actual runlog: [Log](https://github.com/sdasgup3/x86-64-instruction-summary/tree/master/concrete_instances/memory-variants/psrlq_xmm_m128). Similar logs for other instructions can also be found using similar paths as above.

#### Testing using K-interpreter
Empowered by the fact that we can directly execute the se-
mantics using the K framework, we validated our model by co-simulating it against a real machine.





#### Comparing with Stoke (Refer Section 4.2)
In this section, we provide instructions about how we cross-checked (using Z3 comparison) our semantics of those instruction which are modeled by [Stoke](https://github.com/StanfordPL/stoke) (say ST1) as well. We own a separate branch of [Stoke](https://github.com/sdasgup3/strata-stoke) (say ST2) where we manually modeled many instruction's semantics to compare against ST1.

Comparison is achieved by using unsat checks on the corresponding SMT formulas. Such comparison helped in unveiling many bugs as reported in Section 4.2.

Below, we give example of one such instruction `psrlq` for which we found that the ST1 implementation to be buggy(which is fixed in ST1 by now using [pull request](https://github.com/StanfordPL/stoke/pull/984).

```bash
$ cd ~/TestArena
$ ~/Github/binary-decompilation/x86-semantics/scripts/process_spec.pl --prepare_concrete --opcode psrlq_xmm_m128 --workdir concrete_instances/memory-variants/psrlq_xmm_m128
$ ~/Github/master_stoke/bin/stoke_debug_formula  --opc psrlq_xmm_m128 --smtlib_format &> concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.ST1.z3.sym
$ ~/Github/strata/stoke/bin/stoke_debug_circuit  --opc psrlq_xmm_m128 --smtlib_format &> concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.ST2.z3.sym


$ ~/Github/binary-decompilation/x86-semantics/scripts/z3compare.pl --file concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.ST1.z3.sym  --file concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.ST2.z3.sym --opcode psrlq_xmm_m128 --workfile concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.prove.ST1.ST2.z3 ; z3 concrete_instances/memory-variants/psrlq_xmm_m128/instructions/psrlq_xmm_m128/psrlq_xmm_m128.prove.ST1.ST2.z3
```
Expected result: `unsat`


### Artifacts for "Reported Bugs"
- [Bug reported in Intel](https://software.intel.com/en-us/forums/intel-isa-extensions/topic/773342): Refer Section 4.1 --> Instruction Level Validation --> Inconsistencies Found in the Intel Manual
- Bug reported ([Report1](https://github.com/StanfordPL/stoke/issues/983) & [Report2](https://github.com/StanfordPL/stoke/issues/986)) & corresponding pull requests ([Request1](https://github.com/StanfordPL/stoke/pull/984) & [request2](https://github.com/StanfordPL/stoke/pull/985)) accepted in Stoke project: Refer Section 4.2


### Artifacts for "Applications" (Refer Section 5)
In this section we presented some applications to demonstrate that
our semantics can be used for formal reasoning of x86-64
programs for a wide variety of purposes. Here, we present the artifacts for
the applications presented in Sections 5.2, 5.3 and 5.4

### Section 5.2. Program Verification
In this sub-section, we prove the functional correctness of the sum-to-n program.

Working Directory: /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/sum_to_n_32_bit

The directory structure:
- test-spec.k: The actual specification file that is fed to the verifier. The specification has two parts:
             the top-level specification and the loop invariant.
- runlog.txt : The pre-populated output of the verifier.
- run.sh     : Script to run the prover.

#### Reproducing the runlog.txt
```
cd /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/sum_to_n_32_bit
./run.sh
```

#### Note
At the end of the section, we mentioned the time taken by the verifier to be
~1 min, which can be reproduced by the above command or can be cross-checked in the
runlog.txt file.


### Section 5.3. Symbolic Execution
In this section we  demonstrate how the symbolic execution
capability can be used to find a security vulnerability.


**Working Directory:** /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/safe_addrptr_32

The directory structure:
- **test-spec.k:** The actual specification file that is fed to the verifier.
- **runlog.txt :** The pre-populated output of the verifier.
- **run.sh     :** Script to run the prover.
- **path_condition.z3 :** The z3 query that need to be solved in order to get the input triggering the
                    security vulnerability.

#### Interpretation of the runlog.txt and reproducing the vulnerability
1. At line number 87 of runlog.txt, we obtain the path condition when the control reaches L2 when r >= a (refer figure 4(a)). We encode this condition as a Z3 formula in path_condition.z3 and `AND` it with the condition for a + b to overflow. The resulting formula is checked for satisfiability. We mentioned all the details in path_condition.z3 as comments and request the reviewer to have a look at it.
2. Execute `z3 path_condition.z3` to reproduce the inputs triggering the vulnerability.

#### Reproducing the runlog.txt
```
cd /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/safe_addrptr_32
./run.sh
```

### Section 5.4. Translation Validation of Optimizations
In this section, we validated different optimizations of the "popcount" program, by checking the equivalence between the optimized programs.
For the artifact evaluation, we will demonstrate the optimization made by Stoke super-optimizer (as this is the most interesting among the other optimizations). If interested, we will be happy to provide details about the equivalence of other optimizations.

We symbolically execute the un-optimized popcount program and stoke-optimized program individually and compares their return values (i.e., the symbolic expression of the %rax
register value) using Z3.

**Working Directory:** /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/popcnt_loop

The directory structure:
- **test-spec.k:** The actual specification file, of the un-optimized program, that is fed to the symbolic executor.
- **runlog.txt :** The pre-populated output of the symbolic executor.
- **run.sh     :** Script to run the symbolic executor.
- **test.z3    :** The z3 query that need to be solved in order to check the equivalence between the un-optimized and optimized programs.

#### Interpretation of the runlog.txt
1. At line number 8710 of runlog.txt, we obtain the K expression representing the symbolic output stored in %rax for the un-optimized program.
2. ...

#### Reproducing the runlog.txt (take ~23 mins)
```
cd /home/sdasgup3/Github/binary-decompilation_programV_working/x86-semantics/program-veriifcation/popcnt_loop
./run.sh
```

## Artifacts for "Reported Numbers/Claims"
1. In Line 12-13, we mentioned "... This totals 3155 instruction variants, corresponding to 774
mnemonics ..."

2. In Line 51, we mentioned "Heule et al. ...,  but it covers only a fragment (∼47%) of all instructions ..."

3. In Line 66-70, we mentioned "Goel et al. ...  only a small fragment (∼33%) of all user-level instructions..."

4. In Line 136, we mentioned "Our formal semantics is publicly available ..."
    - Public [Github Repo](https://github.com/kframework/X86-64-semantics)
5. In Line 143, we mentioned "It consists of 996 mnemonics, and each mnemonic admits several variants,"

6. In Line 185, we mentioned "Remill  updates the flag with 0 .. Radare  keeps it unmodified."

7. In Line  139, we mentioned "Strata .. 1905 instruction variants (representing 466 unique mnemon- ics) of the x86-64 Haswell ISA"

8. In Line 323-326, we mentioned "To leverage previous work as much as possible, we took the semantic rules for about 60% of the instructions
in scope from the formal semantics in Strata"

9. In Line 351-352, we mentioned "We then manually added K rules for the remaining 40%
of the target instructions..."


10. In Line 363-366, we mentioned "... compared against the semantics defined in the Stoke project for about 330 instructions that were omitted in Strata..."


11. In Line 574-576, we mentioned "For each instruction, we converted the SMT formulas that Strata provides to a K specification using a simple script (∼500 LOC)."

12. In Line 641-644, we mentioned "the original Strata-provided formula for shrxl %edx, %ecx, %ebx consists of 8971 terms (including the operator symbols), but we could simplify it to a formula
consisting of only 7 terms"

13. In Line 814-817, we mentioned "Stoke  contains manually written semantics for ∼1764
x86-64 instruction variants, a large fraction (81%) of which
is also supported by Strata"

14. In Line 819-821 we mentioned "Moreover, these manually written
formulas are based on a similar model of the CPU state to
ours, which makes it easier to compare them against ours by"


15. FLowchart for instruction support.
16. Porting (https://github.com/sdasgup3/binary-decompilation/wiki/Handling-Register-Instructions)

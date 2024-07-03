# Chialisp Workflow

I didn't find a good description of a workflow for writing Chialisp, so I created one.  I think it is going to work for me, 
and I'm posting it here in case it is of benefit to others.  The key thing is that I want to have a collection of tests 
that I can run over and over again during development to make sure I don't break anything that had previously been working.
Also, I like to do test-first development when I can.

The workflow described here depends on both `clvm_tools_rs` and @rigidity's handy `curry` utility being installed.
They are available here:  
https://github.com/Chia-Network/clvm_tools_rs  
https://github.com/rigidity/curry

These test scripts are written in bash and have been tested under Ubuntu 22.04.

The workflow starts with editing your Chialisp code.  I use VSCode with the Chialisp extension.  Or `vim` for small fixes.  
Or whatever.

## Compiling code

I created a simple script to compile code to produce all three useful outputs: the clvm, the hex and 
the hash root: `mod_compile.sh`.

This assumes a default include directory `include/` under the source directory.  The output files end up in a 
subdirectory `output/`.

Should you need it, there is a matching clean script: `mod_clean.sh`.

## Testing

The useful part comes in building automated tests.  The script for that is `mod_test.sh`.  

The test script uses files in a `tests/` subdirectory.  For simple tests, there are either two or three files per test:
```
  tests/<sourcefilename>.clsp/<testname>.curry (optional)
  tests/<sourcefilename>.clsp/<testname>.env
  tests/<sourcefilename>.clsp/<testname>.expected
```

For each test: 
- if there is a curry file, the test script will first curry those parameters into the puzzle
- the script will `brun` the puzzle with the test's `.env` environment (solution)
- the script will compare compare the output with the `.expected` output, and record an error if they don't match

For more complex tests, `mod_test.sh` can generate the input files using Chialisp test generator scripts:
```
  tests/<sourcefilename>.clsp/<testname>.gen_cur.clsp (generates the .curry file)
  tests/<sourcefilename>.clsp/<testname>.gen_env.clsp (generates the .env file)
  tests/<sourcefilename>.clsp/<testname>.gen_exp.clsp (generates the .expected file)
```
These generator scripts are normal Chialisp module files that do not accept an environment:
```
(mod ()
  ; generator code here
)
```

The script can optionally run a single test if given a testname, or will run all tests for a particular sourcefile if 
the testname is omitted.  That way, you can quickly run the entire suite of tests for a sourcefile.

It is worth noting that the curried version of the puzzle (if applicable) and the output of running the puzzle with the 
environment given for each test case are left in `output/` directory after the tests have completed.  If a test case 
doesn't pass, you can review these for clues toward debugging it.

## Debugging

Inevitably, the test will fail and you will need to debug it.  I have a script for that: `mod_debug.sh`.

The debug script uses the same test cases as `mod_test.sh`, except that the script shows you the expected output 
and the actual output, and shows you any print statements that may have been hit while the debugger was running.

It is not as good as stepping through a program, but having an easy way to quickly add some print statements, rerun 
a known-failing test case, and view the output is hugely valuable.

### Print Statements

To add print statements to your code, first make sure the .clsp module file has this line, to use the 2023 compiler:  
`(include *standard-cl-23*)`

You will also need to include this "function definition" for the print function:  
`(defun print (l x) (i (all "$print$" l x) x x))`

How does the print statement work?  It accepts two parameters, the first of which is a list (`l`), and the second (`x`)
can be either an atom or a list.  It does get executed as a part of your program, so it is worth knowing the return
value from this function is `x`.  Both the list and the value get printed.

Here is an example that includes some print statements:
```
(mod (VAL1 val2)
  (include *standard-cl-23*)
  (defun print (l x) (i (all "$print$" l x) x x))
  
  (* (print (list "VAL1:") VAL1) (print (list "val2:") val2))
)
```

The debugger output will look like this:
```
---
- Print: "((\"val2:\") 3000)"
- Print: "((\"VAL1:\") 1000)"
- Final: "3000000"
  Final-Location: "add.clsp(6):4(1):1"
```

## Example

I have included an example module and two tests to demonstrate these scripts.

In the `example/` directory there is a module `add.clsp`.  If you change to that directory, you should be able to:  
`mod_compile.sh add.clsp`  
`mod_test.sh add.clsp test1` (run one test)  
`mod_test.sh add.clsp` (run all tests)  
`mod_debug.sh add.clsp test2`  
`mod_clean.sh add.clsp`


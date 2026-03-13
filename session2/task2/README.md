# Task 2: More GDB Commands

The program in `stats.c` computes summary statistics for up to 10,000 numbers
stored in a text file, named on the command line. It is supposed to ignore
negative numbers or non-numeric data when performing its calculations.

## Initial Observations

Some sample data files are provided in the `data` subdirectory. The outputs
expected when the program is run with these files are summarized below.

| File       | Expected output                  |
| -----------| ---------------------------------|
| test1.txt  | Mean of 0.540, std dev of 0.258  |
| test2.txt  | Mean of 0.525, std dev of 0.287  |
| test3.txt  | Mean of 0.525, std dev of 0.287  |
| test4.txt  | "No valid data" error message    |

If the program is run without a command line argument, it should print an
appropriate usage message and terminate. If the command line argument is not
the name of an accessible file, the program should display a suitable error
message and terminate.

1. Compile the program by entering `make` in the terminal.

2. Run the program to check whether it meets all of the expectations outlined
   above. Make a note of your findings.

   No file: Correct
   File 1: Correct
   File 2: Incorrect
   File 3: Incorrect
   File 4: Incorrect

## Function Breakpoints

1. Load the program into GDB, then establish breakpoints just inside the
   `read_data()` and `mean_value()` functions.

   Note that you don't need to specify line numbers to do this! You can just
   provide function names after the `break` command:

       b read_data
       b mean_value

   Inspect these newly-created breakpoints with

       info b

2. Run the program on a data file, with

       run data/text1.txt

   Execution should pause just inside `read_data()`.

3. Use `c` to resume execution. This should take you to the next function
   breakpoint, inside `mean_value()`.

   Use `c` again to resume execution. The program should run to completion,
   without further interruptions.

   **Stay in the debugger for the next part of the task.**

## Conditional Breakpoints

Let's use GDB to investigate whether the program handles negative numbers or
non-numeric input correctly.

1. Rerun the program within GDB like so:

       run data/test2.txt

   Execution should pause just inside `read_data()`.

2. Next, establish a **conditional breakpoint** on line 24, which will trigger
   only if the variable `value` is negative:

       display value
       b 24 if value < 0.0

   Notice the use of `display` here, to ensure the variable is displayed
   automatically when the program next pauses.

   Inspect the current breakpoints with `info b`.

3. Enter `c` to resume execution. You should see the program pause when
   `value` is -0.6. If you enter `n`, you will see that this negative number
   is incorrectly assigned to the array `data`.
   
   -0.6 is never read?

4. Enter `finish` or `fin` to complete execution of `read_data()`. This
   will put you back in `main()`. If you print the value of `size` you will
   see it is 5, but it should be 4 (because the program should have ignored
   the negative value).

   If you examine the code, you'll see that the program does not test whether
   `value` is negative. It also does not test whether `sscanf()` parsed the
   string successfully. Instead, it always copies `value` into `data` and
   always increments `size`.

   **Stay in the debugger for the next part of the task.**

## Changing Variables & Calling Functions

We ought to also check whether `mean_value()` is computing values correctly,
independently of anything that happens in `read_data()`. We can do this
quite easily from within GDB.

1. Delete all the existing breakpoints by entering `del`, then restart the
   program with

       start

   Notice that no command line argument has been specified here. A filename
   isn't needed because you will be bypassing `read_data()` entirely!

2. Put two numbers inside `data` and set `size` to 2, using these commands:

       set var data[0] = 1.0
       set var data[1] = 1.5
       set size = 2

3. Now invoke the `mean_value()` function like so:

       call mean_value(data, size)

   You should see a result of 1.25 displayed.

## Fixing The Problems

The previous experiments established that `mean_value()` looks OK and the
faults seem to lie in `read_data()`.

Make the necessary changes to this function, then try running the program on
the data files to verify that the expected behaviour is now seen.

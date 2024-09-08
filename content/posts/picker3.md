---
title: "PicoCTF: Picker III"
summary: Exploiting global values!
date: 2024-09-04
tags: ["picoctf"]
categories: ["write-up", "reverse_engineering"]
draft: false
---

# Investigating the Source Code
In this CTF, we are given the source code for the Python script running on the server. This file is much more involved than Picker I/II, so let's break down how the code works.

## Code Breakdown

### Getting User Input
The script will run in a loop, waiting for user input. It calls various functions given user input:
```python3
while(USER_ALIVE):
  choice = input('==> ')
  if( choice == 'quit' or choice == 'exit' or choice == 'q' ):
    USER_ALIVE = False
  elif( choice == 'help' or choice == '?' ):
    help_text()
  elif( choice == 'reset' ):
    reset_table()
  elif( choice == '1' ):
    call_func(0)
  elif( choice == '2' ):
    call_func(1)
  elif( choice == '3' ):
    call_func(2)
  elif( choice == '4' ):
    call_func(3)
  else:
    print('Did not understand "'+choice+'" Have you tried "help"?')
```

### Handling User Input
The code holds a "table" mapping numbers 1 through 4 to different functions, which roughly looks like:

| Input Number | Function in Code  |
|:------------:|:-----------------:|
| 1            | print_table()     |
| 2            | read_variable()   |
| 3            | write_variable()  |
| 4            | getRandomNumber() |

After parsing through the various functions, the interesting function is the `write_variable()` function:

```python3
def write_variable():
  var_name = input('Please enter variable name to write: ')
  if( filter_var_name(var_name) ):
    value = input('Please enter new value of variable: ')
    if( filter_value(value) ):
      exec('global '+var_name+'; '+var_name+' = '+value)
    else:
      print('Illegal value')
  else:
    print('Illegal variable name')

```

The key line is `exec(f'global {var_name}; {var_name} = {value}')`, which executes code to create or modify a global variable with the specified name and value.

## Global Variable Implications

The `global` keyword allows modification of global variables from within functions. In Python, global functions and variables are accessible throughout the module. By using `write_variable()`, we can modify any global variable, including functions.

Here are some functions in the global scope of the file:

| Function Name    | Side Effect                         |
|:----------------:|:-----------------------------------:|
| win()            | outputs flag                        |
| read_variable()  | outputs global scoped variable                      |
| write_variable() | writes a value to a global variable |
| getRandomNumber()  | outputs '4'                                    |

Since we can modify any global value, we can change `getRandomNumber()` to point to `win()`, which will allow us to obtain the flag.

# Getting the Flag

After connecting to the server, let's run the `write_variable()` function, and then run our modified `getRandomNumber()` function:

| Input Number | Function in Code  |
|:------------:|:-----------------:|
| 1            | print_table()     |
| 2            | read_variable()   |
| 3            | write_variable()  |
| 4            | getRandomNumber() |

```
==> 3
Please enter variable name to write: getRandomNumber
Please enter new value of variable: win
==> 4
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x37 0x68 0x31 0x35 0x5f 0x31 0x35 0x5f 0x77 0x68 0x34 0x37 0x5f 0x77 0x33 0x5f 0x67 0x33 0x37 0x5f 0x77 0x31 0x37 0x68 0x5f 0x75 0x35 0x33 0x72 0x35 0x5f 0x31 0x6e 0x5f 0x63 0x68 0x34 0x72 0x67 0x33 0x5f 0x32 0x32 0x36 0x64 0x64 0x32 0x38 0x35 0x7d
```

We got the flag! Let's clean it up.

```python3
flag = "0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x37 0x68 0x31 0x35 0x5f 0x31 0x35 0x5f 0x77 0x68 0x34 0x37 0x5f 0x77 0x33 0x5f 0x67 0x33 0x37 0x5f 0x77 0x31 0x37 0x68 0x5f 0x75 0x35 0x33 0x72 0x35 0x5f 0x31 0x6e 0x5f 0x63 0x68 0x34 0x72 0x67 0x33 0x5f 0x32 0x32 0x36 0x64 0x64 0x32 0x38 0x35 0x7d"

flag = flag.split(" ")
flag = "".join(chr(int(word, 16)) for word in flag)

# picoCTF{7h15_15_wh47_w3_g37_w17h_u53r5_1n_ch4rg3_226dd285}
```

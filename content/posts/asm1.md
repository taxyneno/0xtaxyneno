---
title: "PicoCTF: ASM1"
summary: "Counting in assembly."
date: 2024-09-08
tags: ["picoctf", "asm"]
categories: ["write-up", "reverse_engineering"]
draft: false
---

# Problem Overview
We are given an assembly file. We are asked to output what `asm1(0x2e0)` returns.

Let's take a look at the assembly file:

```assembly
asm1:
    <+0>:	push   ebp
    <+1>:	mov    ebp,esp
    <+3>:	cmp    DWORD PTR [ebp+0x8],0x3fb
    <+10>:	jg     0x512 <asm1+37>
    <+12>:	cmp    DWORD PTR [ebp+0x8],0x280
    <+19>:	jne    0x50a <asm1+29>
    <+21>:	mov    eax,DWORD PTR [ebp+0x8]
    <+24>:	add    eax,0xa
    <+27>:	jmp    0x529 <asm1+60>
    <+29>:	mov    eax,DWORD PTR [ebp+0x8]
    <+32>:	sub    eax,0xa
    <+35>:	jmp    0x529 <asm1+60>
    <+37>:	cmp    DWORD PTR [ebp+0x8],0x559
    <+44>:	jne    0x523 <asm1+54>
    <+46>:	mov    eax,DWORD PTR [ebp+0x8]
    <+49>:	sub    eax,0xa
    <+52>:	jmp    0x529 <asm1+60>
    <+54>:	mov    eax,DWORD PTR [ebp+0x8]
    <+57>:	add    eax,0xa
    <+60>:	pop    ebp
    <+61>:	ret
```

Some quick observations:
1. It looks like we are using Intel syntax
2. It looks like we just need to run through some conditional checks for the given input value. We should focus on jumping to the right labels.

# Walking Through the Code

## First Comparison
Our input, `0x2e0`, is 736 in decimal. I will leave some notes on converted hexadecimal values as we work through the code.

```assembly
;; input value to asm1: 736
asm1:
    <+0>:	push   ebp
    <+1>:	mov    ebp,esp
    <+3>:	cmp    DWORD PTR [ebp+0x8],0x3fb ;; 1019
    ;; if the input value is > 1019, jump to 0x512
    <+10>:	jg     0x512 <asm1+37>
```

In this first block, we check if the value on the stack is greater than 1019. With our input value of 736, we don't jump. Instead, we keep walking through the assembly.

## Second Comparison
```assembly
    <+12>:	cmp    DWORD PTR [ebp+0x8],0x280 ;; 640
    ;; if the input value is not equal to 640, jump to 0x50a
    <+19>:	jne    0x50a <asm1+29>
```

Here, we check if the input is not equal to 640. It actually is not, so let's jump to `0x50a`.

## EAX Shenanigans
```assembly
    <+29>:	mov    eax,DWORD PTR [ebp+0x8] ;; eax = 736
    <+32>:	sub    eax,0xa ;; eax = 726
    <+35>:	jmp    0x529 <asm1+60>
```

Here, we move the input value, 736, on the stack into the `eax` register.

Then, we subtract 10 from `eax`, leaving its value at 726. Let's jump to `0x529`.

## Return

```assembly
    <+60>:	pop    ebp
    <+61>:	ret
```

Here, we just pop the base pointer and return. After converting 726 from the eax register to hexadecimal, we get `2D6`.

The CTF asks us to input the value in the format `0x<RETURN_VALUE>`, so the flag is just `0x2D6`.

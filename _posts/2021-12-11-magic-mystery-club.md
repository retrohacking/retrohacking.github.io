---
layout: post
title: "Writeup Compfest13: Magic Mystery Club"
categories: BinaryExploitation
tags: Hacking BinaryExploitation
---
This has been the challenge that made me suffer the most at the moment. We are provided with a binary in ELF (so executable on Linux). It was proposed as a Reverse Engineering challenge so our objective was to find the correct input that at all the effects is the flag we're searching for.
<!--excerpt-->
So our starting point is disassembling using a tool like Ghidra, Ida, or even gdb. For my examples I'll use Ghidra.

**Let's start!**

![](/img/magicmysteryclub/main.png)

We can see that already the main function provides us a lot of important information:

- it requires an input: the lenght can be seen in che memcmp command, line 29: it compares 98 bytes, so our flag will have that lenght.

- the input we insert will be processed, letter by letter and each letter will be replaced with the value returned by the function cast_magic

- the memcmp compares our input (properly modified) with something that is in the memory, at the address defined by *flag* plus its offset. So we can find what our binary is searching right in the memory. 

- if we click on flag from Ghidra or IDA we won't get any info since the addresses have not been initialized... <sub>spoiler: it will be initialized with the call of init, line 15</sub>

Let's proceed with order: how will the program replace our input? Let's open the function cast_magic:

![](/img/magicmysteryclub/castmagic.png)

Here we find a random value, a **useless**  shifted_rand an then two variables are calculated basing on the return value of fun1 and fun2. In the end state (that is initialized as 0x42) will be multiplied by 3 and the bitwise XOR between the two variables is returned. You can think that it's easy until now? Well fun1 and fun2 call a series of other function that just operate on the second parameter creating a mathematic expression. The value returned by the first function can be simplified and becomes 19 + the integer value for the letter passed to the function.

The second function becomes an expression that is more difficoult to simplify. Going with logic it would be impossible to crack the application if it depended on a random value; so Wolfram Alpha came and helped us showing the simplification of the whole expression to a simple *-55*.

So in the end the second variable is equal to *state-55*.

Now we have the actual effect of the two functions: one thing is missing. What we should compare our input to; so as I've previously said we can find what we're searching in the function *init*:

![](/img/magicmysteryclub/init.png)

Here we have all the values that at the moment of execution will be copied to the actual address of memory of the flag. What we need of all this values are the ones starting from the 52nd Byte to the 150th (Remember we have 98 Bytes of flag).

Also remember to adapt the order basing to the endianness.

<sub>Suggestion: is it easier to obtain what we want with gdb, in particular using the command *"x/98x < flag + offset to the 52nd Byte>"*</sub>

Now we are ready to crack this application: we'll just have to write a script as the following:

```python
state=int("0x42",16)
required_input="5ded7bdcf237ddf80f09d5fead4683daf9681eb76368497f8c488087ebfb019b7dd16fc0d9e8dedb3d1113f999aeb80a0958f74dacb4789782b4bc8d0a2d099c8af769d5e8e7c4c839ce18f8af59bd2dcc8ef7b56c83495380b692b61fda3f94431f"
flag=[None]*98
flag_len=len(flag)
rindex=0
new_req_val=[int(str(required_input[i])+str(required_input[i+1]), 16) for i in range(0, len(required_input), 2)]

def fun1(letter):
    return (19+letter)%256

def fun2(state):
    return (state-55)%256

def cast_magic(letter,state):
    return fun1(letter) ^ fun2(state)


for i in range (flag_len):

    for letter in range (33, 127):
        found_value=cast_magic(letter, state)
        if (found_value==new_req_val[i]):
            flag[i]=chr(letter)
            break
        if letter==126:
            print("There has been a problem with the character at the index: "+ str(i))
            exit()
    state*=3


print  ("".join(flag))
```

As you can see I've chosen to work with integer values to avoid casting to hex and again to int and/or char.

Basically, I've also converted the hexadecimal bytes of the value we've obtained from *flag* to the relative integer value.

I've rewritten the application with the simplification applied in fun1 and fun2 and then written a little for cycle to bruteforce our flag: we cycle in range 33-126 because are the decimal corrispective of the printable ASCII characters. With this cycle we're going to pass every number from 33 to 126 to the function and then compare the return value of cast_magic with the same index's byte from the requested value. If the input number produces the same output as the expected value, the input is converted to char and added to the flag, that will be printed at the end of the execution.

Here is the result!

![](/img/magicmysteryclub/flag.png)

<sub>Flag: COMPFEST13{n3Ver_Tru5t_M4tHemAg1cKal_tR1cK5s_n0BoDY_tH0u6hT_No_0ne_W0ulD_n0t1c3_4nYw4Y_98f66ab185}</sub>

# Project: Program Flow Manipulation

## Description 

The project explores program exploitation and manipulation techniques, specifically those dealing with buffer overflow. Completing the project required knowledge in _gdb_, assembly language, as well as use of utilities such as _hex2raw_. 

When a program does not have security measures in place, like a stack canary or a shadow stack, it is very vulnerable to exploits. One of those exploits is writing over the expected length of a string that is inputted by the user: buffer overflow. When this happens, the buffer can be curated in a way that influences the program's flow, by overwriting return addresses stored on the stack, for example. This is the focus of this project.

The project was completed as part of CS 261: Machine Organization course at University Of Illinois Chicago, in Fall 2025. The author of the project is Adam T. Koehler, PhD, the instructor of the course.  

 

## Components 

The project worked with multiple files: 

   - `ctarget`: An executable program vulnerable to code-injection attacks 

   - `rtarget`: An executable program vulnerable to return-oriented-programming attacks 

   - `cookie.txt`: An 8-digit hex code used as a unique identifier in the attacks. 

   - `farm.c`: The source code of the target’s “gadget farm”, used in generating return-oriented programming attacks. 

   - `hex2raw`: A utility to generate attack strings. 

 

 

 

 

 

 

 

## Flow of the project 

The project had 4 phases that required crafting the right exploit to reach a target function within executables ctarget or rtarget.  

Both programs have the same function that asks user for an input string: 

```
unsigned getbuf() { 
 	char buf[BUFFER_SIZE]; 
 	Gets(buf); 
 	return 1; 
}
```

As function _getbuf()_ is unprotected against stack manipulation, as soon as the buffer gets over the limit (BUFFER_SIZE), it starts overwriting contents on the stack. 


### Phase 1 

The first phase required overwriting the return address that the program uses after executing `getbuf()` with an address to the first assembly instruction of the target function touch1(). This was done following these steps: 

1. Determining the memory address of `touch1()` by setting a breakpoint at `getbuf` and running _ctarget_ in gdb, and then using the `disas` command:

   <img width="1024" height="310" alt="image" src="https://github.com/user-attachments/assets/a0111ecb-3865-41a3-9f9b-783f704d8f00" />

3. Placing the bytes of the memory address in reverse form (little endian) in a .txt file in a format that is readable by hex2raw

   <img width="302" height="101" alt="Screenshot 2025-12-16 203403" src="https://github.com/user-attachments/assets/d25e6d0d-355b-477c-9d01-95f6e7ec818b" />

   _Note: the 24 zeroes account for the buffer size limit. Anything written after them overwrites the return address after Gets()_

  
5. Input redirecting the .txt file through hex2raw to generate a ‘raw’ vector, which when used as input to ctarget will successfully alter the program flow to make it jump to function touch1()


Diagram for Phase 1

<img width="1024" height="506" alt="image" src="https://github.com/user-attachments/assets/b86733f1-6cc5-4866-bfd8-85ef04118b06" />

## Phase 2 & 3
The next two phases followed the same idea, but this time it was required to pass a hex code from `cookie.txt` as an argument to be checked by the target function. Here, byte encodings of assembly instructions had to be injected to achieve this result.

```
void touch2(unsigned val) {
    vlevel = 2; /* Part of validation protocol */
    if (val == cookie) {
        fprintf(stdout,"Touch2!: You called touch2(0x%.8x)\n", val);
        validate(2);
    } else {
        fprintf(stdout,"Misfire: You called touch2(0x%.8x)\n", val);
        fail(2);
    }
exit(0);
}
```

1. Reaching touch2() with the cookie required moving it into %rdi (function argument passing register). This could be done by determining the byte encoding of instruction `mov $0x6c6fcfb8, %rdi`, which is `48 c7 c7 b8 cf 6f 6c` followed by `c3`, which is the encoding for `ret` instruction. All of these bytes needed to be inputted into the buffer. 6c6fcfb8 is the hex code checked by `touch2()`.
2. Instead of overwriting the return address with the address directly to target function as in Phase 1, the program instead needed to be directed to the top of the stack to execute the instruction injected in the step above.
3. Encodings for `nop` and another `ret` instruction needed to be injected to account for 16 byte alignment ahead of jumping to `touch2()` to avoid segmentation fault.

Diagram for Phase 2

<img width="1024" height="506" alt="image" src="https://github.com/user-attachments/assets/cd2fabbd-feac-40f0-b93d-964731749cee" />






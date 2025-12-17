# Project-Program-Flow-Manipulation

Description 

The project explores exploitation and manipulation techniques, specifically those dealing with buffer overflow. Completing the project requires knowledge in gdb, assembly language, as well as use of utilities such as hex2raw. 

When a program does not have security measures in place, like a stack canary or a shadow stack, it is very vulnerable to exploits. One of those exploits is writing over the expected length of a string that is inputted by the user: buffer overflow. When this happens, the buffer can then be curated in a way that influences the program's flow, by overwriting return addresses, for example. This is exactly what this project focused on. 

The project was completed as part of CS 261: Machine Organization course at UIC in Fall 2025. It was created by Adam T Koehler, PhD, the instructor of the course.  

 

## Components 

The project worked with multiple files: 

    ctarget: An executable program vulnerable to code-injection attacks 

    rtarget: An executable program vulnerable to return-oriented-programming attacks 

    cookie.txt: An 8-digit hex code that you will used as a unique identifier in the attacks. 

    farm.c: The source code of the target’s “gadget farm”, used in generating return-oriented programming attacks. 

    hex2raw: A utility to generate attack strings. 

 

 

 

 

 

 

 

## Flow of the project 

The project had 4 phases that required crafting the right exploit to reach a target function within executables ctarget or rtarget.  

Both programs have the same function that asks user for an input string: 

1.  unsigned getbuf() 
2.  { 
3. 	char buf[BUFFER_SIZE]; 
4. 	Gets(buf); 
5. 	return 1; 
6.  } 

As function Gets() is unprotected against stack manipulation, when the buffer is over the limit (BUFFER_SIZE), it can overwrite contents on the stack. 

 

gdb was used as a pivotal tool in parsing the executable files. 

 

Phase 1 required overwriting the return address that the program uses after executing Gets() with an address to the first assembly instruction of the target function touch1(). This was done following these steps: 

# Computer Security (CSE 484), Sp21
Pieter Benjamin (pieterb@uw.edu)

# Introduction to Security
- How do systems fail?
  - Reliability (accidental failures)
  - Usability (operating mistakes by users)
  - Design and goal oversights
  - Computing in the presence of an adversary
- What does it mean for a system to be secure?
  - Not a binary relation (*this* system provides *these* properties in *this* context)
  - Who are the stakeholders?
  - What are the assets to protect?
  - What are the threats to the assets?
  - Who are the adversaries and what are their resources?
  - What is the security policy?
- This class has two main themes
  1. Security mindset (think critically about designs and how they can fail)
  2. Current (2021) web security concepts
- The first theme is a much more valuable takeaway because technologies will inevitably change but critical thinking is always valuable
# Software Security
- The most important thing to learn is how to think about a system in terms of an adversary (threat modeling)
  - Assets (what are we protecting/how valuable)
  - Adversaries (who might attack, why, how strong)
  - Vulnerabilities
  - Threats (reasonably expectable attacks)
  - Risk (how much danger are the assets in)
  - Possible defenses
- Who might benefit from the system being exploited? Who would be harmed?
- What *is* security anyway?
- Common goals: CIA (Confidentiality, Integrity, Availability)
  - Another variant is Control, Authenticity, and Utility
- Confidentiality (privacy)
  - Concealment of information
- Integrity
- Availability
- No such thing as perfect security
- Attackers have limited resources and you want to raise the cost of attack past their tolerable threshold
- Threat Modelling (Example: Electronic Voting)
  - First step - how does the system work without an adversary?
    - First phase: workers show up and load "ballot definition files" onto the machine (controls what the voter sees)
    - Second phase: Voters show up and start voting (authenticate themselves and are given a "voter token")
    - Third phase: Voter inserts card to machine, gets verified, votes, and presses a "cast my ballot" vote (choice is then saved, token is nullified)
  - What do you think are the security goals of the electronic voting system described in class and shown above? What would be some of the assets that must be protected?
    - Goals: each voter is authenticated accurately and only gets access to a single voter token which gives them access to a correct ballot definition file
    - Assets: All the voter tokens themselves, the ballot definition files, the memory on the voting machines
  - Who are the adversaries who might try to attack this electronic voting system? What might be the attacker’s goals? What potential threats or vulnerabilities do you see?
    - Political opponents, foreign powers, trolls, incompetence (handing out used voter cards, damaging files, uploading wrong information at any step), spoofed voting centers
  - The source code was revealed for the system and some concrete vulnerabilities were discovered
    - No way to know what software was being run on the system (hacker could upload anything)
    - There is an access panel on the side of the machine which gave access to all ports (they published a picture of the key on the website AND PEOPLE CAN MAKE COPIES OF KEYS FROM PICTURES)
    - Ballot definition files are not authenticated, which means that votes cast for X can be redirected to Y
    - A voter could clone a voter token and then vote multiple times
- Approaches to Security
  - Prevention
    - Stop an attack from happening
  - Detection
    - Can we figure out during/after an attack that something is wrong?
  - Response and Resilience
    - What do we immediately respond with as an attempt to stop the hacker
- Securing a system requires a whole-system view (you are only as protected as your weakest link)
- Attackers have an asymmetric advantage - they only need to find a single flaws
- Defense in depth - basically redundancy
- Even after choosing a decent system, you can still whiff the implementation
- At large scale, different parties may have different goals/optimizations which might be misaligned  
## Buffer Overflows
- When taking in user input, if you accept more than you allocated space for they may be able to overwrite your stack frame
  - Doing so carefully can hand control of the program over to the potentially (likely?) malicious user
```c
void mycopy (char *input) {
  char buffer [512];
  int i;
  for (i = 0' i <= 512; i++)
    buffer[i] = input[i];
}
```
- The above example is off by one, and one byte likely will not change the return address but can change the pointer to the previous stack frame that is stored next to the return address (in a little endian architecture)
  - In a little endian architecture this can make a difference of 256 addresses
  - 0x00...000[...] ... [buf][FP][RET][str][caller's frame]...[...]0xFF...FFF
  - Above, the stack grows down to the left. The only arg to the function (str) comes before the return address, which is followed by buffer. If an attacker manages to go past the bounds of the buffer into the return address, they can change an entire byte of data in FP.

## Format Strings
- Enable a variable number of arguments
```c
// Proper use (static string)
int foo = 42;
printf("foo = %d in decimal, %X in hex", foo, foo);
// Improper use (variable string)
char buf[14] = "Hello, world!";
printf(buf);
// Should've used printf("%g", buf)
```
- Lower level functions such as printf can advance the stack and get hands on with the OS - you *really* don't want these to be messed up
```c
// Proper/well formed
printf("Numbers: %d, %d", 1, 2);
// Improper/exploit
printf("Numbers: %d, %d");
```
- In the above example, the proper example will "fetch" the numbers 1 and 2 off the stack and print them. The improper one will fetch some random data off the stack frame of printf - privilleged information we don't want the user to have their hands on!
- Extending this example:
```c
char buf[16] = "here is a string: %s";
printf(buf);
```
- The "%n" format string will look at the number of characters written by printf, and write them to the address stored at the next arg/element on the stack.
- Historically used for reasons such as measuring how much you've written to a network.
```c
// Writes 14 into myVar
printf("Overflow this!%n", &myVar);
// Writes somewhere bad, could even overwrite information
// into printfs internal stack frame
char buf[16] = "Overflow this!%n";
printf(buf);
```
- Good pausing point to remind the reader that different compilers/architectures/operating systems may see different effects
- Can mark the regions of memory allocated as writeable memory as non-executable to defend against buffer overflow  
  - Good general policy, prevents code injection
  - Attackers can still corrupt the stack, function pointers, or critical data on the heap
  - Can also jump to already existing poorly behaved code
- return-to-libc
  - Overwrite saved RET with address of a library routine and arrange stack to look like arguments
- Instead of jumping to the start of a function, we can just overwrite the RET to an arbitrary instruction
  - Can chain RETs together which is a Turing-complete language (Shacham et al.)
  - Called return oriented programming
  - Basically jump to a place, execute a small snippet, ret, snippet, etc.
- Another defense is to add "canaries" to the stack frame which have unique values (such as a null terminating byte to stop strcpy) which are checked at the start/end of the execution of the method
## [Smashing the Stack for Fun and Profit](https://avicoder.me/2016/02/01/smashsatck-revived/)
- Programs are split into three parts
- Text
  - Fixed by the program, includes instructions and read-only data
  - Cannot write to this portion of memory without a segmentation fault
- Data
  - Initialized and uninitialized data
  - Static variables
  - New memory added between the data and stack segments
- Stack
  - EBP is the frame pointer on intel - which keeps track of the top of the current frame
  - Local variables and parameters can be stored with relative distances from the frame pointer
  -
## Exploiting Format String Vulnerabilities

# Cryptography

## Guest Lecture: Genie Gebhart

## Symmetric Encryption

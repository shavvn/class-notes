### Mar, 28
#### P6 Virtual Memory
ASIDs
How do we tell  diff between processes generating same VA?
A: 0x0000ABCD-> Physical Page 37
B: 0x0000ABCD-> Physical Page 5
...
Z: 0x0000ABCD-> Physical Page 323294

Solutions:

- define it not to be a prblem e.g. SASOS
- brute force solution: flush the TLB everytime on context switch
- use ASID (usually 8 to 16 bits)
	- extend TLB with ASID, it looks like this:

| ASID | VPN | FPN | etc|
|------|-----|-----|----|
|xxxxxx|yyyyy|zzzzz|RWX, valid, sharing...|


- if you want more processes, the OS needs to map  PID to ASID and flush when appropriate.  ( Some PIDs share same ASID, they cannot run at  the same time)

Caches:
An address looks like:
| tag | index | offset| 

Question: 
- what address do we use 
	- to index the cache?
	- to tag the cache?

- Physically Indexed, Physically tagged
	- VPN goes to TLB, get PFN, and use PFN+Offset
	- Good: no flushing cache required, no aliasing
	- Bad: TLB on critical path, had to waiiiiiiiiiit for translation before reaching cache.
- Virtually Indexed, Virtually tagged
	- VNP and index goes directly to cache
	- Good: no TLB before cache, fast
	- Bad: 
		- sharing problems: need to distinguish between VAs from different processes.
		- aliasing problems:  different processes (address spaces) share a same phisical page, in memory it's at the same location, but because the way the mapped to the cache is  different, so they may end up different cache blocks, so they're not "shared" anymore.
	- How can virtual cache distinguish betweent processes
		- define it not to be a problem SASOS
		- brute force: flush cache on context swithching
		- use ASIDs (have to have HW support)
	- Solution:
		- disallow different names
		- (SUN OS solution) allow different names up to a point, must be the same modulo some large # ( factor of 2) 
		- Use cache size = page size
		- Use physically indexed cache (nah.. slow)
		- But if you combine the above 2, that's the x86 solution
			- use virtual indexed, if all your index bits comes from your VA and you have a set-associtive cache, aliased VAs will end up in same set, but still they're in different blocks...
			- Phisically tagged, because though shared data structures have different VAs, they have same physical address,  so the above problem is solved. 

TLB Reach problem

- usually there are 100~ entries of TLB, each maps a 4KB page, that's less than 1MB, but you have a more than 10MB LLC. So basically even if you things in cache, you cannot touch it before you go to page table...

Solution:

- large pages
- bigger TLB
- normally tagged, caches but use ASIDs 

#### Hypervisor
Traditionally apps runs on OS that runs directly on HW.
Nowadays OSes (guest OSes) runs on a Hypervisor, which runs on a HW.

How to implement this?

- guest OSes must be run through virtual memory
- protection between OSes, don't allow them to turn off/modify virtual memory.
- 3 privileges
	- full privilege: hypervisor
	- full privilege except turning off virtual memory. (Sys)
	- Apps (User)

### Mar 30

#### User API to the HW

you want this:

- user code can make requests
	- e.g. to devices
	- disk
	- network
	- keyboard...
- want OS protected from user

Options:

- Function call
	- when OS is a library
	- advantages
		- simple
		- fast
	- disadvantages: not secure, exposes OS code to user code relies on user code not to hog resources. (this is mostly used in embedded systems)
- full-on IPC - message passing
	- advantages: 
		- kind fast
		- totally secure
		- can be distributed
	- disadvantages
		- kinda slow
		- more complex
		- sending lots of data is impractical
- Trap (most commonly used)
	- SW causes an interrupt (essentially a contex switch)
	- data transfer via shared memory
	- advantages:
		- send lots of data cheaply because it's thru memory
		- fairly secure (use interrupt)
	- disadvantages
		- context swithcing slow
		- not as secure message passing

So how does the trap thing work?

- HW sends interrupt to OS, OS then sends signals to Apps, and your App could trap the interrupt thru something like an SWI

How do you communicate?

- what function to perform
	- malloc
	- create thread
	- write
	- read, oepn, close...
- what arguments? (how to pass it?)
	- if message pasing 
		- the message indicates the function (integer)
		- args: the message has some indicator where they are stored
	- If not, what else ?
		- what forms of command do we have 
		- we might have multiple trap instructions
		- register file
			- has some convention to use certain register to pass function and args
		- for big block of data-- thru shared memory
	- for "unix" there's this "known location", aka a user.struct, where you can put pointers to it. but that's in a user addr space
		- How does the OS access your struct
			- look in page table for mapping
			- put on VM by your ASID

So SW does this:

- put # into reg (func)
- put arg into regs and shared memory
- call trap instruction
- the handler look into register find the function
	- and grab args from regsters or memory
	- call that function with the args
	- return from exception
- back to your app with what you want is already there

Is it possible to do this, if so, how?

read(disk, buffer, size) 
read(network, buffer, size) 
read(flash, buffer, size) 
read(blah, buffer, size) 


### Apr, 6
Persistence

- Persistent memory 
	- kernel, utiilities
	- your apps
	- your files
	- in a word, stuff that affects the program that creates it

In the old time,  it means disk, tape, peachmeat...
Today, it's NVMEM ( non-volatile Mem )
It's interesting  beceause

- main memory: 10~100 cycles
- disk: 10 - 100 million cycle
- tape: eon
- non-volatile (e.g. flash) 100 - 100,000 cycles

It takes too long.. therefore when we access disks, we can afford to spend LOTS of time/attention/energy optimizing the access.

Discussion Questions
Q: What does it mean to be persistent
A: Data lasts beyond the exection of the program that creates it

- What are som implications of it?
	- how to handle ownership when a process exits but the data are still there?
	- how do you encode ownership? where is it stored? how is it checked?
	- If it is really long-lasting - how to read it? e.g old media, floppy disks/tapes, formats, odd OSes, etc
		- what if the data outlasts the system that creates it?
- What should it look like?
	- Files are arbitrary length: 0B, 1B..... xGB
- how do you name things?
	- today unix, windows, mac: all but unix expose the medium: you don't see disk number on the path, you can mount anything anywhere
	- In file systems, directories are *things* 
	- alternatively: "here is the name of my object -- it is long"
	- i.e. you could do this "447/projects/p1/' ...
- How to create it?
	- in file systems you open() or creat() a file and then write() into it at specific locations use sys calls 
	- if it's more like main memory,  would it still work this way?
		- int permanentify(uint page_num, char* name, ownership info)
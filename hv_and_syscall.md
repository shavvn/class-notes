# Hypervisor and System Call
## ( Mar, 28, cont'd)
### Hypervisor
Traditionally apps runs on OS that runs directly on HW.
Nowadays OSes (guest OSes) runs on a Hypervisor, which runs on a HW.

How to implement this?

- guest OSes must be run through virtual memory
- protection between OSes, don't allow them to turn off/modify virtual memory.
- 3 privileges
	- full privilege: hypervisor
	- full privilege except turning off virtual memory. (Sys)
	- Apps (User)

## (Mar 30)

### User API to the HW

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

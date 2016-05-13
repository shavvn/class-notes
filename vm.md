# Virtual Memory
## (Mar, 28)
### P6 Virtual Memory
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
|------|-----|-----|

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
		- use ASIDs (gotta have HW support)
	- Solution:
		- disallow different names
		- (SUN OS solution) allow different names up to a point, must be the same modulo some large # ( factor of 2) 
		- Use cache size = page size
		- Use physically indexed cache (nah.. slow)
		- But if you combine the above 2, that's the x86 solution
			- use virtual indexed, if all your index bits comes from your VA and you have a set-associtive cache, aliased VAs will end up in same set, but still they're in different blocks...
			- Phisically tagged, because though shared data structures have different VAs, they have same physical address,  so the above problem is solved. 
		- But then what if you want to make it larger?
			- make it wide (associativity)
			- when you use a bit from VA, you can double the # of entries of cache
			but now you have aliasing problem
				- The OS needs to make sure those VPN and PFN ends with same bit patterns
				for those bits which are used in translation, this is called page coloring. 

TLB Reach problem

- usually there are 100~ entries of TLB, each maps a 4KB page, that's less than 1MB, but you have a more than 10MB LLC. So basically even if you things in cache, you cannot touch it before you go to page table...

Solution:

- large pages
- bigger TLB
- normally tagged, caches but use ASIDs 





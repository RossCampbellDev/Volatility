#CLI #cheatsheet #malwareanalysis 
# Installation
go to the volatility website and get the linux executables, download, unzip
run using `./volatility`
(might have a much longer name)

## Usage
`./volatility -f dump.raw imageinfo`
[-f dump.raw] is the dump file we want to look at
[imageinfo] is a module that will give us information about what we're looking at.  tells us when image was created, number of processors, kernel, suggested profile which is very important as the profile helps us get more useful info.

may get multiple suggested profiles.  if errors/weirdness, try a different one.

convention:  name the dump file to match the profile for later

`./volatility  -f dump.raw --profile=Win7SP1x64 pslist`

[--profile=Win7SP1x64] this sets the profile to Win 7 service pack 1, 64bit
[pslist] will show us the list of processes that were in memory (address, PID, times etc)

[tasklist] for windows?

`./volatility  -f dump.raw --profile=Win7SP1x64 netscan | grep -vi closed`

[netscan] shows network connections that were active at the time - eg listening, closed, establish.  ports, addresses, processes that owned the connections
[grep -vi closed] will filter out any lines containing the word "closed" so we'll ignore them

`./volatility -f dump.raw --profile=Win7SP1x64 sockets`

detect any sockets listening on TCP/UDP/RAW.  

`./volatility  -f dump.raw --profile=Win7SP1x64 connections`
`./volatility  -f dump.raw --profile=Win7SP1x64 connscan`
list open connections at the time of the memory dump (connections)
or attempt to find traces of connections that had been closed (connscan)

`./volatility  -f dump.raw --profile=Win7SP1x64 psxview`

[psxview] will help us find processes that have been sneaky.  it will compare output of some (volatility) commands to see whether something has been listed or not.
if something is false in the pslist column but true in the others, this is suspicious - it indicates that a process is running in the system but it's detached from the process list (hidden?)

to see true and a bunch of falses might be a flag but less likely

`./volatility  -f dump.raw --profile=Win7SP1x64 pstree`

[pstree] shows us the process tree so we can see the parent processes that led to a process being run
perfect example is if a user reports maybe opening a dodgy email:  explorer.exe - outlook.exe - excel.exe - cmd.exe - virus.exe 

this tree basically confirms it

`./volatility  -f dump.raw --profile=Win7SP1x64 psscan --output=dot  --output-file=data.dot`

[psscan] gives us parent-child relationships
the file outputs will give us dot data that python can then present graphically in a tree

`./volatility  -f dump.raw --profile=Win7SP1x64 cmdline -p <PID`

[cmdline] gives us the command line output of what began the process given by "-p [PID]".  we can then see things like parameters that were involved

`./volatility  -f dump.raw --profile=Win7SP1x64 procdump`

[procdum] dump that process from memory to make a copy for further testing/reverse engineering eg using 
 strings
can submit to virustotal etc for checking

`./volatility  -f dump.raw --profile=Win7SP1x64 yarascan -Y "pattern"`

[yarascan] looks for a specific pattern within the memory dump.  in the example, an IP address was scanned so we could see what process was operating on a connection we found using netscan
we found a process, then can use a different plugin to look into what DLLs were involved:

`./volatility  -f dump.raw --profile=Win7SP1x64 dlllist -p <PID`

[dlllist] gives a list of what DLLs were loaded/used by a process
we can use [dlldump] to explore the DLL further and see what it's doing (eg static analysis)

`./volatility  -f stux.vmem hollowfind`

[hollowfind] is a plugin written by monnappa KA which attempts to find stuxnet-like malware that uses hollow process injection
this is where the process environment block thinks XYZ executable is being run from 0xFFFFFF memory address

however, the attacker has de-allocated that region of memory and then allocated their own malicious code in that space.  XYZ executable is still apparently running there, but it's something else now.

the plugin finds it by looking for "PAGE_EXECUTE_READWRITE" memory protection (RWX instead of WCX, PAGE_EXECUTE_WRITECOPY)

also, the process path will be blank in VAD but not in PEB
also looks for suspicious memory regions

# Suspicious processes

if we find a process we don't trust (eg an svchost.exe that was NOT started by services.exe) then we can use procdump to dump the executable into a file.
on linux we can then use "file [procump file]" to see what it is, eg PE (portable executable) for Windows

we can then do things like use 'strings' on it, or we can take a hash and upload to virus total etc

memdump just straight up dumps the memory of the process into a file for analysis

modscan will show drivers or kernel modules (linux) that are unloaded or hidden/unlinked by rootkits.  helpful to find stuff
can help you find processes that don't have a physical file mapped on disk - indicative of malware that exists only in memory

#CLI #cheatsheet

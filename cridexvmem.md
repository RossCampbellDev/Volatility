# Cridex infected dump of memory
## find out what profile to use
`./volatility -f ../dodgymem/cridex.vmem imageinfo`

## see what processes are running
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 pslist`
most looked normal.  didn't recognise reader_sl.exe or alg.exe

## try process tree
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 pstree`
mostly services started (expectedly) by services.exe
reader_sl.exe started by explorer.exe...?  I wonder if that's suspicious
parent PID for explorer.exe (1464) is not visible

## psxview
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 pxsview`
looking for anomalies.  hoping to see something for PID 1464 but it's not here
everything marked as 'true' in the pslist column.  a bunch of falses for smss, System, csrss but not likely anything

## networking
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 netscan` - not supported on this XP profile
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 connections`
only one connection - local port 1038 and remote address 8080
PID 1484... seems dubious?  started by explorer.exe

## CLI output
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 cmdline -p [pid]`
try on every PID that looks even vaguely suspicious or unfamiliar
all have a location on disk except winlogon.exe - not sure if normal or not

## procdump
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 procdump -p [pid] --dump-dir=~/Downloads/dodgymem/`
dump each of the processes i'm interested in
upload each to virustotal
reader_sl.exe - 29 red flags, trojan, riskware, Artemis.  "AcroSpeedLaunch.exe" (Wisdomeyes malware)
explorer.exe - 17 red flags, malicious, trojan

# following a blog post from this point

## what's in memory
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 memdump -p 1640 --dump-dir=~/Downloads/dodgymem/`
`strings 1640.dmp`
well that returned a whole mess.  narrow it down, try the IP that explorer.exe was connecting to
`strings 1640.dmp` | grep -Fi "41.168.5.140" -C 5`
this finds any lines that match the IP and gives us the 5 lines above and below it
we see a strange POST request

using `strings 1640.dmp | less` and then browsing, we see a ton of banking-related web domains
going further than the blog post, I also found javascript that is prompting users for social security numbers and company tax numbers - seems very dubious

## removal
to begin getting rid of this dodgy process, we need to go into the registry
`./volatility -f ../dodgymem/cridex.vmem --profile=WinXPSP2x86 hivelist`

# subnet-scanner
Bash script to automate initial host discovery and port scanning using a combination of fping, nping, nmap and rustscan (goscan as an option).  
fping, nmap, nping, rustscan and goscan (if used) must be installed as pre-requisites to use subnet-scanner.  

The .txt and .oG (nmap grepable output) files added to the repo are examples of results with subnet-scanner ran against a Metasploitable 2 VM (192.168.0.100) blocking ICMP and a VulnHub's Kioptrix VM (192.168.0.121) allowing ICMP. The goscan.txt example was ran against HTB Chatterbox.  

**Only use on networks that you own and have explicit authorization to scan!**  

## Disclaimer
Users take full responsibility for any actions performed using this tool.  The author accepts no liability for damage caused by this tool.  If these terms are not acceptable to you, then do not use this tool.  

## Usage  
```text
# sudo ./subnet-scanner A.B.C.D/E [-r] [-n] [-a] [-g] [-N] [-h]  
	[ -r | --rustscan ] perform rustscan scan  
	[ -n | --nmap ] perform nmap scan  
	[ -a | --all ] perform both rustscan AND nmap scans (this is the default if no switch specified)  
	[ -g | --goscan] perform an optional goscan scan
	[ -N | --nping ] option to perform nping scan for hosts/subnets that may block ICMP, a target port can be specified, 53 is used by default  
	[ -h | --help ]  
```

Notes:  
* Run with sudo/root privileges so that nmap can perform OS detection by using the -A switch  
* For A.B.C.D/E, use [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation) notation, ex: 10.0.1.0/20, 192.168.5.0/27, etc.  
* To scan a single host, use the /32 prefix length, ex: 10.0.1.111/32, 192.168.5.35/32, etc.  

## fping
subnet-scanner will always start by running [fping](https://fping.org/) to discover responsive hosts on the specified subnet.  
Hosts discovered by fping will be listed in ./fping.txt.  
If only fping is run (no nping), the content of ./fping.txt is copied to ./live_hosts.txt.  

## nping
In cases where ICMP is blocked, subnet-scanner has the optional -N (--nping) command switch to run an additional TCP probe [nping](https://nmap.org/nping/) scan, using target port 53 by default (none specified) or the specified one.  
Hosts discovered by nping will be listed in ./nping.txt.
When both fping AND nping (-N | --nping) scans are run, the contents of ./fping.txt and ./nping.txt are merged into ./live_hosts.txt with duplicate host entries removed.  

## goscan  
The [goscan](https://github.com/sdcampbell/goscan) scan is optional (not run and not included by default). It will be run first when the -g (--goscan) switch is provided, loop over all live hosts in ./live_hosts.txt and scan each for all TCP ports.  

## rustscan
subnet-scanner supports running a [rustscan](https://rustscan.github.io/RustScan/) scan on the discovered ./live_hosts.txt file, scanning for all 65535 ports, storing the detailed results into ./rustscan.txt and the summary results into ./rustscan_summary.txt (and ./rustscan.oG).  
If only a rustscan scan is desired (no nmap), then use the -r (--rustscan) command switch.  

## nmap
subnet-scanner supports running an [nmap](https://nmap.org/) scan on the discovered ./live_hosts.txt file, scanning for all 65535 ports, storing the detailed results into ./nmap.txt and the summary results into ./nmap_summary.txt (and nmap.oG).  
If only an nmap scan is desired (no rustscan), then use the -n (--nmap) command switch. 

## Examples
### Example 1
```text
# sudo ./subnet-scanner 192.168.0.0/25  
```
Discover and scan the 192.168.0.0/25 subnet using all default settings: fping, rustscan and nmap scans.  
Equivalent to commands:  
```text
# sudo ./subnet-scanner 192.168.0.0/25 -a  
# sudo ./subnet-scanner 192.168.0.0/25 -n -r  
```
### Example 2
```text
# sudo ./subnet-scanner 192.168.0.0/25 -N  
```	
Same as previous example, but adds an nping scan on default target port 53 before rustscan. This can help discover hosts that may block ICMP and that would not otherwise be caught by fping. By comparing the ./fping.txt and the ./nping.txt files, we can see which hosts may be blocking ICMP.  

### Example 3
```text
# sudo ./subnet-scanner 192.168.0.0/25 -N 443  
```
Same as previous example, but specifies the target port to use for nping (443 instead of default 53).  

### Example 4
```text
# sudo ./subnet-scanner 192.168.0.0/25 -n  
```	
After fping, only run an nmap scan (-n), no rustscan.  

### Example 5
```text
# sudo ./subnet-scanner 192.168.0.0/25 -r -N  
```	
After fping, run nping on default target port 53 (-N) and only run a rustscan scan (-r), no nmap.

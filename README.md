# subnet-scanner
Bash script to automate initial host discovery and port scanning using fping, nping, nmap and rustscan.  
fping, nmap, nping and rustscan must be installed as pre-requisites to use subnet-scanner.

## Usage  
&ensp;# ./subnet-scanner A.B.C.D/E [-r] [-n] [-a] [-N] [-h]  
&ensp;&ensp;[ -r | --rustscan ] perform rustscan scan  
&ensp;&ensp;[ -n | --nmap ] perform nmap scan  
&ensp;&ensp;[ -a | --all ] perform both rustscan AND nmap scans (default if no switch specified)  
&ensp;&ensp;[ -N | --nping ] option to perform nping scan for hosts/subnets that may block ICMP, a target port can be specified, 53 is used by default  
&ensp;&ensp;[ -h | --help ]  

Note: for A.B.C.D/E, use [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation) notation, ex: 10.0.1.0/20, 192.168.5.0/27, etc.  
To scan a single host, use the /32 prefix length, ex: 10.0.1.111/32, 192.168.5.35/32, etc.  

## fping
subnet-scanner will always start by running [fping](https://fping.org/) to discover responsive hosts on the specified subnet.  
Hosts discovered by fping will be listed in ./fping.txt.  
If only fping is run (no nping), the content of ./fping.txt is copied to ./live_hosts.txt.  

## nping
In cases where ICMP is blocked, subnet-scanner has the optional -N (--nping) command switch to run an additional TCP probe [nping](https://nmap.org/nping/) scan, using target port 53 by default (none specified) or the specified one.  
Hosts discovered by nping will be listed in ./nping.txt.
When both fping AND nping (-N | --nping) scans are run, the contents of ./fping.txt and ./nping.txt are merged into ./live_hosts.txt with duplicate host entries removed.  
## rustscan
subnet-scanner supports running a [rustscan](https://rustscan.github.io/RustScan/) scan on the discovered ./live_hosts.txt file, scanning for all 65535 ports, storing the detailed results into ./rustscan.txt and the summary results into ./rustscan_summary.txt.  
If only a rustscan scan is desired (no nmap), then use the -r (--rustscan) command switch.  

## nmap
subnet-scanner supports running an [nmap](https://nmap.org/) scan on the discovered ./live_hosts.txt file, scanning for all 65535 ports, storing the detailed results into ./nmap.txt and the summary results into ./nmap_summary.txt.  
If only an nmap scan is desired (no rustscan), then use the -n (--nmap) command switch. 

## Examples
&ensp;# ./subnet-scanner 192.168.0.0/25  
&ensp;Discover and scan the 192.168.0.0/25 subnet using all default settings: fping, rustscan and nmap scans.  
&ensp;Equivalent to commands:  
&ensp;&ensp;# ./subnet-scanner 192.168.0.0/25 -a
&ensp;&ensp;# ./subnet-scanner 192.168.0.0/25 -n -r

&ensp;# ./subnet-scanner 192.168.0.0/25 -N  
&ensp;Same as previous example, but adds an nping scan on default target port 53 before rustscan. This can help discover hosts that may block ICMP and that would not otherwise be caught by fping. By comparing the ./fping.txt and the ./nping.txt files, we can see which hosts may be blocking ICMP.  

&ensp;# ./subnet-scanner 192.168.0.0/25 -N 443  
&ensp;Same as previous example, but specifies the target port for nping to use 443 (instead of default 53).  

&ensp;# ./subnet-scanner 192.168.0.0/25 -n
&ensp;After fping, only run an nmap scan (no rustscan)  

&ensp;# ./subnet-scanner 192.168.0.0/25 -r -N
&ensp;After fping, run nping on default target port 53 and only run a rustscan scan (no nmap).

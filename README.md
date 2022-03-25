# subnet-scanner
Bash script to automate initial host discovery and port scanning using fping, nping, nmap and rustscan.  
fping, nmap, nping and rustscan must be installed as pre-requisites to use subnet-scanner.

## Usage  
&ensp;Usage:./subnet-scanner A.B.C.D/E [-r] [-n] [-a] [-N] [-h]  
&ensp;&ensp;[ -r | --rustscan ]&ensp;perform rustscan scan  
&ensp;&ensp;[ -n | --nmap ]&ensp;&ensp;perform nmap scan  
&ensp;&ensp;[ -a | --all ]&ensp;&ensp;perform both rustscan AND nmap scans (default if no switch specified)  
&ensp;&ensp;[ -N | --nping ]&ensp;option to perform nping scan for hosts/subnets that may block ICMP,  
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;a target port can be specified, 53 is used by default  
&ensp;&ensp;[ -h | --help ]  

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

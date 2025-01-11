# Born2beRoot 🌍

This repository contains the **Born2beRoot** project, including a monitoring script that provides detailed system statistics. The script outputs essential system information, such as architecture, CPU, memory usage, disk space, and more. 📊

---

## Script Features 🛠️

The monitoring script provides the following system details:

- **Architecture** 🖥️: System architecture (via `uname`).
- **CPU Information** 🧠:
  - **Physical CPUs** ⚙️: Total number of physical sockets.
  - **Virtual CPUs** 🖧: Total number of virtual processors.
- **Memory Usage** 💾:
  - Total memory and used memory in MB.
  - Percentage of memory utilization.
- **Disk Usage** 💽:
  - Total and used disk space (in GB/MB).
  - Percentage of disk utilization.
- **CPU Load** 🔋: Current percentage of CPU usage.
- **Last Boot Time** 🔄: The date and time of the last system reboot.
- **LVM Use** 📦: Indicates if Logical Volume Manager (LVM) is enabled.
- **TCP Connections** 🌐: Number of active established TCP connections.
- **User Logs** 👥: Number of users currently logged into the system.
- **Network Details** 🌍:
  - IPv4 address of the system.
  - MAC address of the network interface.
- **Sudo Command History** ⏳: Total number of `sudo` commands executed.

---

## Usage Instructions ⚙️

### Clone the Repository 🖥️

```bash
git clone <repository-url>
cd <repository-directory>
```
---

### Make the Script Executable 🔒

```bash
chmod +x monitoring.sh
```
---

### Run the Script ▶️

Execute the script to display system statistics:

```bash
./monitoring.sh
```
---

### Broadcast System Information 📢

Use the following command to broadcast the system statistics to all users:

```bash
wall "$(./monitoring.sh)"
```
---

## Example Output 📄

When executed, the script provides output similar to the following:

```
#Architecture: Linux c1r3p10 5.15.0-73-generic #80-Ubuntu SMP Fri May 26 17:50:02 UTC 2023 x86_64
#CPU physical: 4
#vCPU: 8
#Memory Usage: 1024/4096MB (25.00%)
#Disk Usage: 10240/20480Gb (50%)
#CPU load: 12.5%
#Last boot: 2025-01-09 14:45
#LVM use: yes
#Connections TCP: 10 ESTABLISHED
#User log: 2
#Network: IP 192.168.0.100 (00:1A:2B:3C:4D:5E)
#Sudo: 15 cmd
```

---

## Notes ⚠️

- Ensure you have root permissions when running the script, especially to access `journalctl` and `ss`.
- Modify the script to fit your specific environment if needed.
- The output may vary depending on your system configuration and installed utilities.

---

## Script Code 💻

Below is the complete code for the monitoring script:

```bash
#!/bin/bash

#------------Architecture-----------
archit=$(uname -a | sed "s/ PREEMPT_DYNAMIC//")

#------------CPU physical-----------
pcpu=$(lscpu | grep "^Socket(s):" | awk '{print $2}')

#----------------vCPU---------------
vcpu=$(lscpu | grep "^CPU(s):" | awk '{print $2}')

#------------Memory Usage-----------
arm=$(free -m | awk '$1 == "Mem:" {print $2}')
urm=$(free -m | awk '$1 == "Mem:" {print $3}')
prm=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')

#-------------Disk Usage------------
adm=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{am += $2} END {print am}')
udm=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{um += $3} END {print um}')
pdm=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{um += $3} {am += $2} END {printf("%d"), um/am*100}')

#--------------CPU load-------------
ucpu=$(grep 'cpu ' /proc/stat | awk '{usage=($2+$4)*100/($2+$4+$5)} END {print usage}' | awk '{printf "%.1f", $1}')

#--------------Last boot------------
lb=$(uptime -s | cut -d: -f1,2)

#-----------------LVM---------------
lvm=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo -n "yes"; else echo -n "no"; fi)

#--------------TCP Connections------
tcpc=$(ss -ta | grep -c 'ESTAB')

#---------------User log------------
ul=$(who | awk '{print $1}'| sort -u | wc -l)

#----------------Network------------
ip4=$(hostname -I)
maca=$(ip link | awk '$1 == "link/ether" {print $2}')

#-------------Sudo commands---------
scmd=$(journalctl _COMM=sudo -q | grep 'COMMAND' | wc -l)


wall " #Architecture: $archit
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $urm/${arm}MB ($prm%)
	#Disk Usage: $udm/${adm}Gb ($pdm%)
	#CPU load: $ucpu%
	#Last boot: $lb
	#LVM use: $lvm
	#Connections TCP: $tcpc ESTABLISHED
	#User log: $ul
	#Network: IP $ip4 ($maca)
	#Sudo: $scmd cmd"


---


### 👨‍💻 Author
Zoubair Kharbach


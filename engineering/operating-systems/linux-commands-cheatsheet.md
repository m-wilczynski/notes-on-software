[Operating systems](/engineering/operating-systems)
# Linux commands cheatsheet

### Filesystem
|Command|Usage|
--- | ---
`cd <dirname>`|navigate to given directory
`ls` | list directory contents; `ls -l -a` to list all (-a) with more details (-l); `ls -l` can be shortened to `ll`
`touch <filename>` | create file
`mkdir <dirname>` | create directory
`cp <source> <destination>` | copies source file to given destination
`mv <source> <destination>` | moves source file to given destination; commonly used to rename file if destination is the same as source but with changed filename
`chmod <permissions> <file-or-directory>` | give permissions to file or directory; for example `chmod 600 document.pdf` will give **-rw-** permissions only to owner (6) and no permissions to the rest (0 and 0)

### Find, browse and modify files
|Command|Usage|
--- | ---
`cat <filename>` | print file contents
`grep <phrase> <space-delimited-filenames>` | search for a phrase in given filenames; for example `grep '[ERROR]' today.log yesterday.log` will look for *[ERROR]* in files today.log and yesterday.log
`grep -r <phrase> <directory>` | search recursively for a phrase in given directory; for example `grep -r '###' *` will look for h3 markdown tag *###* in all of the directories including this one and all of the subdirectories (recursively)
`find <directory> -name <filename-pattern>` | look for a files of given pattern in given directory; for example `find . -name '*.md'` will look for all the markdown files in current directory, recursively
`find <directory> -type <type>` | look for a files of given type; (common, but not all) types are `f` - file, `d` - directory, `l` - symbolic link, `p` - named pipe and `s` - socket
`sed -i 's/SEARCH_REGEX/NEW_VALUE/g <filename>` | will look for a *SEARCH_REGEX* in a given file and replace all of its occurences with *NEW_VALUE*; if `g` is omitted, only first ocurrence will be replaced
`awk` | this needs a separate article; **awk** is general purpose scripting language for text processing
`<command> \| less` | allows paged browsing  of output of given command


### System
|Command|Usage|
--- | ---
`watch` | watch given command refreshing every 2s, for ex. `watch 'vmstat -s'` to continously watch memory stats
`ps <options>`, commonly used as `ps aux` | prints all (`a`) processes with user-oriented format (`u`) excluding executing terminal (`x`)
`pstree` | prints processes as a tree
`htop` | interactive monitor showing current system load; package requires installation!
`top` | if you cannot install anything on target system, you can use this simpler version of `htop`
`vmstat` | monitor either memory with `-s` or disk IO with `-d` argument
`pidstat` | resources usage by process
`iostat` | IO statistics usage per device
`cat /proc/meminfo` | display memory info
`cat /proc/cpuinfo` | display CPU info
`df -h` | display free space per mounted device
`iptables` | another one requiring separate note; iptables deals with network packets filtering and is most commonly used as a firewall; `iptables --list` prints all of the rules in use
`ss -ltpan` | lists open/listening ports in system (`t` stands for tcp, use `u` to include udp)
`lsof -c <process_name>` | lists files used by process named `process_name`
`lsof <file_path>` | lists processes using file with given `file_path`
`journalctl` | display journal log of `systemd` (think like `eventvwr` on Windows)
`perf` | used to analyze performance counters and other trace events emmited by applications; **this needs seperate article!**; not installed by default - use ```apt-get install linux-tools-common linux-tools-generic linux-tools-`uname -r` ``` to get it installed
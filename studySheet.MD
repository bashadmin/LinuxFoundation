### Links:
* https://opencontainers.org/

# Learning Objectives
_______
By the end of this chapter, you should be able to:

Explain why Linux requires the organization of one big filesystem tree, and what the major considerations are for how it is done.
Explain the role played by the Filesystem Hierarchy Standard.
Describe what must be available at boot in the root (/) directory, and what can be available only once the system has started.
Explore each of the main subdirectory trees, explain their purposes, and examine their contents.


## One Big Filesystem
_______
Linux, like all UNIX-based operating systems, consists of one big filesystem tree. Actually, it is usually diagrammed as an inverted tree with the root directory, /, being at the top of the tree.

Within this one large logical filesystem there may be more than one, even many, distinct filesystems, mounted at various points, which appear as subdirectories. These distinct filesystems are usually on different partitions, which can be on any number of devices, including those which are on a network.

Regardless of exactly how things are joined together, it all just looks like one big filesystem; applications do not usually care at all about what physical device files actually reside on.

Once upon a time, different UNIX-like operating systems organized this one big tree in varying ways; even among the various Linux distributions there were many differences. This made both writing applications and accomplishing system administration tasks on more than one kind of system difficult and often frustrating.

As a consequence, the Linux ecosystem has worked hard to establish standardized procedures to minimize such pain.

## Data Distinctions

When talking about how files and data are organized in the one big directory tree, it is important to learn some taxonomy on what kind of information has to be read and written. In particular, there are two kinds of distinctions.

Close Shareable vs. non-shareable
Shareable data is that which can be shared between different hosts. Non-shareable data is that which must be specific to a particular host. For example, user home directories may be shareable, while device lock files are not.

Close Variable vs. static
Static data include binaries, libraries, documentation, and anything that does not change without system administrator assistance. Variable data is anything that may change even without a system administrator's help.

A small root filesystem is less prone to corruption as the result of a system crash.

Rationale
______
There are several reasons why creating a new subdirectory of the root filesystem is prohibited:
* It demands space on a root partition which the system administrator may want kept small and
simple for either performance or security reasons.
The Root Filesystem

* It evades whatever discipline the system administrator may have set up for distributing standard
file hierarchies across mountable volumes.
Distributions should not create new directories in the root hierarchy without extremely careful
consideration of the consequences including for application portability.

## 3.4. /bin : Essential user command binaries (for use by all users)
## 3.4.1. Purpose
/bin contains commands that may be used by both the system administrator and by users, but which are
required when no other filesystems are mounted (e.g. in single user mode). It may also contain commands
which are used indirectly by scripts. 


 Command binaries that are not essential enough to place into /bin must be placed in /usr/bin, instead. Items that are required only by nonroot users (the X Window System, chsh, etc.) are generally not essential enough to be placed into the root partition.

 

## 3.5. /boot : Static files of the boot loader
This directory contains everything required for the boot process except configuration files not needed at
boot time and the map installer. Thus /boot stores data that is used before the kernel begins executing
user-mode programs. This may include saved master boot sectors and sector map files.
Programs necessary to arrange for the boot loader to be able to boot a file must be placed in /sbin.
Configuration files for boot loaders that are not required at boot time must be placed in /etc.

If it is possible that devices in /dev will need to be manually created, /dev must contain a command
named MAKEDEV, which can create devices as needed. It may also contain a MAKEDEV.local for any
local devices.
If required, MAKEDEV must have provisions for creating any device that may be found on the system, not
just those that a particular distribution installs.

No binaries may be located under /etc.

 To be clear, /etc may contain executable scripts, such as the command scripts commonly called by init to start and shut down the system
and start daemon processes. "Executable binary" in this context refers to direct machine code or pseudocode not in a human-readable format, such
as native ELF executables.

 Systems that use the shadow password suite will have additional configuration files in /etc (/etc/shadow and others) and programs in /
usr/sbin (useradd, usermod, and others).


/etc/fstab is a list of filesystems to be mounted at boot time. If you want your Windows or file-storage partitions mounted once your computer boots, you'll need to put appropriate entries into /etc/fstab.

/etc/mtab is a list of currently mounted filesystems. If you have a disk connected but not mounted, it won't show up in the /etc/mtab file. Once you mount it, it will show up there.

Host-specific configuration files for add-on application software packages must be installed within the
directory /etc/opt/<subdir>, where <subdir> is the name of the subtree in /opt where the static
data from that package is stored.


To find a user's home directory, use a library function such as getpwent, getpwent_r of fgetpwent rather than relying on /etc/passwd
because user information may be stored remotely using systems such as NIS.
 It is recommended that, apart from autosave and lock files, programs should refrain from creating non dot files or directories in a home directory
without user consent.

The /lib directory contains those shared library images needed to boot the system and run the commands
in the root filesystem, ie. by binaries in /bin and /sbin.

Shared libraries that are only necessary for binaries in /usr (such as any X Window binaries) must not be in /lib. Only the shared libraries
required to run binaries in /bin and /sbin may be here. In particular, the library libm.so.* may also be placed in /usr/lib if it is not
required by anything in /bin or /sbin.
The usual placement of this binary is /usr/bin/cpp.
This is commonly used for 64-bit or 32-bit support on systems which support multiple binary formats, but require libraries of the same name. In
this case, /lib32 and /lib64 might be the library directories, and /lib a symlink to one of them.

The internal format of PID files remains unchanged. The file must consist of the process identifier in
ASCII-encoded decimal, followed by a newline character. For example, if crond was process number 25,
/run/crond.pid would contain three characters: two, five, and newline.
Programs that read PID files should be somewhat flexible in what they accept; i.e., they should ignore extra
whitespace, leading zeroes, absence of the trailing newline, or additional lines in the PID file. Programs
that create PID files should use the simple specification located in the above paragraph.

We recommend that users have read and execute permission for everything in /sbin except, perhaps, certain setuid and setgid programs. The
division between /bin and /sbin was not created for security reasons or to prevent users from seeing the operating system, but to provide a good
partition between binaries that everyone uses and ones that are primarily used for administration tasks. There is no inherent security advantage in
making /sbin off-limits for users.

## 3.17. /srv : Data for services provided by this
system
3.17.1. Purpose
/srv contains site-specific data which is served by this system.
Rationale
This main purpose of specifying this is so that users may find the location of the data files for
a particular service, and so that services which require a single tree for readonly data, writable
data and scripts (such as cgi scripts) can be reasonably placed. Data that is only of interest to a
specific user should go in that users' home directory. If the directory and file structure of the data
is not exposed to consumers, it should go in /var/lib.
The methodology used to name subdirectories of /srv is unspecified as there is currently no
consensus on how this should be done. One method for structuring data under /srv is by
protocol, eg. ftp, rsync, www, and cvs. On large systems it can be useful to structure /srv
by administrative context, such as /srv/physics/www, /srv/compsci/cvs, etc. This
setup will differ from host to host. Therefore, no program should rely on a specific subdirectory
structure of /srv existing or data necessarily being stored in /srv. However /srv should
always exist on FHS compliant systems and should be used as the default location for such data.
Distributions must take care not to remove locally placed files in these directories without
administrator permission. 


 Miscellaneous architecture-independent application-specific static files and subdirectories must be placed in /usr/share.

See below, in the /usr/libexec section, for a discussion of /usr/lib vs. /usr/libexec for executable binaries.

 For example, the perl5 subdirectory for Perl 5 modules and libraries.

 Some executable commands such as makewhatis and sendmail have also been traditionally placed in /usr/lib. makewhatis is an internal
binary and must be placed in a binary directory; users access only catman. Newer sendmail binaries are now placed by default in /usr/sbin.
Additionally, systems using a sendmail-compatible mail transfer agent must provide /usr/sbin/sendmail as the sendmail command, either as the
executable itself or as a symlink to the appropriate executable.

 Host-specific data for the X Window System must not be stored in /usr/lib/X11. Host-specific configuration files such as xorg.conf
must be stored in /etc/X11. This includes configuration data such as system.twmrc even if it is only made a symbolic link to a more global
configuration file (probably in /usr/lib/X11).


## 4.9. /usr/local : Local hierarchy
4.9.1. Purpose
The /usr/local hierarchy is for use by the system administrator when installing software locally. It
needs to be safe from being overwritten when the system software is updated. It may be used for programs
and data that are shareable amongst a group of hosts, but not found in /usr.
Locally installed software must be placed within /usr/local rather than /usr unless it is being
installed to replace or upgrade software in /usr. 

## 4.10. /usr/sbin : Non-essential standard system
binaries
4.10.1. Purpose
This directory contains any non-essential binaries used exclusively by the system administrator. System
administration programs that are required for system repair, system recovery, mounting /usr, or other essential functions must be placed in /sbin instead


 Locally installed system administration programs should be placed in /usr/local/sbin.
10 Much of this data originally lived in /usr (man, doc) or /usr/lib (dict, terminfo, zoneinfo)/usr/local/share

______
This naming of language subdirectories of /usr/share/man is based on Appendix E of the POSIX
1003.1 standard which describes the locale identification string — the most well-accepted method to
describe a cultural environment. The <locale> string is:
<language>[_<territory>][.<character-set>][,<version>]
The <language> field must be taken from ISO 639 (a code for the representation of names of languages).
It must be two characters wide and specified with lowercase letters only.
The <territory> field must be the two-letter code of ISO 3166 (a specification of representations of
countries), if possible. (Most people are familiar with the two-letter codes used for the country codes in
email addresses.) It must be two characters wide and specified with uppercase letters only. 13
The <character-set> field must represent the standard describing the character set. If the -
<character-set> field is just a numeric specification, the number represents the number of the
international standard describing the character set. It is recommended that this be a numeric representation
if possible (ISO standards, especially), not include additional punctuation symbols, and that any letters
be in lowercase.
A parameter specifying a <version> of the profile may be placed after the <character-set> field,
delimited by a comma. This may be used to discriminate between different cultural needs; for instance,
dictionary order versus a more systems-oriented collating order. This standard recommends not using the
<version> field, unless it is necessary.
______

In Unix a file is just a sequence of bytes, however some files have more structure. The file command can look at the sequence of bytes and tell you things.for example if the first 4bytes are \177 E L F then it will tell you it is an elf file. It will then look at more bytes and tell you if it is a core file, an executable or an object file. If the bytes 6 thru 9 are JFIF then it is a picture file and again the program can read more and tell you about it.

A lot of this is driven by data tables rather than being compiled into the program. These data tables are what is in the magic files.

So the reason why you have magic files is to allow you to say file * and get the program to tell you what you have in the current directory.

## Chapter 5. The /var Hierarchy
5.1. Purpose
/var contains variable data files. This includes spool directories and files, administrative and logging
data, and transient and temporary files.
Some portions of /var are not shareable between different systems. For instance, /var/log, /var/
lock, and /var/run. Other portions may be shared, notably /var/mail, /var/cache/man, /
var/cache/fonts, and /var/spool/news.
/var is specified here in order to make it possible to mount /usr read-only. Everything that once went
into /usr that is written to during system operation (as opposed to installation and software maintenance)
must be in /var.
If /var cannot be made a separate partition, it is often preferable to move /var out of the root partition
and into the /usr partition. (This is sometimes done to reduce the size of the root partition or when space
runs low in the root partition.) However, /var must not be linked to /usr because this makes separation
of /usr and /var more difficult and is likely to create a naming conflict. Instead, link /var to /usr/
var.
Applications must generally not add directories to the top level of /var. Such directories should only be
added if they have some system-wide implication, and in consultation with the FHS mailing list.

## 5.5. /var/cache : Application cache data
5.5.1. Purpose
/var/cache is intended for cached data from applications. Such data is locally generated as a result of
time-consuming I/O or calculation. The application must be able to regenerate or restore the data. Unlike
/var/spool, the cached files can be deleted without data loss. The data must remain valid between
invocations of the application and rebooting the system.
Files located under /var/cache may be expired in an application specific manner, by the system
administrator, or both. The application must always be able to recover from manual deletion of these files
(generally because of a disk space shortage). No other requirements are made on the data format of the
cache directories.
Rationale
The existence of a separate directory for cached data allows system administrators to set different
disk and backup policies from other directories in /var.

Rationale
Release 1.2 of this standard specified /var/catman for this hierarchy. The path has been
moved under /var/cache to better reflect the dynamic nature of the formatted man pages. The
directory name has been changed to man to allow for enhancing the hierarchy to include postprocessed formats other than "cat", such as PostScript, HTML, or DVI.

## 5.8. /var/lib : Variable state information
5.8.1. Purpose
This hierarchy holds state information pertaining to an application or the system. State information is data
that programs modify while they run, and that pertains to one specific host. Users must never need to
modify files in /var/lib to configure a package's operation, and the specific file hierarchy used to store
the data must not be exposed to regular users. 3
State information is generally used to preserve the condition of an application (or a group of interrelated applications) between invocations and between different instances of the same application. State
information should generally remain valid after a reboot, should not be logging output, and should not
be spooled data.
An application (or a group of inter-related applications) must use a subdirectory of /var/lib for its data.
There is one required subdirectory, /var/lib/misc, which is intended for state files that don't need
3
 Data with exposed filesystem structure should be stored in /srv.
The /var Hierarchy
34
a subdirectory; the other subdirectories should only be present if the application in question is included
in the distribution. 4
/var/lib/<name> is the location that must be used for all distribution packaging support. Different
distributions may use different names, of course.

## 5.9. /var/lock : Lock files
5.9.1. Purpose
Lock files should be stored within the /var/lock directory structure.
Lock files for devices and other resources shared by multiple applications, such as the serial device lock
files that were originally found in either /usr/spool/locks or /usr/spool/uucp, must now be
stored in /var/lock. The naming convention which must be used is "LCK.." followed by the base name
of the device. For example, to lock /dev/ttyS0 the file "LCK..ttyS0" would be created. 6
The format used for the contents of such lock files must be the HDB UUCP lock file format. The HDB
format is to store the process identifier (PID) as a ten byte ASCII decimal number, with a trailing newline.
For example, if process 1230 holds a lock file, it would contain the eleven characters: space, space, space,
space, space, space, one, two, three, zero, and newline.

## 5.10. /var/log : Log files and directories
5.10.1. Purpose
This directory contains miscellaneous log files. Most logs must be written to this directory or an appropriate
subdirectory.

UUCP is an acronym of Unix-to-Unix Copy. The term generally refers to a suite of computer programs and protocols allowing remote execution of commands and transfer of files, email and netnews between computers.

## 5.15. /var/tmp : Temporary files preserved
between system reboots
5.15.1. Purpose
The /var/tmp directory is made available for programs that require temporary files or directories that are preserved between system reboots. Therefore, data stored in /var/tmp is more persistent than data in /tmp.
Files and directories located in /var/tmp must not be deleted when the system is booted. Although data stored in /var/tmp is typically deleted in a site-specific manner, it is recommended that deletions occur at a less frequent interval than /tmp.

## 6.1.5. /proc : Kernel and process information virtual
filesystem
The proc filesystem is the de-facto standard Linux method for handling process and system information, rather than /dev/kmem and other similar methods. We strongly encourage this for the storage and retrieval of process information as well as other kernel and memory information.

## 6.1.7. /sys : Kernel and system information virtual
filesystem
The sys filesystem is the location where information about devices, drivers, and some kernel features is exposed. Its underlying structure is determined by the particular Linux kernel being used at the moment, and is otherwise unspecified.

## The Root (/) Directory
______
While the entire filesystem can be viewed as one big tree, as we have pointed out, there may be multiple partitions and filesystems joined together.

The partition and filesystem that the root directory itself is contained in is rather special and is often in a special dedicated partition, with other components with directories such as /home, /var and /opt, etc., to be mounted later.

The root partition must contain all essential files required to boot the system and then mount all other filesystems. Thus, it needs utilities, configuration files, boot loader information, and other essential startup data. It must be adequate to:

Boot the system.​
Restore the system from system backups on external media such as tapes and other removable media or NAS etc.​
Recover and/or repair the system; an experienced maintainer must have the tools to diagnose and reconstruct a damaged system.
According to the FHS, no application or package should create new subdirectories of the root directory.

## /bin
______
The /bin directory is very important because:

It contains executable programs and scripts needed by both system administrators and unprivileged users, which are required when no other filesystems have yet been mounted, for example when booting into single user or recovery mode.
It may also contain executables which are used indirectly by scripts.
It may not include any subdirectories.
Required programs which must exist in /bin/ include:

cat, chgrp, chmod, chown, cp, date, dd, df, dmesg, echo, false, hostname, kill, ln, login, ls, mkdir, mknod, more, mount, mv, ps, pwd, rm, rmdir, sed, sh, stty, su, sync, true, umount and uname

[ and test may be there as well, and optionally, it may include: csh, ed, tar, cpio, gunzip, zcat, netstat and ping.

Command binaries that are deemed nonessential enough to merit a place in /bin go in /usr/bin. Programs required only by non-root users are placed in this category.

These files have longer names which depend on the kernel version, with the exact form depending on the Linux distribution. In addition, instead of initramfs, the file may be called initrd, which stands for initial RAM disk, which was the older method, but the name survives.

/boot stores data used before the kernel begins executing user-mode programs. It also includes two files used for information and debugging:

config
Used to configure the kernel compilation.
System.map
Kernel symbol table, used for debugging.
It may include saved master boot sectors, sector map files, and other data that is not directly edited by hand. The exact contents of /boot will vary by distribution and time; below, you can see an example of what we have on one Fedora system.


/sys
This directory is the mount point for the sysfs pseudo-filesystem where all information resides only in memory, not on disk. Like /dev and /proc, the /sys directory is empty on a non-running system. It contains information about devices and drivers, kernel modules, system configuration structures, etc.

sysfs is used both to gather information about the system, and modify its behavior while running. In that sense, it resembles /proc, but it is younger than and has adhered to strict standards about what kind of entries it can contain. For example, almost all pseudo-files in /sys contain only one line, or value; there are none of the long entries you can find in /proc.

As with /proc, we will examine entries in /sys throughout this course, and it will become relevant in upcoming chapters about kernel configuration and system monitoring.

/var
This directory contains variable (or volatile) data files that change frequently during system operation. These include:

Log files
Spool directories and files
Administrative data files
Transient and temporary files, such as cache contents.
Obviously, /var cannot be mounted as a read-only filesystem.

For security reasons, it is often considered a good idea to mount /var as a separate filesystem. Furthermore, if the directory gets filled up, it should not lock up the system.

/var/log is where most of the log files are located, and /var/spool is where local files for processes such as mail, printing, and cron jobs are stored while awaiting action.

/run
A new directory tree mounted at /run has been in use for several years by major Linux distributions, and, while proposed for FHS, it has not been formally accepted. However, it is good to know about it, as you are quite likely to encounter it.

The purpose of /run is to store transient files: those that contain runtime information, which may need to be written early in system startup, and which do not need to be preserved when rebooting.

Generally, /run is implemented as an empty mount point, with a tmpfs ram disk (like /dev/shm) mounted there at runtime. Thus, this is a pseudo-filesystem existing only in memory.

Some existing locations, such as /var/run and /var/lock, will be now just symbolic links to directories under /run. Other locations, depending on distribution taste, may also just point to locations under /run.

Learning Objectives
By the end of this chapter, you should be able to:

Describe a process and the resources associated with it.
Describe the role of the init process.
Distinguish between processes, programs and threads.
Understand process attributes, permissions and states, and know how to control limits.
Explain the difference between running in user and kernel modes.
Describe daemon processes.
Understand how new processes are forked (created).
Use nice and renice to set and modify priorities.
Understand how shared and static libraries are used.


Programs, Processes, and Threads
A program is a set of instructions, along with any internal data used while carrying the instructions out. Programs may also use external data. Internal data might include text strings inside the program, which are used to display user prompts. External data might include data from a database. Many user commands, such as ls, cat and rm are programs which are external to the operating system kernel, or shell (in other words, they have their own executable program on disk).

A process is an executing program and associated resources, including environment, open files, signal handlers, etc. The same program may be executing more than once simultaneously, and thus, be responsible for multiple processes.

At the same time, two or more tasks, or threads of execution, can share various resources, such as their entire memory spaces (or just particular memory areas), open files, etc. When there is an everything shared circumstance, one speaks of a multi-threaded process.

In other operating systems, there may be a big distinction between full heavy weight processes and light weight ones; strictly speaking, the heavy weight process may include a number of light weight processes, or just one of them.

In Linux, the situation is quite different. Each thread of execution is considered individually, the difference between heavy and light having to do only with sharing of resources and somewhat faster context switching between threads of execution.

Unlike some other operating systems, Linux has always done an exceptionally fast job of creating, destroying, and switching between processes. Thus, the model adopted for multi-threaded applications resembles multiple processes; each thread is scheduled individually and normally, as if it were a stand-alone process. This is done instead of involving more levels of complication, such as having a separate method of scheduling among the threads of a process, as well as having a scheduling method between different processes.

At the same time, Linux respects POSIX and other standards for multi-threaded processes; e.g., each thread returns the same process ID (called the thread group ID internally), while returning a distinct thread ID (called the process ID internally). This can lead to confusion for developers, but should be invisible to administrators.

What Is a Process?
A process is an instance of a program in execution. It may be in a number of different states, such as running or sleeping. Every process has a pid (Process ID), a ppid (Parent Process ID), and a pgid (Process Group ID). In addition, every process has program code, data, variables, file descriptors, and an environment.

init is usually the first user process run on a system, and thus becomes the ancestor of all subsequent processes running on the system, except for those initiated directly from the kernel (which show up with [] around their name in a ps listing).

If the parent process dies before the child, the ppid of the child is set to 1; i.e., the process is adopted by init. (Note: in recent Linux systems using systemd, the ppid will be set to 2, which corresponds to an internal kernel thread known as kthreadd, which has taken over from init the role of adopter of orphaned children.)

A child process which terminates (either normally or abnormally) before its parent, which has not waited for it and examined its exit code, is known as a zombie (or defunct) process. Zombies have released almost all resources and remain only to convey their exit status. One function of the init process is to check on its adopted children and let those who have terminated die gracefully. Hence, it is sometimes known as the zombie killer, or more grimly, the child reaper.

Processes are controlled by scheduling, which is completely preemptive. Only the kernel has the right to preempt a process; they cannot do it to each other.

For historical reasons, the largest PID has been limited to a 16-bit number, or 32768. It is possible to alter this value by changing /proc/sys/kernel/pid_max, since it may be inadequate for larger servers. As processes are created, eventually they will reach pid_max, at which point they will start again at PID = 300.


Process Attributes
All processes have certain attributes:

The program being executed
Context (state)
Permissions
Associated resources.
Every process is executing some program. At any given moment, the process may take a snapshot of itself by trapping the state of its CPU registers, where it is executing in the program, what is in the process' memory, and other information. This is the context of the process.

Since processes can be scheduled in and out when sharing CPU time with others (or have to be put to sleep while waiting for some condition to be fulfilled, such as the user to make a request or data to arrive), being able to store the entire context when swapping out the process and being able to restore it upon execution resumption is critical to the kernel's ability to do context switching.

Every process has permissions based on which user has called it to execute. It may also have permissions based on who owns its program file. Programs which are marked with an ”s” execute bit have a different ”effective” user id than their ”real” user id. These programs are referred to as setuid programs. They run with the user-id of the user who owns the program, where a non-setuid program runs with the permissions of the user who starts it. setuid programs owned by root can be a security problem.

The passwd command is an example of a setuid program. It is runnable by any user. When a user executes this program, the process runs with root permission in order to be able to update the write-restricted files, /etc/passwd and /etc/shadow, where the user passwords are maintained.

Note that every process has resources such as allocated memory, file handles, etc.

Process Resource Isolation
When a process is started, it is isolated in its own user space to protect it from other processes. This promotes security and creates greater stability.

Processes do not have direct access to hardware. Hardware is managed by the kernel, so a process must use system calls to indirectly access hardware. System calls are the fundamental interface between an application and the kernel.

A system administrator may need to change some of these values in either direction:

* To restrict capabilities so an individual user and/or process cannot exhaust system resources, such as memory, cpu time or the maximum number of processes on the system.
* To expand capabilities so a process does not run into resource limits; for example, a server handling many clients may find that the default of 1024 open files makes its work impossible to perform.
There are two kinds of limits:

Hard
The maximum value, set only by the root user, that a user can raise the resource limit to.
$ ulimit -H -n
4096
Soft
The current limiting value, which a user can modify, but cannot exceed the hard limit.
$ ulimit -S -n
1024
One can set any particular limit by doing:

$ ulimit [options] [limit]

as in

$ ulimit -n 1600

which would increase the maximum number of file descriptors to 1600.

Note that the changes only affect the current shell. To make changes that are effective for all logged-in users, you need to modify /etc/security/limits.conf, a very nicely self-documented file, and then reboot.

Execution Modes
At any given time, a process (or any particular thread of a multi-threaded process) may be executing in either user mode or system mode, which is usually called kernel mode by kernel developers.

What instructions can be executed depends on the mode and is enforced at the hardware, not software, level.

The mode is not a state of the system; it is a state of the processor, as in a multi-core or multi-CPU system each unit can be in its own individual state.

In Intel parlance, user mode is also termed Ring 3, and system mode is termed Ring 0.

 

User Mode
Except when executing a system call, processes execute in user mode, where they have lesser privileges than in the kernel mode.

When a process is started, it is isolated in its own user space to protect it from other processes. This promotes security and creates greater stability. This is sometimes called process resource isolation.

Each process executing in user mode has its own memory space, parts of which may be shared with other processes; except for the shared memory segments, a user process is not able to read or write into or from the memory space of any other process.

Even a process run by the root user or as a setuid program runs in user mode, except when jumping into a system call, and has only limited ability to access hardware.

System (Kernel) Mode
In kernel (system) mode, the CPU has full access to all hardware on the system, including peripherals, memory, disks, etc. If an application needs access to these resources, it must issue a system call, which causes a context switch from user mode to kernel mode. This procedure must be followed when reading and writing from files, creating a new process, etc.

Application code never runs in kernel mode, only the system call itself which is kernel code. When the system call is complete, a return value is produced and the process returns to user mode with the inverse context switch.

There are other times when the system is in kernel mode that have nothing to do with processes, such as when handling hardware interrupts or running the scheduling routines and other management tasks for the system.

Daemons
A daemon process is a background process whose sole purpose is to provide some specific service to users of the system:

* Daemons can be quite efficient because they only operate when needed.
* Many daemons are started at boot time.
* Daemon names often (but not always) end with d.
* Some examples include httpd and systemd-udevd.
* Daemons may respond to external events (systemd-udevd) or elapsed time (crond).
* Daemons generally have no controlling terminal and no standard input/output devices.
* Daemons sometimes provide better security control.
When using SysVinit, scripts in the /etc/init.d directory start various system daemons. These scripts invoke commands as arguments to a shell function named daemon, defined in the /etc/init.d/functions file.


## Creating Processes
An average Linux system is always creating new processes. This is often called forking; the original parent process keeps running, while the new child process starts.

Often, rather than just a fork, one follows it with an exec, where the parent process terminates, and the child process inherits the process ID of the parent. The term fork and exec is used so often, people think of it sometimes as one word.

Older UNIX systems often used a program called spawn, which is similar in many ways to fork and exec, but differs in details. It is not part of the POSIX standard and is not a normal part of Linux.

To see how new processes may start, consider a web server that handles many clients. It may launch a new process every time a new connection is made with a client. On the other hand, it may simply start only a new thread as part of the same process; in Linux, there really is not much difference on a technical level between creating a full process or just a new thread, as each mechanism takes about the same time and uses roughly the same amount of resources.

Creating Processes in a Command Shell
What happens when a user executes a command in a command shell interpreter, such as bash?

A new process is created (forked from the user's login shell).
A wait system call puts the parent shell process to sleep.
The command is loaded onto the child process's space via the exec system call. In other words, the code for the command replaces the bash program in the child process's memory space.
The command completes executing, and the child process dies via the exit system call.
The parent shell is re-awakened by the death of the child process and proceeds to issue a new shell prompt. 
The parent shell then waits for the next command request from the user, at which time the cycle will be repeated.
If a command is issued for background processing (by adding an ampersand -&- at the end of the command line), the parent shell skips the wait request and is free to issue a new shell prompt immediately, allowing the background process to execute in parallel. Otherwise, for foreground requests, the shell waits until the child process has completed or is stopped via a signal.

Some shell commands (such as echo and kill) are built into the shell itself, and do not involve loading of program files. For these commands, no fork or exec are issued for the execution.

As another example, the sshd daemon is started when the init process executes the sshd init script, which then is responsible for launching the sshd daemon. This daemon process listens for ssh requests from remote users.

When a request is received, sshd creates a new copy of itself to service the request. Each remote user gets their own copy of the sshd daemon running to service their remote login. The sshd process will start the login program to validate the remote user. If the authentication succeeds, the login process will fork off a shell (say bash) to interpret the user commands, and so on.

Internal kernel processes take care of maintenance work, such as making sure buffers get flushed out to disk, that the load on different CPUs is balanced evenly, that device drivers handle work that has been queued up for them to do, etc. These processes often run as long as the system is running, sleeping except when they have something to do.

External processes are processes which run in user space like normal applications, but which the kernel started. There are very few of these and they are usually short lived.

It is easy to see which processes are of this nature; when you run a command such as

$ ps -elf

to list all processes on the system while showing the parent process IDs, they will all have PPID = 2, which refers to kthreadd, the internal kernel thread whose job is to create such processes, and their names will be encapsulated in square brackets, such as [ksoftirqd/0].

Static and Shared Libraries
Programs are built using libraries of code, developed for multiple purposes and used and reused in many contexts.

Click to flip each card and learn what the two types of libraries are.

The code for the library functions is inserted in the program at compile time, and does not change thereafter, even if the library is updated.

The code for the library functions is loaded into the program at run time, and if the library is changed later, the running program runs with the new library modifications.

Using shared libraries is more efficient because they can be used by many applications at once; memory usage, executable sizes, and application load time are reduced.

Shared Libraries are also called Dynamic Link Libraries (DLLs).


Shared Library Versions
Shared libraries need to be carefully versioned. If there is a significant change to the library and a program is not equipped to handle it, serious problems can be expected. This is sometimes known as DLL Hell.

Therefore, programs can request a specific major library version, rather than the latest one on the system. However, usually the program will always use the latest minor version available.

Some application providers will use static libraries bundled into the program to avoid these problems. However, if there are improvements or bugs and security holes fixed in the libraries, they may not make it into the applications in a timely fashion.

Shared libraries have the extension .so. Typically, the full name is something like libc.so.N, where N is a major version number.

Under Linux, shared libraries are carefully versioned. For example:

c7:/usr/lib64>ls -lF libgdbm.so*
lrwxrwxrwx 1 root root    16 Apr  9 2015 libgdbm.so -> libgdbm.so.4.0.0*
lrwxrwxrwx 1 root root    16 Apr  9 2015 libgdbm.so.4 -> libgdbm.so.4.0.0*
-rwxr-xr-x 1 root root 36720 Jan 24 2014 libgdbm.so.4.0.0*
c7:/usr/lib64>

so a program that just asks for libgdm gets libgdm.so and the others for specific major and minor versions.

dconfig is generally run at boot time (but can be run anytime), and uses /etc/ld.so.conf, which lists the directories that will be searched for shared libraries. ldconfig must be run as root, and shared libraries should only be stored in system directories when they are stable and useful.

Besides searching the database built up by ldconfig, the linker will first search any directories specified in the environment variable LD_LIBRARY_PATH, a colon separated list of directories, as in the PATH variable. So, you can do:

$ LD_LIBRARY_PATH=$HOME/foo/lib ; foo [args]

or

$ LD_LIBRARY_PATH=$HOME/foo/lib foo [args]

## How git Works
______
Technically, git is not a source control management system in the usual sense, and the basic units it works with are not files. It has two important data structures: an object database and a directory cache.

The object database contains objects of three varieties:

Blobs: Chunks of binary data containing file contents
Trees: Sets of blobs including file names and attributes, giving the directory structure
Commits: Changesets describing tree snapshots.
The directory cache captures the state of the directory tree.

By liberating the controls system from a file-by-file-based system, one is better able to handle changesets which involve many files.

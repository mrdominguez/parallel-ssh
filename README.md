## Synopsis

AUTHOR: Mariano Dominguez, <marianodominguez@hotmail.com>

VERSION: 2.1

FEEDBACK/BUGS: Please contact me by email.

`mdssh.pl` (as in my initials, MD) is an asynchronous parallel SSH/SCP command-line utility that does not require setting up SSH keys.

`mdssh.pl` enables process concurrency and calls `sshexp.pl` and `scpexp.pl` in the background to connect to remote hosts (one host per process) via `ssh` or `scp` respectively.

## Sample Output

Check the status of the the `ntpd` service in `node1`, `node2`, `node3` and `cdsw`:
```
MacBook-Pro:~ mdominguez$ mdssh.pl -s="node{1..3} cdsw" 'service ntpd status'
[node1] [18818] -> OK
Jun 01 08:51:39 node1.localdomain ntpd[8184]: 0.0.0.0 c61c 0c clock_step -2....s
Jun 01 08:51:37 node1.localdomain ntpd[8184]: 0.0.0.0 c614 04 freq_mode
Jun 01 08:51:38 node1.localdomain ntpd[8184]: 0.0.0.0 c618 08 no_sys_peer
Jun 01 09:06:48 node1.localdomain ntpd[8184]: 0.0.0.0 c612 02 freq_set kerne...M
Jun 01 09:06:48 node1.localdomain ntpd[8184]: 0.0.0.0 c615 05 clock_sync
Jun 01 09:54:49 node1.localdomain ntpd[8184]: 0.0.0.0 0613 03 spike_detect -...s
Jun 01 10:07:52 node1.localdomain ntpd[8184]: 0.0.0.0 061c 0c clock_step -0....s
Jun 01 10:07:52 node1.localdomain ntpd[8184]: 0.0.0.0 0615 05 clock_sync
Jun 01 10:07:53 node1.localdomain ntpd[8184]: 0.0.0.0 c618 08 no_sys_peer
Hint: Some lines were ellipsized, use -l to show in full.
[cdsw] [18820] -> Error (rc=3)
Jun 01 09:00:55 cdsw-cdh.cdhdomain ntpd[940]: new interface(s) found: waking...r
Jun 01 09:01:02 cdsw-cdh.cdhdomain ntpd[940]: 0.0.0.0 c61c 0c clock_step -1....s
Jun 01 09:01:00 cdsw-cdh.cdhdomain ntpd[940]: 0.0.0.0 c614 04 freq_mode
Jun 01 09:01:01 cdsw-cdh.cdhdomain ntpd[940]: 0.0.0.0 c618 08 no_sys_peer
Jun 01 09:16:45 cdsw-cdh.cdhdomain ntpd[940]: 0.0.0.0 c612 02 freq_set kerne...M
Jun 01 09:16:45 cdsw-cdh.cdhdomain ntpd[940]: 0.0.0.0 c615 05 clock_sync
Jun 01 10:28:58 cdsw-cdh.cdhdomain ntpd[940]: ntpd exiting on signal 15
Jun 01 10:28:58 cdsw-cdh.cdhdomain systemd[1]: Stopping Network Time Service...
Jun 01 10:28:58 cdsw-cdh.cdhdomain systemd[1]: Stopped Network Time Service.
Hint: Some lines were ellipsized, use -l to show in full.
[node3] [18823] -> OK
Jun 01 08:51:32 node3.localdomain ntpd_intres[8193]: DNS 3.centos.pool.ntp.or...
Jun 01 08:51:33 node3.localdomain ntpd[8186]: Listen normally on 4 eno167777...3
Jun 01 08:51:33 node3.localdomain ntpd[8186]: Listen normally on 5 eno167777...3
Jun 01 08:51:33 node3.localdomain ntpd[8186]: new interface(s) found: waking...r
Jun 01 08:51:41 node3.localdomain ntpd[8186]: 0.0.0.0 c61c 0c clock_step -5....s
Jun 01 08:51:35 node3.localdomain ntpd[8186]: 0.0.0.0 c614 04 freq_mode
Jun 01 08:51:36 node3.localdomain ntpd[8186]: 0.0.0.0 c618 08 no_sys_peer
Jun 01 09:06:55 node3.localdomain ntpd[8186]: 0.0.0.0 c612 02 freq_set kerne...M
Jun 01 09:06:55 node3.localdomain ntpd[8186]: 0.0.0.0 c615 05 clock_sync
Hint: Some lines were ellipsized, use -l to show in full.
ssh: connect to host node2 port 22: No route to host
[node2] (auth) Premature EOF
-----
Number of hosts: 4
~
OK: 2 | node1 node3
~
Error (rc=3): 1 | cdsw
~
Error (rc=255): 1 | node2
MacBook-Pro:~ mdominguez$
```

Restart the `ntpd` service and use verbose output (`-v`), which is especially helpful to track progress when managing hundreds of hosts:
```
MacBook-Pro:~ mdominguez$ mdssh.pl -v -s="node{1..3} cdsw" 'service ntpd restart'
threads = 10
timeout = 20 seconds
o = 1
olines = 10
username = root
Password file /root/ssh_pass found
tcount = 25
ttime = 5 seconds
-----
[node1] [22781] process_1 forked
[node2] [22782] process_2 forked
[node3] [22784] process_3 forked
[cdsw] [22786] process_4 forked
[node3] [22789] -> OK
Redirecting to /bin/systemctl restart  ntpd.service
[node1] [22790] -> OK
Redirecting to /bin/systemctl restart  ntpd.service
[node3] [22784] process_3 exited (Pending: 3 | Forked: 4 | Completed: 1/4 -25%- | OK: 1 | Error: 0)
[node1] [22781] process_1 exited (Pending: 2 | Forked: 4 | Completed: 2/4 -50%- | OK: 2 | Error: 0)
[cdsw] [22792] -> OK
Redirecting to /bin/systemctl restart  ntpd.service
[cdsw] [22786] process_4 exited (Pending: 1 | Forked: 4 | Completed: 3/4 -75%- | OK: 3 | Error: 0)
ssh: connect to host node2 port 22: No route to host
[node2] (auth) Premature EOF
[node2] [22782] process_2 exited (Pending: 0 | Forked: 4 | Completed: 4/4 -100%- | OK: 3 | Error: 1)
All processes completed
-----
Number of hosts: 4
~
OK: 3 | cdsw node1 node3
~
Error (rc=255): 1 | node2
MacBook-Pro:~ mdominguez$
```

## Installation

These utilities are written in Perl and have been tested using *Perl 5.1x.x* on *RHEL 6/7*, as well as *macOS Sierra* and after.

Authentication and credentials are handled using the **Expect.pm** module. The interactive mode functionality in `sshexp.pl` requires **IO::Stty**.

Use [cpan](http://perldoc.perl.org/cpan.html) to install the aforementioned modules; alternately, download them from the [CPAN Search Site](http://search.cpan.org/) for manual installation.

**IMPORTANT: Set the `$shell_prompt` variable in `sshexp.pl` to a regex matching the end of `$PS1` (prompt shell variable) for Expect to correctly catch command execution termination as the default value `'\][\$\#] $'` may not always work**.

## Setting user credentials

The username can be set by using the `-u` option in the command line or the `$SSH_USER` environment variable. If not set, the default username is `$USER`.

The password can be passed by setting the `-p` option or the `$SSH_PASS` environment variable to:
- The actual password string (**not recommended**).
- A file containing the password.

If not set, the password will be undefined.

## Usage

**mdssh.pl**
```
Usage: mdssh.pl [-help] [-version] [-u=username] [-p=password]
    [-sudo[=sudo_user]] [-sshOpts=ssh_options] [-timeout=n] [-threads=n]
    [-scp [-tolocal] [-multiauth] [-r] [-d=target_path] [-meter]]
    [-tcount=throttle_count] [-ttime=throttle_time]
    [-o[=0|1] -olines=n -odir=path] [-v [-timestamp]] (-s="host1 host2 ..." | -f=hosts_file) <command|source_path>

     -help : Display usage
     -version : Display version information
     -u : Username (default: $USER -current user-)
     -p : Password or path to password file (default: undef)
     -sudo : Sudo to sudo_user and run <command> (default: root)
     -sshOpts : Additional SSH options
                (default: -o StrictHostKeyChecking=no -o CheckHostIP=no)
                Example: -sshOpts='-o UserKnownHostsFile=/dev/null -o ConnectTimeout=10'
     -timeout : Timeout value for Expect (default: 20 seconds)
     -threads : Number of concurrent processes (default: 10)
     -scp : Copy <source_path> from local host to @remote_hosts:<target_path>
     -tolocal : Copy @remote_hosts:<source_path> to <target_path> in local host
                The remote hostnames will be appended to <target_path> as a directory
                If permissions allow it, non-existent local directories will be created
     -multiauth : Always authenticate when password prompted (default: single authentication attempt)
     -r : Recursively copy entire directories
     -d : Target path (default: $HOME)
     -meter : Display scp progress (default: disabled)
     -tcount : Number of forked processes before throttling (default: 25)
     -ttime : Throttling time (default: 5 seconds)
     -o : (Not defined) Buffer output; display it after command completion
          (0) Do not display command output
          (1) Display command output as it happens
     -olines : Ignore -o and display the last n lines of buffered output (default: 10 | full output: 0)
     -odir : Local directory in which the command output will be stored as a file (default: $PWD -current folder-)
             If permissions allow it, the directory will be created if it does not exit
     -v : Enable verbose messages / progress information
     -timestamp : Display timestamp
     -s : Space-separated list of hostnames (brace expansion supported)
     -f : File containing hostnames (one per line)
     Set -tcount or -ttime to 0 to disable throttling
     Use environment variables $SSH_USER and $SSH_PASS to pass credentials
     Enable -multiauth along with -tolocal when <source_path> uses brace expansion
     Encase <command> in quotes (single argument)
```

NOTES:
- Once a process is running, a timeout occurs when the executed command does not output anything after the `-timeout` value is reached.
- Unless overridden by the SSH **ConnectTimeout** option, the system's TCP connect timeout value will be used (the default for Linux is 20 seconds). To change it, set `-sshOpts` as follows  `-sshOpts='-o ConnectTimeout=10'` (in seconds).
- Both `-f` and `-s` can be used at the same time.
- Lines containing the `#` character in the hosts file will be skipped.

**sshexp.pl**
```
Usage: sshexp.pl [-help] [-version] [-u=username] [-p=password] [-sudo[=sudo_user]]
    [-sshOpts=ssh_options] [-timeout=n] [-o[=0|1] -olines=n -odir=path] [-v] <host> [<command>]

     -help : Display usage
     -version : Display version information
     -u : Username (default: $USER -current user-)
     -p : Password or path to password file (default: undef)
     -sudo : Sudo to sudo_user and run <command> (default: root)
     -sshOpts : Additional SSH options
                (default: -o StrictHostKeyChecking=no -o CheckHostIP=no)
                Example: -sshOpts='-o UserKnownHostsFile=/dev/null -o ConnectTimeout=10'
     -timeout : Timeout value for Expect (default: 20 seconds)
     -o : (Not defined) Display command output as it happens
          (0) Do not display command output
          (1) Buffer output; display it after command completion (useful for concurrent execution)
     -olines : Ignore -o and display the last n lines of buffered output (default: 10 | full output: 0)
     -odir : Directory in which the command output will be stored as a file (default: $PWD -current folder-)
     -v : Enable verbose messages
     Use environment variables $SSH_USER and $SSH_PASS to pass credentials
     Encase <command> in quotes to pass it as a single argument
     Omit <command> for interactive mode
```
**scpexp.pl**
```
Usage: scpexp.pl [-help] [-version] [-u=username] [-p=password]
    [-sshOpts=ssh_options] [-timeout=n] [-tolocal] [-multiauth] [-r] [-v] [-q] <source_path> <host> [<target_path>]

     -help : Display usage
     -version : Display version information
     -u : Username (default: $USER -current user-)
     -p : Password or path to password file (default: undef)
     -sshOpts : Additional SSH options
                (default: -o StrictHostKeyChecking=no -o CheckHostIP=no)
                Example: -sshOpts='-o UserKnownHostsFile=/dev/null -o ConnectTimeout=10'
     -timeout : Timeout value for Expect (default: 20 seconds)
     -tolocal : Copy from remote host to local host (default: local -> remote)
                If permissions allow it, non-existent local directories in <target_path> will be created
     -multiauth : Always authenticate when password prompted (default: single authentication attempt)
     -q : Quiet mode disables the progress meter (default: enabled)
     -r : Recursively copy entire directories
     -v : Enable verbose messages
     Use environment variables $SSH_USER and $SSH_PASS to pass credentials
     If omitted, <target_path> default value is $HOME
     Enable -multiauth along with -tolocal when <source_path> uses brace expansion
```

## How-To

(Assuming `$SSH_USER` and `$SSH_PASS` have been set)

* Check the OS (RHEL) and kernel version on the remote hosts:

	`$ mdssh.pl -f=hosts 'cat /etc/redhat-release; uname -r'`

* Execute `df -h` and send the output to a file in the local `./df_output` directory:

	`$ mdssh.pl -f=hosts -odir=df_output 'df -h'`

* Push `package.rpm` to `/var/local/tmp` using 3 copy processes:

	`$ mdssh.pl -threads=3 -f=hosts -scp -d=/var/local/tmp package.rpm`

* Install (as root) the package, set timeout to 5 minutes:

	`$ mdssh.pl -timeout=300 -f=hosts -sudo 'rpm -ivh /var/local/tmp/package.rpm'`

* Delete the rpm file:

	`$ mdssh.pl -f=hosts 'rm /var/local/tmp/package.rpm'`

* Pull `/var/log/messages` to the local `./remote_files` directory:

	`$ mdssh.pl -f=hosts -scp -tolocal -d=remote_files /var/log/messages`

<!-- Title: Linux Process-Level Network Monitoring Using eBPF Kernel Probes -->
<!-- Summary: My journey attempting to create a process-specific network bandwidth monitor, similar to Nethogs, using built-in Linux tools and libraries. -->
<!-- Author: jhuckaby -->
<!-- Date: 2024/01/01 -->
<!-- Tags: Networking, Linux, Perl -->

So, I am building a monitoring app for Linux servers, and I need to be able to track and report real-time network throughput (bytes/sec), but do it *per process*.  "This will be easy", I thought.  "I'll just read from the `/proc/` filesystem!  Piece of cake!".

After all, `/proc` has things like `/proc/PID/io` which contains I/O read/write statistics for each process:

```
# cat /proc/18430/io

rchar: 668946374
wchar: 728822907
read_bytes: 90324992
write_bytes: 114073600
```

The first idea I had here was to subtract `read_bytes` and `write_bytes` from `rchar` and `wchar` respectively.  I learned that `read_bytes` and `write_bytes` were specific to the filesystem, but `rchar` and `wchar` were for the entire process I/O (including network!).  So if you subtract the filesystem counters from the total I/O counters, you end up with network throughput for the process, right?

Alas, no, the total process I/O includes more than just disk and network.  It also includes reading/writing to pipes, and TTYs.  So if the process was generating a lot of STDIO or communicating with other processes via pipes, this would skew the measurement.

`/proc/PID/io` was a dead end.

But surely something had to exist in `/proc` for reading process network read/write stats.  I was not giving up so easily.

So next, like many before me, I discovered the `/proc/PID/net/dev` mirage.  This *seems* like it would surely contain network stats for each process:

```
# cat /proc/18430/net/dev

Inter-| Receive                                                         | Transmit
 face | bytes      packets errs drop fifo  frame  compressed multicast  | bytes      packets errs drop fifo colls carrier compressed
    lo: 273390281  277659  0    0    0     0      0          0            273390281  277659  0    0    0    0     0       0
  eth0: 1946372033 3310617 0    0    0     0      0          0            2663685450 3655892 0    0    0    0     0       0
```

I mean, just look at that!  It contains actual byte counters which are counting upwards, and it is located inside the PID-specific directory in `/proc`.  I really thought I hit the jackpot here.

But long story short, this is not what it appears to be.  `/proc/PID/net/dev` contains interface-level stats *from the view of the process* (like if the process was in a network namespace, for example).  I was fooled!

"Okay, fine!", I said, "Forget about process-level.  I'll just track network throughput per each *connection*, and then match those up with processes separately."

I mean, surely the `/proc` filesystem has throughput stats at the connection level.  It must.  I mean, each connection is listed in `/proc/net/tcp` or `/proc/net/tcp6`.  I can just grab network throughput stats from this, right?  **Right?**

```
# cat /proc/net/tcp

  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 00000000:006F 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 16425 1 ffff888037f90000 100 0 0 10 0
   1: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20243 1 ffff888037f94000 100 0 0 10 0
   2: 00000000:0019 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 19572 1 ffff888037f91800 100 0 0 10 0
```

Alas, the answer is still no.  This table has tons of useful information about each socket, including the local and remote IPs, the ports, the state, queue sizes, dropped packets, UID, timeout, and INODE, but it doesn't have byte throughput counters.  Phooey!

"But wait a minute!", I thought, "That table has the INODE!  The socket's INODE number!  Everything in Linux is a file, and therefore each socket is a file!  OMG!  This is it!"

So that led me down the path to `/proc/PID/fd`, which contains a list of all open filehandles for every process, including sockets.  My idea was to read `/proc/net/tcp` and iterate over every socket, looking up the INODE in `/proc/PID/fd` and getting read/write stats for it.  Long story short, this was another dead end.  Linux doesn't expose read/write stats per each file handle in `/proc`.

In the end, I had to throw in the towel.  Not even the latest Linux kernel offers anything in the `/proc` filesystem that tracks network throughput.  Not on the process level, nor on the connection level, nor on the filehandle level.  I was really taken aback by this.

I also tried using standard tools like `netstat` and `lsof`, but could not get network throughput counters of any kind.

## 3rd Party Software

But what about 3rd party software?  Surely something exists that tracks network throughput *per process* which you can just install?

Well, yes, it does.  It's called [Nethogs](https://github.com/raboof/nethogs), and it definitely does the needful.  It even has a machine-readable output mode:

```
# nethogs -t

Refreshing:
curl/949/1001	0.284766	872.055
sshd: jhuckaby@pts/0/1299/1001	1.63164	0.116016
PoolNoodle Server/18430/0	0.0421875	0.046875
/opt/confsync/satellite.bin/714/0	0	0
Performa Server/2421/0	0	0
unknown TCP/0/0	0	0
```

I could theoretically just spawn `nethogs -t` as a child process, and parse the output every second.  That may work...

But herein lies a problem (for me, anyway).  I need to ship my stats collector system out to many servers, and it cannot have any external dependencies.  I guess I could somehow ship a copy of Nethogs inside my stats collector, but there are both technical and legal complications here.  I want to ship a single static binary for one, but also, including a GPLv2 dependency would lead me into sketchy legal territory (licensing restrictions, etc.).

I really want to use built-in Linux tools and libraries, and/or write something myself to accomplish this.

So how does Nethogs work under the hood?  Well, it uses [libpcap](https://github.com/the-tcpdump-group/libpcap) (part of the [tcpdump](https://www.tcpdump.org/) family), and that means packet-level capture and C/C++ development.  I do know C and C++ fairly well, but getting process-level network throughput using packet-level capture in libpcap is extremely tricky, and you have to handle many edge cases.  It would be a long road for me to travel just for one small feature of my app.

"There's got to be a better way!", I said.

I could use the `tcpdump` CLI tool directly, and parse the output in real-time, mapping connections to processes.  But in my experience, running tcpdump slows down the entire server, lags network traffic, and eats up tons of CPU cycles.  Plus, it's another external dependency I'd need my users to install.

At this point I gave up my search for many months.  What I wanted to do just didn't seem feasible, given my restrictions.

## Discovery of eBPF

Then one day I stumbled upon something called [eBPF](https://ebpf.io/), which has the headline:

> Dynamically program the kernel for efficient networking, observability, tracing, and security

Okay, *this* is interesting!  eBPF allows you to run sandboxed programs inside the kernel itself (🤯).  Meaning, it can be used to safely and efficiently extend the capabilities of the kernel without requiring a change in the kernel source code or loading kernel modules.

Incredible, but, how can I use this?  Me, a mere mortal, who doesn't know anything about Linux kernel development.  Well, enter [bpftrace](https://github.com/iovisor/bpftrace), which is written atop eBPF:

> bpftrace is a high-level tracing language for Linux enhanced Berkeley Packet Filter (eBPF) available in recent Linux kernels (4.x).

Here is where the rubber meets the road.  Using bpftrace, you can write a very simple program in an easy-to-learn scripting language, which is compiled into the kernel sandbox environment and runs as native code.  Check out this simple example script:

```c
kprobe:tcp_connect
{
	printf("A TCP socket has connected!\n");
}
```

Using the bpftrace scripting language, you can hook almost any kernel function (in this example `tcp_connect`), and perform your own logic.  You also have full access to the arguments passed to the kernel function, and you can traverse into structures (like `tcp_sock` for example).  You can then aggregate stats and output data in several formats (including JSON).  Once launched, your script runs continually until it is killed.

In fact, bpftrace ships with an example script which *almost* gives me exactly what I need.  Check out [tcplife.bt](https://github.com/iovisor/bpftrace/blob/master/tools/tcplife.bt), which shows the lifespan of TCP sessions, including total throughput statistics:

```
# ./tcplife.bt

PID   COMM       LADDR           LPORT RADDR           RPORT  TX_KB RX_KB MS
20976 ssh        127.0.0.1       56766 127.0.0.1       22         6 10584 3059
20977 sshd       127.0.0.1       22    127.0.0.1       56766  10584     6 3059
14519 monitord   127.0.0.1       44832 127.0.0.1       44444      0     0 0
```

Lord almighty, this comes so, *so* close!  It lists byte count throughput per connection and includes the PIDs!  Finally!  The problem is, this script only outputs when a connection *closes*.  It won't work for me because I need to report real-time network throughput statistics every second, including long-lived connections.

But I figured it *must* be possible to achieve what I need with bpftrace.  So I rolled up my sleeves, and learned the language.  It's easy to pick up, and has primitives like hash tables which come in very handy.  There's a [tutorial](https://github.com/iovisor/bpftrace/blob/master/docs/tutorial_one_liners.md), a nice [reference guide](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md), and [lots of examples](https://github.com/iovisor/bpftrace/tree/master/tools).

## My Script

What I ended up doing was extending the `tcplife.bt` script, and probing two additional kernel functions: `tcp_sendmsg` and `tcp_recvmsg`.  In these, I keep track of the connection read/write byte counters in two separate BPF hash maps:

```c
kprobe:tcp_sendmsg,
kprobe:tcp_recvmsg
{
	$sk = (struct sock *)arg0;
	
	if (@birth[$sk]) {
		$tp = (struct tcp_sock *)$sk;
		@sktx[$sk] = $tp->bytes_acked;
		@skrx[$sk] = $tp->bytes_received;
	}
}
```

These hash maps are all keyed by the socket memory address.  I also keep track of the PID and process name in other hashes, and print them all to STDOUT every second using a bpftrace [interval timer](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#12-interval-timed-output):

```c
interval:s:1
{
	print(@skcomm);
	print(@skpid);
	print(@sktx);
	print(@skrx);
}
```

When the socket closes, I remove the associated keys from all the hashes.  I also print a machine-readable "close" event when this happens, so I know which sockets closed and when:

```c
printf("CLOSE: %x: PID %d, %s, TX %d, RX %d, %d MS\n", $sk, $pid, $comm, $tp->bytes_acked, $tp->bytes_received, $delta_ms);
```

The raw output of the script looks absolutely hideous, but it does have all the "ingredients" I need:

```
@skcomm[0xffff88800392e800]: Performa Server
@skcomm[0xffff888003929000]: Mendo Server
@skcomm[0xffff888003928800]: node
@skcomm[0xffff88800865f800]: PoolNoodle Serv

@skpid[0xffff88800392e800]: 2421
@skpid[0xffff888003929000]: 2863
@skpid[0xffff888003928800]: 17678
@skpid[0xffff88800865f800]: 18430

@sktx[0xffff888003928800]: 0
@sktx[0xffff88800392e800]: 1
@sktx[0xffff888003929000]: 1
@sktx[0xffff88800865f800]: 2

@skrx[0xffff888003928800]: 0
@skrx[0xffff88800392e800]: 6
@skrx[0xffff888003929000]: 6
@skrx[0xffff88800865f800]: 44

CLOSE: 865a000: PID 17868, curl, TX 0, RX 10, 55 MS
CLOSE: 4b65500: PID 18430, PoolNoodle Serv, TX 10, RX 0, 55 MS
CLOSE: 3928800: PID 17678, node, TX 13, RX 0, 1162 MS
CLOSE: 38100880: PID 2421, Performa Server, TX 0, RX 13, 1162 MS
```

It's actually quite a bit cleaner in JSON output mode, which you activate by passing `-f json` to the bpftrace binary:

```js
{"type": "map", "data": {"@skcomm": {"0xffff88800392e000": "Mendo Server", "0xffff888003928800": "PoolNoodle Serv"}}}
{"type": "map", "data": {"@skpid": {"0xffff88800392e000": 2863, "0xffff888003928800": 18430}}}
{"type": "map", "data": {"@sktx": {"0xffff88800392e000": 1, "0xffff888003928800": 1, "0xffff888038105500": 12}}}
{"type": "map", "data": {"@skrx": {"0xffff888038105500": 2, "0xffff88800392e000": 6, "0xffff888003928800": 12}}}
```

The idea here is that the `@sktx` and `@skrx` hashes contain total network transmit and receive byte counts since each connection opened.  By reading these continuously I can calculate deltas (i.e. read/write throughput per second), and then map connections back to their respective PIDs and process names, by using the hash keys and looking up the values in `@skcomm` and `@skpid`.

## Perl Wrapper

However, this is about as far as I could get with bpftrace directly, so what I did next was write a little [Perl](https://www.perl.org/) wrapper script around it, which spawns bpftrace as a child process, communicates over STDIO, and performs the final aggregation and process-level reporting every second.  I chose Perl for this because it uses very little memory (8MB or so), and it comes preinstalled on most Linux distros.  Plus it's really designed for this sort of thing (I mean, it stands for "Practical Extraction and Reporting Language", right?).

The final script is called `net-proc-watch`, and is up on GitHub here:

https://github.com/pixlcore/net-proc-watch

The output looks like this, and it updates every second:

```
# net-proc-watch

PID, COMMAND, CONNS, TX_SEC, RX_SEC
30408, curl, 1, 0 bytes/sec, 96.86 K/sec
2863, Mendo Server, 1, 373 bytes/sec, 100 bytes/sec
2421, Performa Server, 2, 141.97 K/sec, 3.35 K/sec
30466, node, 1, 333 bytes/sec, 697 bytes/sec
18430, PoolNoodle Serv, 1, 8.89 K/sec, 83 bytes/sec
```

It can also output in JSON format, for machine-readability.  Hooray!

So is this the Holy Grail?  Did I achieve my goal?  Well, yes, and no.  Yes, it works quite well, runs on all the Linux flavors I tried (see [Tested Using](https://github.com/pixlcore/net-proc-watch#tested-using)), and seems to be accurate and matches up with Nethogs, more or less.  It uses practically zero CPU, and even provides things that Nethogs doesn't, like the number of connections per process.  Finally, it's all built on eBPF, bpftrace and Perl, all of which have no licensing restrictions.  This is all great stuff!  But...

## Caveats

It has two major caveats:

- **Memory Usage**
	- Currently, as of this writing, bpftrace scripts require about *130 MB* of memory to run.
	- Compare this to Nethogs which only uses around 10 MB or so.
- **Compiler Toolchain**
	- Currently, bpftrace scripts require that the *entire LLVM toolchain* be installed on the machines where it runs.
	- This is obviously a difficult sell for production servers.

These are unfortunate, and that darn compiler toolchain requirement is basically a showstopper for me.  I can't ship my stats collector and ask my users to install all those dependencies on their production servers.  The full package dependency list for bpftrace on a fresh Linux install is a tad bit insane:

<details><summary>bpftrace Package Dependencies</summary>

```
===========================================================================================================================
 Package                                   Architecture       Version                           Repository            Size
===========================================================================================================================
Installing:
 bpftrace                                  aarch64            0.17.0-2.el9                      appstream            1.9 M
Installing dependencies:
 bcc                                       aarch64            0.26.0-4.el9                      appstream            578 k
 binutils                                  aarch64            2.35.2-42.el9                     baseos               4.8 M
 binutils-gold                             aarch64            2.35.2-42.el9                     baseos               884 k
 bison                                     aarch64            3.7.4-5.el9                       appstream            933 k
 checkpolicy                               aarch64            3.6-0.rc1.1.el9                   appstream            348 k
 clang-libs                                aarch64            17.0.1-2.el9                      appstream             88 M
 clang-resource-filesystem                 noarch             17.0.1-2.el9                      appstream             10 k
 cpp                                       aarch64            11.4.1-2.3.el9                    appstream             10 M
 elfutils-debuginfod-client                aarch64            0.190-1.el9                       baseos                37 k
 elfutils-libelf-devel                     aarch64            0.190-1.el9                       appstream             23 k
 environment-modules                       aarch64            5.3.0-1.el9                       baseos               588 k
 flex                                      aarch64            2.6.4-9.el9                       appstream            306 k
 gcc                                       aarch64            11.4.1-2.3.el9                    appstream             30 M
 gcc-toolset-13-binutils                   aarch64            2.40-15.el9                       appstream            6.0 M
 gcc-toolset-13-binutils-gold              aarch64            2.40-15.el9                       appstream            940 k
 gcc-toolset-13-gcc                        aarch64            13.2.1-5.el9                      appstream             38 M
 gcc-toolset-13-gcc-c++                    aarch64            13.2.1-5.el9                      appstream             12 M
 gcc-toolset-13-libstdc++-devel            aarch64            13.2.1-5.el9                      appstream            3.5 M
 gcc-toolset-13-runtime                    aarch64            13.0-2.el9                        appstream             28 k
 glibc-devel                               aarch64            2.34-88.el9                       appstream            562 k
 groff-base                                aarch64            1.22.4-10.el9                     baseos               1.0 M
 jansson                                   aarch64            2.14-1.el9                        baseos                47 k
 kernel-headers                            aarch64            5.14.0-391.el9                    appstream            7.0 M
 less                                      aarch64            590-2.el9                         baseos               161 k
 libasan                                   aarch64            11.4.1-2.3.el9                    appstream            412 k
 libmpc                                    aarch64            1.2.1-4.el9                       appstream             63 k
 libpipeline                               aarch64            1.5.3-4.el9                       baseos                48 k
 libpkgconf                                aarch64            1.7.3-10.el9                      baseos                36 k
 libubsan                                  aarch64            11.4.1-2.3.el9                    appstream            185 k
 libxcrypt-devel                           aarch64            4.4.18-3.el9                      appstream             29 k
 libzstd-devel                             aarch64            1.5.1-2.el9                       appstream             47 k
 llvm-libs                                 aarch64            17.0.1-3.el9                      appstream             46 M
 m4                                        aarch64            1.4.19-1.el9                      appstream            297 k
 make                                      aarch64            1:4.3-7.el9                       baseos               535 k
 man-db                                    aarch64            2.9.3-7.el9                       baseos               1.2 M
 ncurses                                   aarch64            6.2-10.20210508.el9               baseos               399 k
 openssl-devel                             aarch64            1:3.0.7-25.el9                    appstream            4.1 M
 perl-AutoLoader                           noarch             5.74-480.el9                      appstream             22 k
 perl-B                                    aarch64            1.80-480.el9                      appstream            182 k
 perl-Carp                                 noarch             1.50-460.el9                      appstream             30 k
 perl-Class-Struct                         noarch             0.66-480.el9                      appstream             23 k
 perl-Data-Dumper                          aarch64            2.174-462.el9                     appstream             55 k
 perl-Digest                               noarch             1.19-4.el9                        appstream             26 k
 perl-Digest-MD5                           aarch64            2.58-4.el9                        appstream             37 k
 perl-Encode                               aarch64            4:3.08-462.el9                    appstream            1.7 M
 perl-Errno                                aarch64            1.30-480.el9                      appstream             16 k
 perl-Exporter                             noarch             5.74-461.el9                      appstream             32 k
 perl-Fcntl                                aarch64            1.13-480.el9                      appstream             21 k
 perl-File-Basename                        noarch             2.85-480.el9                      appstream             18 k
 perl-File-Path                            noarch             2.18-4.el9                        appstream             36 k
 perl-File-Temp                            noarch             1:0.231.100-4.el9                 appstream             60 k
 perl-File-stat                            noarch             1.09-480.el9                      appstream             18 k
 perl-FileHandle                           noarch             2.03-480.el9                      appstream             17 k
 perl-Getopt-Long                          noarch             1:2.52-4.el9                      appstream             61 k
 perl-Getopt-Std                           noarch             1.12-480.el9                      appstream             17 k
 perl-HTTP-Tiny                            noarch             0.076-461.el9                     appstream             54 k
 perl-IO                                   aarch64            1.43-480.el9                      appstream             89 k
 perl-IO-Socket-IP                         noarch             0.41-5.el9                        appstream             43 k
 perl-IO-Socket-SSL                        noarch             2.073-1.el9                       appstream            219 k
 perl-IPC-Open3                            noarch             1.21-480.el9                      appstream             24 k
 perl-MIME-Base64                          aarch64            3.16-4.el9                        appstream             31 k
 perl-Mozilla-CA                           noarch             20200520-6.el9                    appstream             13 k
 perl-Net-SSLeay                           aarch64            1.92-2.el9                        appstream            389 k
 perl-POSIX                                aarch64            1.94-480.el9                      appstream             98 k
 perl-PathTools                            aarch64            3.78-461.el9                      appstream             88 k
 perl-Pod-Escapes                          noarch             1:1.07-460.el9                    appstream             21 k
 perl-Pod-Perldoc                          noarch             3.28.01-461.el9                   appstream             87 k
 perl-Pod-Simple                           noarch             1:3.42-4.el9                      appstream            225 k
 perl-Pod-Usage                            noarch             4:2.01-4.el9                      appstream             41 k
 perl-Scalar-List-Utils                    aarch64            4:1.56-461.el9                    appstream             72 k
 perl-SelectSaver                          noarch             1.02-480.el9                      appstream             13 k
 perl-Socket                               aarch64            4:2.031-4.el9                     appstream             55 k
 perl-Storable                             aarch64            1:3.21-460.el9                    appstream             94 k
 perl-Symbol                               noarch             1.08-480.el9                      appstream             15 k
 perl-Term-ANSIColor                       noarch             5.01-461.el9                      appstream             49 k
 perl-Term-Cap                             noarch             1.17-460.el9                      appstream             23 k
 perl-Text-ParseWords                      noarch             3.30-460.el9                      appstream             17 k
 perl-Text-Tabs+Wrap                       noarch             2013.0523-460.el9                 appstream             24 k
 perl-Time-Local                           noarch             2:1.300-7.el9                     appstream             34 k
 perl-URI                                  noarch             5.09-3.el9                        appstream            121 k
 perl-base                                 noarch             2.27-480.el9                      appstream             17 k
 perl-constant                             noarch             1.33-461.el9                      appstream             24 k
 perl-if                                   noarch             0.60.800-480.el9                  appstream             15 k
 perl-interpreter                          aarch64            4:5.32.1-480.el9                  appstream             72 k
 perl-libnet                               noarch             3.13-4.el9                        appstream            130 k
 perl-libs                                 aarch64            4:5.32.1-480.el9                  appstream            2.1 M
 perl-mro                                  aarch64            1.23-480.el9                      appstream             29 k
 perl-overload                             noarch             1.31-480.el9                      appstream             47 k
 perl-overloading                          noarch             0.02-480.el9                      appstream             14 k
 perl-parent                               noarch             1:0.238-460.el9                   appstream             15 k
 perl-podlators                            noarch             1:4.14-460.el9                    appstream            114 k
 perl-subs                                 noarch             1.03-480.el9                      appstream             13 k
 perl-vars                                 noarch             1.05-480.el9                      appstream             14 k
 pkgconf                                   aarch64            1.7.3-10.el9                      baseos                40 k
 pkgconf-m4                                noarch             1.7.3-10.el9                      baseos                15 k
 pkgconf-pkg-config                        aarch64            1.7.3-10.el9                      baseos                11 k
 policycoreutils-python-utils              noarch             3.6-0.rc1.1.el9                   appstream             77 k
 python3-audit                             aarch64            3.1.2-2.el9                       appstream             83 k
 python3-bcc                               noarch             0.26.0-4.el9                      appstream             97 k
 python3-distro                            noarch             1.5.0-7.el9                       baseos                37 k
 python3-libselinux                        aarch64            3.6-0.rc1.1.el9                   appstream            184 k
 python3-libsemanage                       aarch64            3.6-0.rc1.1.el9                   appstream             79 k
 python3-netaddr                           noarch             0.8.0-5.el9                       appstream            1.6 M
 python3-policycoreutils                   noarch             3.6-0.rc1.1.el9                   appstream            2.1 M
 python3-setools                           aarch64            4.4.3-1.el9                       baseos               594 k
 python3-setuptools                        noarch             53.0.0-12.el9                     baseos               944 k
 scl-utils                                 aarch64            1:2.0.3-4.el9                     appstream             37 k
 tar                                       aarch64            2:1.34-6.el9                      baseos               874 k
 tcl                                       aarch64            1:8.6.10-7.el9                    baseos               1.1 M
 vim-filesystem                            noarch             2:8.2.2637-20.el9                 baseos                18 k
 zlib-devel                                aarch64            1.2.11-41.el9                     appstream             45 k
 bcc-tools                                 aarch64            0.26.0-4.el9                      appstream            549 k
 compiler-rt                               aarch64            17.0.1-1.el9                      appstream            2.0 M
 kernel-devel                              aarch64            5.14.0-391.el9                    appstream             20 M
 libatomic                                 aarch64            11.4.1-2.3.el9                    baseos                37 k
 libomp                                    aarch64            17.0.1-2.el9                      appstream            647 k
 libomp-devel                              aarch64            17.0.1-2.el9                      appstream            317 k
 perl-NDBM_File                            aarch64            1.15-480.el9                      appstream             23 k

Transaction Summary
===========================================================================================================================
Install  119 Packages

Total download size: 300 M
Installed size: 1.1 G
Is this ok [y/N]: 
```

</details>

300 MB of packages to download, which end up using 1.1 GB of disk space.  Ouch.

## AOT Compilation

But there is hope on the horizon!  Right now, as I write this, [smart people](https://dxuuu.xyz/about.html) are working on AOT (ahead-of-time) compilation for bpftrace!  From [what I am told](https://github.com/iovisor/bpftrace/discussions/2892) this will solve both of my caveats.  Memory usage should be reduced, and you will be able to ship a compiled binary that doesn't require all those compiler dependencies.

This is really exciting news.  Check out the work in progress here:

https://dxuuu.xyz/aot-bpftrace.html

## Other eBPF Libraries

There are several other libraries written atop eBPF that I haven't checked out yet.  They are:

- [BCC Python Library](https://github.com/iovisor/bcc)
- [eBPF Go Library](https://github.com/cilium/ebpf)
- [libbpf C/C++ Library](https://github.com/libbpf/libbpf)

It is possible that these avoid the dependency and memory caveats of bpftrace, but I haven't had the time to learn them all.

## Conclusion

In summary, [eBPF](https://ebpf.io/) is a very powerful and exciting technology, and [bpftrace](https://github.com/iovisor/bpftrace) greatly eases the learning curve.  bpftrace scripts are easy to write, and they work across a wide variety of Linux kernels, including ARM64.  I believe this will absolutely become my process-level network monitoring solution, once I can precompile a binary for it.

Thanks for reading, and stay tuned!

## Addendum

Welp.  After finishing this article, I took a quick look at [BCC](https://github.com/iovisor/bcc) just to see what it was about, and in short, my jaw is on the floor.  Brendan Gregg already created what I just created, and he did it way back in 2016.  It's called [tcptop.py](https://github.com/iovisor/bcc/blob/master/tools/tcptop.py), and is built using BCC.  His script is way better than mine, too:

```
# tcptop
Tracing... Output every 1 secs. Hit Ctrl-C to end
19:46:24 loadavg: 1.86 2.67 2.91 3/362 16681

PID    COMM         LADDR                 RADDR                  RX_KB  TX_KB
16648  16648        100.66.3.172:22       100.127.69.165:6684        1      0
16647  sshd         100.66.3.172:22       100.127.69.165:6684        0   2149
14374  sshd         100.66.3.172:22       100.127.69.165:25219       0      0
14458  sshd         100.66.3.172:22       100.127.69.165:7165        0      0
```

Gosh darn it!  If only I had done a *little* more research, it would have saved me a day of reinventing the wheel.  But hey, I learned a lot, and it was fun.

Also, I need to point out that `tcptop.py` still suffers from both of my caveats listed above.  Admittedly it uses less memory than my script (around 80 MB, down from 130 MB), but 80 MB is still a lot for something like this, and it also requires a complete LLVM compiler toolchain to run.  So, let's just say I'm still looking forward to AOT compilation.

## Addendum 2

Welp.  My jaw is on the floor *again*.  I just discovered that [ss](https://man7.org/linux/man-pages/man8/ss.8.html) (the modern replacement for [netstat](https://man7.org/linux/man-pages/man8/netstat.8.html)) already has network throughput reporting built right into it, and has had for many years.  You just need to pass it the correct flags:

```
# ss -ntip

State         Recv-Q           Send-Q                   Local Address:Port                      Peer Address:Port          Process
ESTAB         2305184          0                          172.30.0.26:38290                   52.219.194.147:80             users:(("curl",pid=19401,fd=5))
	 cubic wscale:8,7 rto:204 rtt:0.164/0.082 ato:80 mss:1432 pmtu:9001 rcvmss:1432 advmss:8961 cwnd:10 bytes_acked:121 bytes_received:13481491 segs_out:241 segs_in:9457 data_segs_out:1 data_segs_in:9439 send 698536585bps lastsnd:10168 lastrcv:544 lastack:520 pacing_rate 1397073168bps delivery_rate 44925488bps app_limited rcv_rtt:0.295 rcv_space:557048 rcv_ssthresh:1353570 minrtt:0.151
```

Notice the `bytes_acked:121 bytes_received:13481491` in the output, which is reported *per connection*.  It also can include the PID and process name for each connection!

I guess I've been living under a rock for the last decade, while Linux reinvented itself.

Anyway, this is awesome.  I can theoretically just poll `ss -ntip` every few seconds, and I basically have my process network monitor, more or less.  The `ss` command comes preinstalled on most modern Linux distros, and it's extremely fast.  Of course, polling won't catch short-lived connections, but I believe could be a viable solution for my needs, at least for now.

I'm still going to keep an eye on eBPF and AOT compilation, because I believe that will ultimately be better than polling `ss`.  Not only will it catch short-lived connections, but it will likely consume fewer resources with high network traffic (i.e. tens of thousands of open connections).

---
title: "Trace, Common Trace Format, gdb, ebpf"
categories:
  - blog
tags:
  - trace
  - ctf
  - gdb
  - ebpf
  - bumblebee
  - 
---
I investigate a bit on tracing today. These are some interesting posts I read. 
- [Trace Visualization: Analyze perf, bpf, kernel and userspace traces with Trace Compass](http://blog.linuxplumbersconf.org/2017/ocw/sessions/4780.html)
>The Common Trace Format (CTF) is a compact, yet flexible, binary trace format used by tracers. LTTng, perf, gdb and now ebpf can generate traces in this format.
I heard about CTF, but did not know that LTTng, gdb and ebpf can emit CTF trace data.

Selected blog posts from LTTng:
- [barectf 2: Continuous Bare-Metal Tracing on the Parallella Board](https://lttng.org/blog/2015/07/21/barectf-2/)
- [Tutorial: Remotely tracing an embedded Linux system](https://lttng.org/blog/2016/03/07/tutorial-remote-tracing/)
- [Debugging Python applications on Linux with LTTng-UST](https://lttng.org/blog/2019/07/16/debugging-python-applications/)
- [LTTng session rotation: the practical way to do continuous tracing](https://lttng.org/blog/2019/10/15/lttng-session-rotation/)
- [The new dynamic user space tracing feature in LTTng](https://lttng.org/blog/2019/10/15/new-dynamic-user-space-tracing-in-lttng/)

The current version of barectf is 3 [barectf.org](barectf.org)

LTTng post about user space tracing feature has this link inside the post.
[Adding User Space Probing to an Application](https://sourceware.org/systemtap/wiki/AddingUserSpaceProbingToApps) This is a generic step for planting probe points throughout your code. It is applicable even if you use ebpf, instead of lttng, to trigger any action.

[Using user-space tracepoints with BPF](https://lwn.net/Articles/753601/)
[What is BPF and why is it taking over Linux Performance Analysis?](https://www.singlestore.com/blog/bpf-linux-performance/)
[Solo.io Open Sources BumbleBee to Make eBPF Development Easier](https://www.infoq.com/news/2022/02/bumblebee/)

gdb trace requires gdbserver. gdbserver mentions LTTng Userspace Tracer [link](https://sourceware.org/gdb/onlinedocs/gdb/Server.html#Server)
>20.3.4 Tracepoints support in gdbserver
I think the gdbserver must be built to support UST mechanism:
>This support is automatically available if UST development headers are found in the standard include path when gdbserver is built, or if gdbserver was explicitly configured using --with-ust to point at such headers. You can explicitly disable the support using --with-ust=no.

I think your application must also support UST mechanism:
>There are several ways to load the in-process agent in your program:
>
>Specifying it as dependency at link time
>You can link your program dynamically with the in-process agent library. On most systems, this is accomplished by adding -linproctrace to the link command.
>
>Using the system's preloading mechanisms
>You can force loading the in-process agent at startup time by using your system’s support for preloading shared libraries. Many Unixes support the concept of preloading user defined libraries. In most cases, you do that by specifying LD_PRELOAD=libinproctrace.so in the environment. See also the description of gdbserver’s --wrapper command line option.

I got the impression if the above two conditons are met, gdb/gdbserver allows you to define trace point and then you can capture trace. i.e. You do not have to plant UST probes throughout your source code. (It will be much easier if you do.)

[Fast Tracing with GDB](https://suchakra.wordpress.com/2016/06/29/fast-tracing-with-gdb/) explains the meaning of fast tracing mentioned in the gdb documentation. However, I start to wonder if I already planted my UST probes, then there should be no need for dynamic patching. Hence, I am no longer certain if I need libinproctrace.so for my application.
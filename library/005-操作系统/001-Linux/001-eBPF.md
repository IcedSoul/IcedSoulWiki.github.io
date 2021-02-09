# eBPF

## 什么是eBPF

eBPF的介绍可以参考[1][https://cloudnative.to/blog/bpf-intro/],[2][https://ebpf.io/],简单看来，eBPF是一种通用执行引擎，使得开发者可以在Linux内核中执行特定的代码，从而基于eBPF设计实现丰富的上层应用。目前很多公司已经在观测、故障排查领域基于eBPF实现各种应用用于性能分析、故障排查等功能。

我们同样想基于eBPF来实现应用程序以提高微服务系统的可观测性，因为有必要先了解一下eBPF相关的知识点，并且了解一下基于eBPF实现的工具，并且进行尝试。

> The Linux kernel has always been an ideal place to implement monitoring/observability, networking, and security. Unfortunately this was often impractical as it required changing kernel source code or loading kernel modules, and resulted in layers of abstractions stacked on top of each other. eBPF is a revolutionary technology that can run sandboxed programs in the Linux kernel without changing kernel source code or loading kernel modules.

通过官方文档可以认识到eBPF似乎很厉害，但是它不是一种编程语言或者第三方以来，使用起来并非易事。通过官网列出的相关项目，我找到了bpftrace[3]和kubectl-trace[4]来进一步了解eBPF本身。

根据介绍，bpftrace是一门实用eBPF新特性的高级编程语言，先根据文档简单试用一下。



## 参考资料

1. eBPF技术简介：https://cloudnative.to/blog/bpf-intro/
2. eBPF offical site：https://ebpf.io/
3. bpftrace：https://bpftrace.org/
4. kubectl-trace：https://github.com/iovisor/kubectl-trace
5. eBPF相关知识介绍博客：https://blog.csdn.net/zhaowei121/article/details/90639945?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-3&spm=1001.2101.3001.4242
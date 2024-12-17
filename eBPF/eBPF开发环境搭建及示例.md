
# 环境

```bash
# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.4 LTS
Release:	22.04
Codename:	jammy
# uname -a
Linux ubu22 5.15.0-100-generic #110-Ubuntu SMP Wed Feb 7 13:28:04 UTC 2024 aarch64 aarch64 aarch64 GNU/Linux
```

# 包安装

*安装软件包*

```bash
apt install -y bison flex build-essential git cmake make libelf-dev strace tar libfl-dev libssl-dev libedit-dev zlib1g-dev  python  python3-distutils libcap-dev llvm clang
```

## 安装 bcc

`bcc` 是一组用于 `eBPF` 开发的工具，它包括了一组用于编写和调试 `eBPF` 程序的库和命令行工具。使用 `bcc`，可以更加方便地开发和调试 `eBPF` 程序，提高开发效率和代码质量。

```bash
apt install bpfcc-tools
```

## 安装 bpftool

`bpftool` 是一个用于管理和调试 `eBPF` 代码的命令行工具。它允许你查看和分析系统中运行的 `eBPF` 程序和映射，以及与内核中的 `eBPF` 子系统进行交互。更多内容可以查看 [bpftool Github](https://github.com/libbpf/bpftool)

使用 `bpftool`，你可以执行以下操作：

* 列出当前系统中所有加载的 `eBPF` 程序和映射
* 查看指定 `eBPF` 程序或映射的详细信息，例如指令集、内存布局等
* 修改 `eBPF` 程序或映射的属性，例如禁用一个程序或清空一个映射
* 将一个 `eBPF` 程序或映射导出到文件中，以便在其他系统上重新导入
* 调试 `eBPF` 程序，例如跟踪程序的控制流、访问内存等

```bash
apt install -y linux-tools-$(uname -r)
```

默认情况下 `bpftool` 命令会安装到/`usr/local/sbin/` 下

```bash
# bpftool version -p
{
    "version": "5.15.143",
    "features": {
        "libbfd": false,
        "skeletons": false
    }
}
```


## 安装linux源码

```bash
apt install linux-source-5.15.0
```

# 编译bpf sample


*解压缩*

```bash
cd /usr/src/linux-source-5.15.0/debian
tar -jxvf linux-source-5.15.0.tar.bz2
```

*Copy config文件*

```bash
cd linux-source-5.15.0
cp -v /boot/config-$(uname -r) .config
make oldconfig && make prepare
```


*编译Samples*

```bash
make -C samples/bpf
```

若遇到如下错误

```bash
......
  CC      /usr/src/linux-source-5.15.0/samples/bpf/bpftool/prog.o
  LINK    /usr/src/linux-source-5.15.0/samples/bpf/bpftool/bpftool
/usr/src/linux-source-5.15.0/samples/bpf/Makefile:369: *** Cannot find a vmlinux for VMLINUX_BTF at any of "  /usr/src/linux-source-5.15.0/vmlinux", build the kernel or set VMLINUX_BTF or VMLINUX_H variable.  Stop.
make[1]: *** [Makefile:1911: /usr/src/linux-source-5.15.0/samples/bpf] Error 2
make[1]: Leaving directory '/usr/src/linux-source-5.15.0'
make: *** [Makefile:275: all] Error 2
make: Leaving directory '/usr/src/linux-source-5.15.0/samples/bpf'
```

说明需要 vmlinux，可以指定vmlinux再进行编译

```bash
make VMLINUX_BTF=/sys/kernel/btf/vmlinux -C samples/bpf
```

or

```bash
make VMLINUX_BTF=/sys/kernel/btf/vmlinux M=samples/bpf
```

# 示例

示例需要在 `usr/src/linux-source-5.15.0/samples/bpf`目录下进行编译。`samples/bpf` 下的程序一般组成方式是 xxx_user.c 和 xxx_kern.c
 * xxx_user.c：为用户空间的程序用于设置 BPF 程序的相关配置、加载 BPF 程序至内核、设置 BPF 程序中的 map 值和读取 BPF 程序运行过程中发送至用户空间的消息等。
 * xxx_kern.c：为 BPF 程序代码，通过 clang 编译成字节码加载至内核中，在对应事件触发的时候运行，可以接受用户空间程序发送的各种数据，并将运行时产生的数据发送至用户空间程序。

新建两个文件：`bob_user.c`和`bob_kern.c`

*bob_kern.c*

```C
#include <uapi/linux/bpf.h>
#include <linux/version.h>
#include <bpf/bpf_helpers.h>
#include <bpf/bpf_tracing.h>

SEC("tracepoint/syscalls/sys_enter_execve")
int bpf_prog(struct pt_regs *ctx)
{
	char fmt[] = "bob %s !\n";
	char comm[16];
	bpf_get_current_comm(&comm, sizeof(comm));
	bpf_trace_printk(fmt, sizeof(fmt), comm);

	return 0;
}

char _license[] SEC("license") = "GPL";
u32 _version SEC("version") = LINUX_VERSION_CODE;
```

*bob_user.c*

```C
#include <stdio.h>
#include <unistd.h>
#include <bpf/libbpf.h>
#include "trace_helpers.h"

int main(int ac, char **argv)
{
	struct bpf_link *link = NULL;
	struct bpf_program *prog;
	struct bpf_object *obj;
	char filename[256];

	snprintf(filename, sizeof(filename), "%s_kern.o", argv[0]);
	obj = bpf_object__open_file(filename, NULL);
	if (libbpf_get_error(obj)) {
		fprintf(stderr, "ERROR: opening BPF object file failed\n");
		return 0;
	}

	prog = bpf_object__find_program_by_name(obj, "bpf_prog");
	if (!prog) {
		fprintf(stderr, "ERROR: finding a prog in obj file failed\n");
		goto cleanup;
	}

	/* load BPF program */
	if (bpf_object__load(obj)) {
		fprintf(stderr, "ERROR: loading BPF object file failed\n");
		goto cleanup;
	}

	link = bpf_program__attach(prog);
	if (libbpf_get_error(link)) {
		fprintf(stderr, "ERROR: bpf_program__attach failed\n");
		link = NULL;
		goto cleanup;
	}

	read_trace_pipe();

cleanup:
	bpf_link__destroy(link);
	bpf_object__close(obj);
	return 0;
}
```

修改 `samples/bpf/Makefile`

```Makefile
...
tprogs-y += xdp_sample_pkts
tprogs-y += ibumad
tprogs-y += hbm
# 增加 user programs
tprogs-y += bob
...
ibumad-objs := ibumad_user.o
hbm-objs := hbm.o $(CGROUP_HELPERS)
# 增加 objs
bob-objs := bob_user.o $(TRACE_HELPERS)
...
always-y += hbm_edt_kern.o
always-y += xdpsock_kern.o
# 增加 kernel programs
always-y += bob_kern.o
```

编译

```bash
make VMLINUX_BTF=/sys/kernel/btf/vmlinux M=samples/bpf
```


运行

```bash
# cd samples/bpf
# ./bob
libbpf: elf: skipping unrecognized data section(4) .rodata.str1.1
           <...>-3622529 [001] d...1 1751426.283602: bpf_trace_printk: bob sshd !


            sshd-3622530 [001] d...1 1751426.288591: bpf_trace_printk: bob sshd !


            sshd-3622531 [001] d...1 1751430.924951: bpf_trace_printk: bob sshd !


           <...>-3622532 [000] d...1 1751430.929273: bpf_trace_printk: bob sshd !


            bash-3622532 [000] d...1 1751430.930114: bpf_trace_printk: bob bash !


           <...>-3622533 [001] d...1 1751430.934470: bpf_trace_printk: bob sshd !
```


# 参考&鸣谢

* [eBPF开发环境搭建](https://blog.csdn.net/lizhijun_buaa/article/details/131644323)
* [如何在 Ubuntu 上配置 eBPF 开发环境](https://yaoyao.io/posts/how-to-setup-ebpf-env-on-ubuntu#linux-tools)
* [基于ubuntu22.04-深入浅出 eBPF](https://blog.csdn.net/baidu_29900103/article/details/131188347)

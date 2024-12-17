Version: 5.4.119
# 初始化

初始化动作发生在 `module_init(init_nfsd)`的 `init_nfsd`函数中。

```C
  static int __init init_nfsd(void)
  {
      int retval;
      printk(KERN_INFO "Installing knfsd (copyright (C) 1996 okir@monad.swb.de).\n");
      retval = register_cld_notifier();
      retval = nfsd4_init_slabs();
      retval = nfsd4_init_pnfs();
      nfsd_fault_inject_init(); /* nfsd fault injection controls */
      nfsd_stat_init();   /* Statistics */
      retval = nfsd_drc_slab_create();
      nfsd_lockd_init();  /* lockd->nfsd callbacks */
      retval = create_proc_exports_entry();
      retval = register_filesystem(&nfsd_fs_type);
      retval = register_pernet_subsys(&nfsd_net_ops);
      return 0;
  }
```

## `register_cld_notifier()`

	调用`rpc_pipefs_notifier_register(&nfsd4_cld_block)`注册 RPC（远程过程调用）管道文件系统的通知回调函数。这个函数的作用是向 RPC 管道文件系统注册一个通知回调函数，以便在管道文件系统中的特定事件发生时得到通知并执行相应的处理操作。
```C
static struct notifier_block nfsd4_cld_block = {
    .notifier_call = rpc_pipefs_event,
};
```
	RPC 管道文件系统（rpc_pipefs）是 Linux 内核中用于支持 RPC 机制的一种文件系统，它允许进程之间通过管道进行通信。
```C
  rpc_pipefs_event(struct notifier_block *nb, unsigned long event, void *ptr)
  {
      struct super_block *sb = ptr;
      struct net *net = sb->s_fs_info;
      struct nfsd_net *nn = net_generic(net, nfsd_net_id);
      struct cld_net *cn = nn->cld_net;
      struct dentry *dentry;
      int ret = 0;

      if (!try_module_get(THIS_MODULE))
          return 0;

      if (!cn) {
          module_put(THIS_MODULE);
          return 0;
      }

      switch (event) {
      case RPC_PIPEFS_MOUNT:
          dentry = nfsd4_cld_register_sb(sb, cn->cn_pipe);
          if (IS_ERR(dentry)) {
              ret = PTR_ERR(dentry);
              break;
          }
          cn->cn_pipe->dentry = dentry;
          break;
      case RPC_PIPEFS_UMOUNT:
          if (cn->cn_pipe->dentry)
              nfsd4_cld_unregister_sb(cn->cn_pipe);
          break;
      default:
          ret = -ENOTSUPP;
          break;
      }
      module_put(THIS_MODULE);
      return ret;
  }
```

1.  **RPC_PIPEFS_MOUNT**:
    - `RPC_PIPEFS_MOUNT` 是指 RPC 管道文件系统成功挂载的事件。
    - 当 RPC 管道文件系统被挂载到系统中时，会触发 `RPC_PIPEFS_MOUNT` 事件。
    - 挂载 RPC 管道文件系统通常发生在系统启动时或者在运行时通过 `mount` 命令手动挂载。
2. **RPC_PIPEFS_UMOUNT**:
    - `RPC_PIPEFS_UMOUNT` 是指 RPC 管道文件系统成功卸载的事件。
    - 当 RPC 管道文件系统被从系统中卸载时，会触发 `RPC_PIPEFS_UMOUNT` 事件。
    - 卸载 RPC 管道文件系统通常发生在系统关闭或者通过 `umount` 命令手动卸载。
    
## `nfsd4_init_slabs()`

	用于初始化 NFSv4 服务器中用于分配内存的 Slab 内存池。内存池可以提高内存分配的效率和性能，避免频繁地进行内存分配和释放，从而提高系统的响应速度和稳定性。

*Slab 内存分配器（Slab Allocator）是一种高效的内存分配机制，它通过预先分配一定大小的内存块（Slab），并将其缓存起来，当需要分配内存时，可以直接从这些缓存中分配，而不必每次都向操作系统请求内存。这样可以避免频繁的内存碎片和提高内存分配的速度。*

```C
client_slab = kmem_cache_create("nfsd4_clients",sizeof(struct nfs4_client), 0, 0, NULL);
openowner_slab = kmem_cache_create("nfsd4_openowners",sizeof(struct nfs4_openowner), 0, 0, NULL);
lockowner_slab = kmem_cache_create("nfsd4_lockowners",sizeof(struct nfs4_lockowner), 0, 0, NULL);
file_slab = kmem_cache_create("nfsd4_files",sizeof(struct nfs4_file), 0, 0, NULL);
stateid_slab = kmem_cache_create("nfsd4_stateids",sizeof(struct nfs4_ol_stateid), 0, 0, NULL);
deleg_slab = kmem_cache_create("nfsd4_delegations",sizeof(struct nfs4_delegation), 0, 0, NULL);
odstate_slab = kmem_cache_create("nfsd4_odstate",sizeof(struct nfs4_clnt_odstate), 0, 0, NULL);
```

Todo...（说明每种内存池的作用）
## `nfsd4_init_pnfs()`
Todo...
## `nfsd_fault_inject_init()`

	用于初始化NFSv4 服务器中的故障注入功能。使得 NFSv4 服务器能够在运行时模拟和处理各种故障情况。这些故障可以包括但不限于：

- 网络异常或丢包
- 存储系统故障
- 客户端请求超时或错误处理
- 内存分配失败等

*Linux 内核中，故障注入是一种调试和测试技术，用于模拟系统中的各种错误和异常情况，以验证系统的稳定性和可靠性。故障注入功能允许开发者向系统中注入特定类型的错误或故障，观察系统的行为和处理方式，以便更好地理解和调试系统。*

```C
  void nfsd_fault_inject_init(void)
  {
      unsigned int i;
      struct nfsd_fault_inject_op *op;
      umode_t mode = S_IFREG | S_IRUSR | S_IWUSR;

      debug_dir = debugfs_create_dir("nfsd", NULL);

      for (i = 0; i < ARRAY_SIZE(inject_ops); i++) {
          op = &inject_ops[i];
          debugfs_create_file(op->file, mode, debug_dir, op, &fops_nfsd);
      }
  }
```

Todo...(如何模拟，如何观察？)

## `nfsd_stat_init()`

	用于初始化 NFSv4 服务器中的统计信息功能。NFSv4 服务器（nfsd）会跟踪和记录各种与 NFS 协议相关的统计信息，以便管理员和开发者了解服务器的运行状况、性能表现和客户端请求情况。这些统计信息可以包括但不限于：

- 请求处理数量（例如读取、写入、打开、关闭等）
- 请求成功和失败的数量
- 客户端连接数量和活跃情况
- 文件系统操作的性能指标

	`nfsd_stat_init()`会调用`svc_proc_register(&init_net, &nfsd_svcstats, &nfsd_proc_fops);`进行proc文件系统注册。

```C
static const struct file_operations nfsd_proc_fops = {
      .open = nfsd_proc_open,
      .read  = seq_read,
      .llseek = seq_lseek,
      .release = single_release,
  };
```

Todo...(上述这些fops，什么情况会被调用)
## `nfsd_drc_slab_create()`

	用于在 NFSv4 服务器中创建用于存储请求缓存（DRC，Duplicate Request Cache）的 Slab 内存池。请求缓存（DRC）用于缓存和重用客户端发送的请求，以提高性能并减少重复工作。例如，如果客户端发送了一个读取文件的请求，而服务器已经处理过类似的请求并得到相同的结果，则可以直接从请求缓存中返回结果，而无需再次执行读取操作。

```C
  int nfsd_drc_slab_create(void)
  {
      drc_slab = kmem_cache_create("nfsd_drc",
                  sizeof(struct svc_cacherep), 0, 0, NULL);
      return drc_slab ? 0: -ENOMEM;
  }
```

Todo...(具体工作流程)

## `nfsd_lockd_init()`

	用于在 NFSv4 服务器中初始化 NFS 文件锁管理器（NFS Lock Manager，简称NLM）所需的资源和数据结构。
	
具体来说，NFSv4 服务器中的 `nfsd_lockd_init()` 函数主要完成以下工作：
1. 初始化与 NFS 文件锁管理器相关的数据结构和状态变量。
2. 注册 NFS 文件锁管理器（NLM）的回调函数，用于处理和响应来自客户端的锁请求和释放操作。
3. 初始化必要的内存池或缓存，用于存储和管理文件锁信息。
4. 启动或准备与 NFS 文件锁服务相关的其他子系统或模块。

```C
  static const struct nlmsvc_binding nfsd_nlm_ops = {
      .fopen      = nlm_fopen,        /* open file for locking */
      .fclose     = nlm_fclose,       /* close file */
  };
  
  void nfsd_lockd_init(void)
  {
      dprintk("nfsd: initializing lockd\n");
      nlmsvc_ops = &nfsd_nlm_ops;
  }
```

Todo...(关于`nlmsvc_ops`的具体使用场景 )
## `create_proc_exports_entry()`

	用于创建和注册 NFSv4 服务器中的 `/proc/fs/nfsd/exports` 文件系统条目的函数。`/proc/fs/nfsd/exports` 是 NFSv4 服务器（nfsd）中的一个虚拟文件（Virtual File），用于显示当前服务器上的 NFS 导出信息。NFS 导出指的是将本地文件系统的目录或文件通过 NFS 协议分享给客户端的操作。
	一旦 `/proc/fs/nfsd/exports` 文件系统条目被创建和注册，用户可以通过读取这个文件来获取 NFSv4 服务器当前的导出信息，包括导出的路径、权限设置、客户端访问控制等。将 NFSv4 服务器的导出信息暴露给用户空间，方便管理员和开发者查看和管理 NFS 导出配置。

```C
 #ifdef CONFIG_PROC_FS
  static int create_proc_exports_entry(void)
  {
      struct proc_dir_entry *entry;

      entry = proc_mkdir("fs/nfs", NULL);
      if (!entry)
          return -ENOMEM;
      entry = proc_create("exports", 0, entry,
                   &exports_proc_operations);
      if (!entry) {
          remove_proc_entry("fs/nfs", NULL);
          return -ENOMEM;
      }
      return 0;
  }
  #else /* CONFIG_PROC_FS */
  static int create_proc_exports_entry(void)
  {
      return 0;
  }
  #endif
```
*需要内核编译时开启`CONFIG_PROC_FS`*

Todo...
## `register_filesystem(&nfsd_fs_type)`

	用于向 Linux 内核注册一个文件系统类型，使得该文件系统可以被内核识别和挂载。
	
```C
  int register_filesystem(struct file_system_type * fs)
  {
      int res = 0;
      struct file_system_type ** p;

      if (fs->parameters && !fs_validate_description(fs->parameters))
          return -EINVAL;
      BUG_ON(strchr(fs->name, '.'));
      if (fs->next)
          return -EBUSY;
      write_lock(&file_systems_lock);
      p = find_filesystem(fs->name, strlen(fs->name));
      if (*p)
          res = -EBUSY;
      else
          *p = fs;
      write_unlock(&file_systems_lock);
      return res;
  }
```

Todo...(后续详细介绍linux文件系统注册机制)
## `register_pernet_subsys(&nfsd_net_ops)`

	用于向 Linux 内核注册网络命名空间的子系统（Subsystem）。

*在 Linux 内核中，网络命名空间（Network Namespace）是一种机制，允许多个独立的网络栈共存于同一个内核实例中。每个网络命名空间都有自己独立的网络设备、路由表、防火墙规则等网络资源，从而实现网络隔离和虚拟化。*

```C
 static struct pernet_operations nfsd_net_ops = {
      .init = nfsd_init_net,
      .exit = nfsd_exit_net,
      .id   = &nfsd_net_id,
      .size = sizeof(struct nfsd_net),
 };
```

Todo... (nfsd_net_ops 中的函数，何时被回调，id合适被修改？)
# super


![[nfs-super.drawio.png]]

* mount -> syscall -> vfs_kern_mount -> fc_mount -> vfs_get_tree -> nfsd_fs_context_ops.get_tree -> nfsd_fill_supper 
* nfsd_fs_type 注册到 file_systems 中
* vfs_kern_mount -> fs_context_for_mount -> alloc_fs_context 负责动态分配`fs_context`资源，并初始化 其中大部分资源。
* vfs_kern_mount -> `vfs_parse_fs_string(fc, "source", name, strlen(name));`初始化`fs_context` 中的`const char  *source;` 资源
* nfsd_init_fs_context 初始化 `fs_context` 中的 `const struct fs_context_operations *ops` 和`struct user_namespace   *user_ns;`

Version: 5.4.119
# SunRPC

## xprt queue

![[nfs-sunrpc框架.drawio.png]]

* add svc_xprt to sp_sockets by svc_xprt_enqueue  and weak up process
* get rq_xprt from sp_sockets by svc_xprt_dequeue

## RPC Protocol

![[nfs-RPC_Header.drawio.png]]

## Task execute flow

![[nfs-sunrpc_task_execute_flow.drawio.png]]

# [NFSD](obsidian://open?vault=ANewOne&file=FileSystems%2FNFS%2FNFSD)

# NFS Client

## Ops

![[nfs-client.drawio.png]]
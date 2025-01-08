From : https://docs.ipfs.tech/reference/kubo/rpc/#rpc-commands

# 标准RPC命令

这是主要的稳定API，包含以下几个主要功能组

** 文件操作**

	- /api/v0/add - 添加文件到IPFS
	- /api/v0/cat - 显示IPFS对象数据
	- /api/v0/get - 下载IPFS对象
	- /api/v0/files/* - MFS(可变文件系统)相关操作

**区块操作**

	- /api/v0/block/* - 处理原始IPFS块的操作
	- /api/v0/dag/* - 处理IPLD DAG对象的操作

**网络操作**

	- /api/v0/swarm/* - 管理p2p网络连接
	- /api/v0/bootstrap/* - 管理启动节点
	- /api/v0/bitswap/* - 管理数据交换

**名称和密钥管理**

	- /api/v0/name/* - IPNS名称操作
	- /api/v0/key/* - 密钥管理

**Pin操作**

	- /api/v0/pin/* - 管理固定的对象

# 实验性RPC命令

这些是正在开发中的新功能，API可能会改变：

	- /api/v0/files/chmod - 更改权限
	- /api/v0/key/sign - 签名操作
	- /api/v0/p2p/* - P2P流操作
	- /api/v0/routing/* - 路由相关操作

# 已弃用RPC命令

这些命令将在未来版本中移除：

	- /api/v0/dht/query
	- /api/v0/object/* - 旧的对象操作
	- /api/v0/pubsub/* - 旧的发布订阅操作

# 已移除RPC命令

这些是已经被移除的命令，仅作为文档参考：

	- /api/v0/dht/* - 已被routing命令替代
	- /api/v0/object/* - 已被dag命令替代

# 总结

	- 所有命令都是基于HTTP的RPC风格API，不是REST API
	- 命令路径都以/api/v0/开头
	- 支持查询参数来传递参数
	- 部分命令支持文件上传
	- 默认绑定到localhost:5001
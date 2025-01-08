# kubo in docker

1. 拉取kubo镜像
	使用`docker pull ipfs/kubo` 拉取[kubo镜像](https://hub.docker.com/r/ipfs/kubo/)
2. 启动kubo镜像
	
	```shell
	docker run -d --name ipfs_host -v $ipfs_staging:/export -v $ipfs_data:/data/ipfs -p 4001:4001 -p 4001:4001/udp -p 127.0.0.1:8080:8080 -p 127.0.0.1:5001:5001 ipfs/kubo:latest
	```
	启动容器，导出端口`4001`(P2P TCP/QUIC transports),、`8080` (Gateway)、`5001`(RPC API)

		
	```shell
	export ipfs_staging=</absolute/path/to/somewhere/>
	export ipfs_data=</absolute/path/to/somewhere_else/>
	```
	
	要使文件在容器中可见，您需要将带有-v选项的主机目录挂载到Docker。选择一个您想要用于从IPFS导入和导出文件的目录。您还应该选择一个目录来存储IPFS文件，该文件在重新启动容器时会持续存在。	

3. 等待kubo容器启动完成

	```shell
	$ docker logs -f ipfs_host
	Changing user to ipfs
	ipfs version 0.32.1
	generating ED25519 keypair...done
	peer identity: 12D3KooWQ761xsS5HYLdWPrusChbmnpFh2Dd6MU8biFSpuNDF64Q
	initializing IPFS node at /data/ipfs
	Initializing daemon...
	Kubo version: 0.32.1-9017453
	Repo version: 16
	System version: arm64/linux
	Golang version: go1.23.3
	PeerID: 12D3KooWQ761xsS5HYLdWPrusChbmnpFh2Dd6MU8biFSpuNDF64Q
	2024/12/17 08:05:12 failed to sufficiently increase receive buffer size (was: 208 kiB, wanted: 7168 kiB, got: 416 kiB). See https://github.com/quic-go/quic-go/wiki/UDP-Buffer-Sizes for details.
	Swarm listening on 127.0.0.1:4001 (TCP+UDP)
	Swarm listening on 172.17.0.8:4001 (TCP+UDP)
	Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.
	RPC API server listening on /ip4/0.0.0.0/tcp/5001
	WebUI: http://127.0.0.1:5001/webui
	Gateway server listening on /ip4/0.0.0.0/tcp/8080
	Daemon is ready
	```

详细请见：https://docs.ipfs.tech/install/run-ipfs-inside-docker/#set-up
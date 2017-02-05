# Docker安装
## 增加repo
	[dockerrepo]
	name=DockerRepository	baseurl=https://yum.dockerproject.org/repo/main/centos/7/	enabled=1	gpgcheck=1	gpgkey=https://yum.dockerproject.org/gpg##搜索
	yum search docker
	yum install docker-engine
	docker build -t ambari:centos6 .
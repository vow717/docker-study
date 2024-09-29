# 官网安装virtualbox
## 下载对应的CentOS 7 
https://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/
下载CentOS-7-x86_64-DVD-2207-02.iso
在虚拟机的光驱，引入下载的CentOS镜像
(右Ctrl可以切换鼠标键盘的控制)
## VB网络地址转换(NAT)模式端口映射，例如将宿主机10022端口映射到虚拟机22端口；
 安装步骤可以参照：
 https://blog.csdn.net/2302_78886445/article/details/141327875?ops_request_misc=%257B%2522request%255Fid%2522%253A%25221F676A04-1F85-4A6D-97C5-2EC5D1D04FB4%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=1F676A04-1F85-4A6D-97C5-2EC5D1D04FB4&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-141327875-null-null.142^v100^pc_search_result_base3&utm_term=virtualbox%E5%AE%89%E8%A3%85centos%E8%99%9A%E6%8B%9F%E6%9C%BA&spm=1018.2226.3001.4187

 ## centos换yum源
 1.cd .. 退回到ls能看到etc的页面
 2.cd /etc/yum.repos.d
 3.sudo curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
 4.yum clean all
 5.yum makecache
 6.yum update -y

 ## 安装epel存储库
 sudo yum install epel-release
 ## 安装docker
 sudo yum install docker
 ## 启动docker服务
 sudo systemctl start docker
 ## 设置开机启动
 sudo systemctl enable docker
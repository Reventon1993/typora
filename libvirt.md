libvirt

libvirt.conf指定默认的虚拟化后端(uri)

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211124174619.png" alt="企业微信截图_1edf94c3-7dcd-4279-8a66-18c7068449f4" style="zoom:25%;" />

没有指定uri_default时候，libvirt也会有默认的uri，qemu优先级最高？



virsh -c 指定使用的后端，如果不指定，使用默认后端。

virsh uri获取默认的后端

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211124174614.png" alt="企业微信截图_8567660d-6b3a-4288-92c2-34620108d1df" style="zoom:33%;" />

后端的相关文件

![企业微信截图_e941993b-4f8e-4f4b-aa6e-b8e7cd2dda45](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211124174808.png)

![企业微信截图_8d7e63ee-468f-4f5f-97ee-1823006fb1fc](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211124174847.png)

libvirt和qemu交互，并不感知kvm

qemu和kvm交互


# SAN

Storage Area Network，存储区域网络。主体是网络，表示一种提供**块存储**能力的网络方案。

早期的SAN采用的是[光纤通道](https://baike.baidu.com/item/光纤通道)(FC，Fibre Channel)技术，到了iSCSI协议出现以后，为了区分，业界就把SAN分为FC-SAN和IP-SAN

IPSAN使用以太网，应用层协议为iscsi。FCSAN使用FC网络，”应用层”协议为FC-3\FC-4协议。 

SAN提供的块存储被称为lun(Logical Unit Number)

# FCSAN

## FC网络

![image-20220218173520559](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173526.png)

### FC交换机

FC组网需要光纤交换机。

### HBA卡

![image-20220218173547569](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173547.png)

HBA卡一般是PCI设备

![image-20220218173604213](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173604.png)

获取节点上的hba信息：

*systool -c fc_host -v*

![image-20220218173802460](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173802.png)

### systool

systool是一个通用工具

![image-20220218173913028](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173913.png)

systool输出hba卡的信息都来源sys文件系统

比如hba的port_state信息:

![image-20220218173918513](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218173918.png)

### wwpn&wwnn

FC两端使用WWPN/WWNN通信

https://cloud.tencent.com/developer/article/1455080



通常通信的双方是HBA卡和lun，HBA卡自带wwpn/wwnn，类似mac地址。

lun也有wwpn/wwnn，应该是生成的。

## FCSAN使用流程

### FCSAN存储设备端

创建lun并创建和initiator(hba卡)的关系。

以华为OceanStor为例

A.创建lun，创建的lun属于一个lun组

B.使用initiator(hba卡)的wwpn/iqn号创建主机，主机属于一个主机组。

C.创建映射视图，表示lun组和主机组的”关联关系”。此时，会生成target(lun)的wwpn用于后续的通信。 

**LUN：**

![image-20220218174116880](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218174116.png)

做了C步骤才会变成已映射状态

**主机：**

![image-20220218174132290](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218174132.png)

![image-20220218174149369](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218174149.png)

创建映射后，会生成target的wwpn。有几个target就会在initiator端发现多少个设备(多路径)

**映射：**

![image-20220218174206401](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218174206.png)

**三者关系：**

![image-20220218174213118](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218174213.png)

### 主机端

通过主机上的hba卡发现块设备，映射为本地一个块设备(比如sdb)

发现命令:

echo '- - -' > /sys/class/scsi_host/{hba设备}/scan

![image-20220218174227797](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220218180521.png)

如果存在多路径，可以使用multipath服务，multipath.conf配置和具体的存储设备有关。

## cinder对接OceanStor

cinder把OceanStor设备做为一种提供块存储能力的driver。

### 创建删除fc云盘

创建最主要的流程就是完成上述fcsan存储设备端的A操作。流程就是cinder调用OceanStor提供的api，这部分已经被封装成cinder的驱动代码(华为提供)。



### 挂载fc云盘

挂载最主要的流程就是上述fcsan存储设备端的B、C操作和主机端的操作。和其他类型云盘的挂载一样，整个操作需要nova、cinder共同完成,具体过程如下:

1.nova-compute使用systool命令获取hba卡的wwpn/wwnn调用cinder的initailize_connction接口分给cinder。

2.cinder接到请求后使用驱动代码完成B、C操作，返回包含lun wwn的连接信息。

3.nova使用连接信息完成客户端的操作

```python
# nova/compute/manager.py
def attach_volume(self, context, volume_id, mountpoint,
                  instance, bdm=None):
    ....
  
    self._attach_volume(context, instance, driver_bdm)
  
    ....
  

def _attach_volume(self, context, instance, bdm):
    ....
  
    bdm.attach(context, instance, self.volume_api, self.driver,
               do_check_attach=False, do_driver_attach=True)
  
    ....
  
  
# nova/virt/block_device.py

class DriverVolumeBlockDevice(DriverBlockDevice):
    ....
  
    def attach(self, context, instance, volume_api, virt_driver,
               do_check_attach=True, do_driver_attach=False):
        ....
        # datastruct1
        connector = virt_driver.get_volume_connector(instance)
        # datastruct2
        connection_info = volume_api.initialize_connection(context,
                                                         volume_id,
                                                         connector)
        ....
    
        virt_driver.attach_volume(context, connection_info, instance,
                          self['mount_device'], disk_bus=self['disk_bus'],
                          device_type=self['device_type'], encryption=encryption)
      
        ....
      
      ....
  
# nova/virt/libvirt/driver.py

def attach_volume(....):
    ....
    self._attach_volume(....)
    ....
  
def _attach_volume(....):
    ....
    self._connect_volume(....)
    ....

def _connect_volume(....)
    ....
    driver = self._get_volume_driver(connection_info)
    driver.connect_volume(connection_info, disk_info)
    ....
  
# nova/virt/libvirt/volume.py
class LibvirtFibreChannelVolumeDriver(LibvirtBaseVolumeDriver):
    ....
	  def connect_volume(self, connection_info, disk_info):
        fc_properties = connection_info['data']
        possible_devs = self._get_possible_devices(fc_properties['target_wwn'])
        # datastruct6
        host_devices = []
        for device in possible_devs:
            pci_num, target_wwn = device
            ....
            host_device = ("/dev/disk/by-path/pci-%s-fc-%s-lun-%s" %
                       (pci_num, target_wwn, fc_properties.get('target_lun', 0)))
    
        linuxscsi.rescan_hosts(libvirt_utils.get_fc_hbas_info())
      
        for device in host_devices:
            if os.path.exists(device):
                self.host_device = device
                self.device_name = os.path.realpath(device)
                break
          
         mdev_info = linuxscsi.find_multipath_device(self.device_name)
         # datastruct3
         if mdev_info is not None:
             device_path = mdev_info['device']
             connection_info['data']['device_path'] = device_path
             connection_info['data']['devices'] = mdev_info['devices']
             connection_info['data']['multipath_id'] = mdev_info['id']
         else:
              device_path = self.host_device
              device_info = linuxscsi.get_device_info(self.device_name)
              # datastruct4、5
              connection_info['data']['device_path'] = device_path
              connection_info['data']['devices'] = [device_info]
      ....
  ....

connector
{'ip': '10.175.203.12', 'host': 'cmp-203-12', 'wwnns': ['2000f4c7aa03e3f3', '2000f4c7aa03dfa1'], 'initiator': 'iqn.1994-05.com.redhat:70b0297b29f2', 'wwpns': ['2100f4c7aa03e3f3', '2100f4c7aa03dfa1']} 
注意：initiator是用于iscsi流程的，fc流程中该数据没用

connection_info
{u'driver_volume_type': u'fibre_channel', u'data': {u'initiator_target_map': {u'2100f4c7aa03dfa1': [u'2002d4d51b13902e', u'2003d4d51b13902e', u'2012d4d51b13902e', u'2013d4d51b13902e'], u'2100f4c7aa03e3f3': [u'2000d4d51b13902e', u'2001d4d51b13902e', u'2010d4d51b13902e', u'2011d4d51b13902e']}, u'target_discovered': True, u'qos_specs': {}, u'volume_id': u'59a0b72b-1958-4596-bf45-a4aca1fb69bc', u'target_lun': 3, u'access_mode': u'rw', u'target_wwn': [u'2000d4d51b13902e', u'2001d4d51b13902e', u'2010d4d51b13902e', u'2011d4d51b13902e', u'2002d4d51b13902e', u'2003d4d51b13902e', u'2012d4d51b13902e', u'2013d4d51b13902e'], u'map_info': {u'lun_id': u'18', u'view_id': u'1', u'aval_luns': u'[0,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,289,290,291,292,293,294,295,296,297,298,299,300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315,316,317,318,319,320,321,322,323,324,325,326,327,328,329,330,331,332,333,334,335,336,337,338,339,340,341,342,343,344,345,346,347,348,349,350,351,352,353,354,355,356,357,358,359,360,361,362,363,364,365,366,367,368,369,370,371,372,373,374,375,376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,436,437,438,439,440,441,442,443,444,445,446,447,448,449,450,451,452,453,454,455,456,457,458,459,460,461,462,463,464,465,466,467,468,469,470,471,472,473,474,475,476,477,478,479,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,495,496,497,498,499,500,501,502,503,504,505,506,507,508,509,510,511]'}}}

mdev_info
{'device': '/dev/mapper/36d4d51b10013902e2807f79700000016', 'id': '36d4d51b10013902e2807f79700000016', 'devices': [{'device': '/dev/sdd', 'host': '16', 'id': '2', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdh', 'host': '18', 'id': '2', 'channel': '0', 'lun': '1'}, {'device': '/dev/sde', 'host': '16', 'id': '3', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdi', 'host': '18', 'id': '3', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdb', 'host': '16', 'id': '0', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdf', 'host': '18', 'id': '0', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdc', 'host': '16', 'id': '1', 'channel': '0', 'lun': '1'}, {'device': '/dev/sdg', 'host': '18', 'id': '1', 'channel': '0', 'lun': '1'}], 'name': '36d4d51b10013902e2807f79700000016'}

device_path
'/dev/disk/by-path/pci-0000:31:00.1-fc-0x2000d4d51b13902e-lun-1'

device_info
{'device': '/dev/sdb', 'host': '16', 'id': '0', 'channel': '0', 'lun': '1'}

possible_devs
[('0000:31:00.0', '0x2000d4d51b13902e'), ('0000:31:00.0', '0x2001d4d51b13902e'), ('0000:31:00.0', '0x2010d4d51b13902e'), ('0000:31:00.0', '0x2011d4d51b13902e'), ('0000:31:00.0', '0x2002d4d51b13902e'), ('0000:31:00.0', '0x2003d4d51b13902e'), ('0000:31:00.0', '0x2012d4d51b13902e'), ('0000:31:00.0', '0x2013d4d51b13902e'), ('0000:31:00.1', '0x2000d4d51b13902e'), ('0000:31:00.1', '0x2001d4d51b13902e'), ('0000:31:00.1', '0x2010d4d51b13902e'), ('0000:31:00.1', '0x2011d4d51b13902e'), ('0000:31:00.1', '0x2002d4d51b13902e'), ('0000:31:00.1', '0x2003d4d51b13902e'), ('0000:31:00.1', '0x2012d4d51b13902e'), ('0000:31:00.1', '0x2013d4d51b13902e'), ('0000:4b:00.0', '0x2000d4d51b13902e'), ('0000:4b:00.0', '0x2001d4d51b13902e'), ('0000:4b:00.0', '0x2010d4d51b13902e'), ('0000:4b:00.0', '0x2011d4d51b13902e'), ('0000:4b:00.0', '0x2002d4d51b13902e'), ('0000:4b:00.0', '0x2003d4d51b13902e'), ('0000:4b:00.0', '0x2012d4d51b13902e'), ('0000:4b:00.0', '0x2013d4d51b13902e'), ('0000:4b:00.1', '0x2000d4d51b13902e'), ('0000:4b:00.1', '0x2001d4d51b13902e'), ('0000:4b:00.1', '0x2010d4d51b13902e'), ('0000:4b:00.1', '0x2011d4d51b13902e'), ('0000:4b:00.1', '0x2002d4d51b13902e'), ('0000:4b:00.1', '0x2003d4d51b13902e'), ('0000:4b:00.1', '0x2012d4d51b13902e'), ('0000:4b:00.1', '0x2013d4d51b13902e')]
注意：
就是hba卡pci地址*target wwpn的组合。
# lspci -nnn |grep -i fib
31:00.0 Fibre Channel [0c04]: QLogic Corp. Device [1077:2261] (rev 01)
31:00.1 Fibre Channel [0c04]: QLogic Corp. Device [1077:2261] (rev 01)
4b:00.0 Fibre Channel [0c04]: QLogic Corp. Device [1077:2261] (rev 01)
4b:00.1 Fibre Channel [0c04]: QLogic Corp. Device [1077:2261] (rev 01)

ls /dev/disk/by-path/ |grep lun
pci-0000:31:00.1-fc-0x2000d4d51b13902e-lun-1
pci-0000:31:00.1-fc-0x2001d4d51b13902e-lun-1
pci-0000:31:00.1-fc-0x2010d4d51b13902e-lun-1
pci-0000:31:00.1-fc-0x2011d4d51b13902e-lun-1
pci-0000:4b:00.1-fc-0x2002d4d51b13902e-lun-1
pci-0000:4b:00.1-fc-0x2003d4d51b13902e-lun-1
pci-0000:4b:00.1-fc-0x2012d4d51b13902e-lun-1
pci-0000:4b:00.1-fc-0x2013d4d51b13902e-lun-1
```



    挂载流程中使用的命令：
    使用systool获取hba卡的pci地址、scsi_host地址
    使用echo '- - -' > /sys/class/scsi_host/host16/scan 做映射
    使用pci*target组合可以得知映射后的可能设备路径，实际的设备路径为其子集
    如果不使用多路径，那么直接使用其中一个路径
    如果使用多路径：
    使用multipath -l 获取多路径设备的路径
    流程中还有用sgcan获取映射设备的scsi地址：
    # sg_scan /dev/sdb
    /dev/sdb: scsi16 channel=0 id=0 lun=1 [em]

### 卸载fc云盘

卸载流程主要流程如下：

1.nova-compute删除本地映射(包括多路径)，调用cinder的terminate_connection接口。

2.cinder接受调用后，调用驱动，删除华为段的映射和主机

```python
# nova/compute/manager.py
def detach_volume(self, context, volume_id, instance):
    ....
	  self._detach_volume(context, volume_id, instance)
    ....
    
def _detach_volume(self, context, volume_id, instance, destroy_bdm=True):
    ....
    # detach
    self._driver_detach_volume(context, instance, bdm)
    # terminate
    connector = self.driver.get_volume_connector(instance)
    self.volume_api.terminate_connection(context, volume_id, connector)
    .... 
 
def _driver_detach_volume(self, context, instance, bdm):
    ....
    self.driver.detach_volume(....)
    ....
    

# nova/virt/libvirt/driver.py
def detach_volume(self, connection_info, instance, mountpoint, encryption=None):
    ....
    self._disconnect_volume(connection_info, disk_dev)
    ....
    
def _disconnect_volume(self, connection_info, disk_dev):
    ....
    driver = self._get_volume_driver(connection_info)
    driver.disconnect_volume(connection_info, disk_dev)
    ....
    
# nova/virt/libvirt/volume.py
class LibvirtFibreChannelVolumeDriver(LibvirtBaseVolumeDriver):
    ....
    def disconnect_volume(self, connection_info, mount_device):
        if 'multipath_id' in connection_info['data']:
            multipath_id = connection_info['data']['multipath_id']
            mdev_info = linuxscsi.find_multipath_device(multipath_id)
            devices = mdev_info['devices']
        else:
            devices = connection_info['data'].get('devices', [])
        for device in devices:
            linuxscsi.remove_device(device)
```

    卸载流程中使用的命令：
    使用如下命令:
    echo '1' >  /sys/bus/scsi/drivers/sd/18\:0\:3\:3/delete
    删除所有映射，所有映射删除后，多路径设备自动会删除。
    命令中的地址源于挂载流程中获得scsi地址



### 热扩容fc云盘

热扩容功能主要有两步：

1.cinder端的扩容操作

2.cinder扩容完成后会通知nova，nova端完成热扩容。

 

fc云盘热扩容和rbd云盘的不同在于第2步。

虚拟机对rbd的云盘是通过网络直接使用。步骤1中cinder完成了rbd的扩容，

步骤2中，nova只需要调用virsh blockresize即可。

 

fc云盘的使用流程是，先把lun映射为本地的块设备，然后libvirt透传块设备给虚拟机使用。

步骤1完成了lun的扩容。步骤2需要完成本地块设备的扩容，然后再调用virsh blockresize。

本地块设备的扩容其实就是重新进行映射。

```python
# nova/compute/manager.py
def extend_volume(self, context, instance, extended_volume_id):
	  ....
  
    self.driver.extend_volume(connection_info, instance, 
                            bdm.volume_size * pow(1024, 3), bdm.device_name)
    ....
  
# nova/compute/manager.py
class LibvirtFibreChannelVolumeDriver(LibvirtBaseVolumeDriver):
    ....
    def extend_volume(self, connection_info, instance, requested_size):
        ....
        linuxscsi.extend_volume(devices, multipath_id=multipath_id)
        ....
    ....
   
# nova/storage/linuxscsi.py
def extend_volume(devices, multipath_id=None):
    for device in devices:
        device_id = ("%(host)s:%(channel)s:%(id)s:%(lun)s" %
              {'host': device['host'],
               'channel': device['channel'],
               'id': device['id'],
               'lun': device['lun']})
        scsi_path = ("/sys/bus/scsi/drivers/sd/%(device_id)s" %
             {'device_id': device_id})

        rescan_path = "%(scsi_path)s/rescan" % {'scsi_path': scsi_path}
        echo_scsi_command(rescan_path, "1")

    if multipath_id:
        multipath_reconfigure()
        multipath_resize_map(multipath_id)

def multipath_reconfigure():
    (out, err) = utils.execute('multipathd', 'reconfigure',
                               run_as_root=True)
    return out
  
def multipath_resize_map(mpath_id):
    (out, err) = utils.execute('multipathd', 'resize', 'map', mpath_id,
                               run_as_root=True)
    return out

def echo_scsi_command(path, content):
    """Used to echo strings to scsi subsystem."""
    args = ["-a", path]
    kwargs = dict(process_input=content, run_as_root=True)
    utils.execute('tee', *args, **kwargs)
```

    扩容本地映射和多路径设备流程中使用的命令：
    使用如下命令扩容映射设备
    echo '1' > /sys/bus/scsi/drivers/sd/18\:0\:3\:3/rescan
    使用如下命令扩容多路径设备
    multipathd reconfigure
    multipathd resize map 36d4d51b10013902e2807f79700000016

# IPSAN

## iscsi

基于tcp的应用层协议

https://en.wikipedia.org/wiki/ISCSI



**IQN**

iSCSI Qualified Name (IQN)

表示initiator和target的地址，双方使用iqn通信



## iscsi.service

软件实现的initiator 

如下命令查看iqn

cat /etc/iscsi/initiatorname.iscsi 　　　　　　 

InitiatorName=iqn.1994-05.com.redhat:b53d39193da1



## iscsi的使用流程

和FCSAN的流程几乎一致。

wwpn/wwnn变为iqn

主机端的操作都有iscsiadm命令进行。
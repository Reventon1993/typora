https://amir.rachum.com/blog/2017/07/28/python-entry-points/



在python包路径下每个python包都会有个对应的egg文件夹，文件夹中包含了一些关于包"metadata"数据。其中entry_points.txt里包含了包的entry_points。

可以使用pkg_resources 或者 stevedore来使用entry_points。entry_points的来源就是一个个entry_points.txt文件。

其中console_sripts是一个特殊的section。所有entry_points.txt文件中的console_sripts都会被叠加在一起。



e.g

pkg_resources.iter_entry_points(group=group)   #group为类似'console_sripts'这样的写在entry_points.txt的section。



pkg_resources.load_entry_point('ceilometer', 'ceilometer.compute.virt', 'libvirt')  

'ceilometer'为包名，'ceilometer.compute.virt'为entry_points.txt的section，'libvirt'为section中具体的一配置项。



#TODO

stevedore?
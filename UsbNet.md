2020/9/30 11:23:13 


----------
移远模组可以通过AT指令配置网卡类型

<font color=0x00FFFF>
AT+QCFG=”usbnet”
</font>


<!-- Row Highlight Javascript -->
<script type="text/javascript">
	window.onload=function(){
	var tfrow = document.getElementById('tfhover').rows.length;
	var tbRow=[];
	for (var i=1;i<tfrow;i++) {
		tbRow[i]=document.getElementById('tfhover').rows[i];
		tbRow[i].onmouseover = function(){
		  this.style.backgroundColor = '#f3f8aa';
		};
		tbRow[i].onmouseout = function() {
		  this.style.backgroundColor = '#ffffff';
		};
	}
};
</script>

<style type="text/css">
table.tftable {font-size:12px;color:#333333;width:100%;border-width: 1px;border-color: #729ea5;border-collapse: collapse;}
table.tftable th {font-size:12px;background-color:#acc8cc;border-width: 1px;padding: 8px;border-style: solid;border-color: #729ea5;text-align:left;}
table.tftable tr {background-color:#ffffff;}
table.tftable td {font-size:12px;border-width: 1px;padding: 8px;border-style: solid;border-color: #729ea5;}
</style>

<table id="tfhover" class="tftable" border="1">
<tr><th>模块网卡类型</th><th>Linux 驱动</th><th>拨号方式</th><th>物理层数据</th></tr>
<tr><td>rmnet/qmi/ndis
at+qcfg=”usbnet”,0
</td><td>qmi_wwan&cdc_wdm
CONFIG_USB_NET_QMI_WWAN
 添加移远模块的vid和pid
内核版本 >= 3.4
；
GobiNet
 移远提供源码

</td><td>libqmi(Ubuntu)
uqmi(openWRT)
quectel-CM
AT$QCRMCALL=1,1
Autoconnect(不推荐, 不适用于5G)
</td><td>IP帧</td></tr>
<tr><td>ecm
a+qcfg=”usbnet”,1
车载方案OpenLinux
</td><td>CONFIG_USB_NET_CDCETHER</td><td>autoconnect</td><td>以太网帧</td></tr>
<tr><td>mbim
at+qcfg=”usbnet”,2
Win10平板EM/EP系列
</td><td>CONFIG_USB_NET_CDC_MBIM
内核版本 >= 3.18
</td><td>mbim协议拨号
libmbim(Ubuntu)
，quectel-CM也支持</td><td>MBIM帧</td></tr>
<tr><td>rndis
at+qcfg=”usbnet”,3
</td><td>CONFIG_USB_NET_RNDIS_HOST</td><td>autoconnect</td><td>以太网帧</td></tr>
</table>

其他如NCM和ECM、RNDIS类似。
NCM内核开启

----------

# RMNET #
### QMI ###
内核自带的
[qmi_wwan.c](https://elixir.bootlin.com/linux/v4.14.181/source/drivers/net/usb/qmi_wwan.c)
可以用于LTE模组。
但是该驱动不支持Quectel IP聚合和IP复用功能，并且不支持5G模组（5G模组强制打开IP聚合）


### GobiNet ###
如果要用AT指令拨号，将驱动中的qcrmcall_mode 设置成1. （5G模组暂不支持AT指令拨号）

### 什么是IP聚合、IP复用 ###

Quectel网卡支持IP聚合和IP复用。
IP聚合（IP aggregation protocol）指的是一次传输urb中携带多个IP数据包，相比于传统的QMI和GobiNet驱动一个urb只能传输一个IP数据包，IP聚合减少CPU中断数量和频率，降低了CPU负载。IP聚合功能对应网卡驱动qmap_mode为1或者大于1的情况。
IP复用（IP Multiplexing Protocol）指是通过USB Bus传输qmap数据包，支持多路PDN拨号的情况，对应qmap_mode 大于1。
usb网卡驱动默认没有打开IP聚合功能和使用qmap功能。推荐打开IP聚合，将qmi_wwan_q.c中的qmap_mode = 0 改成qmap_mode = 1。

<table><tr><td bgcolor=yellow>请向Quectel索要最新版本的QMI驱动和GobiNet驱动 </td></tr></table>

	Mailto:support@quectel.com

















----------

# ECM #

CDC ECM 驱动是模块适配标准的 ECM 通用驱动，无需额外的代码修改，直接配置编译项即可。 
CDC ECM 驱动的相关配置项：
	
	CONFIG_USB_USBNET=y 
	CONFIG_NETDEVICES=y 
	CONFIG_USB_NET_CDCETHER=y

----------

# MBIM #
CDC MBIM 驱动是华为模块适配标准的通用驱动，无需额外的代码修改，直接配置编
译项即可。（Kernel 3.18以后才支持）
CDC MBIM 驱动的相关配置项：
	 
	CONFIG_USB_USBNET=y 
	CONFIG_NETDEVICES=y 
	CONFIG_USB_NET_CDC_MBIM=y


----------


#RmNet和CDC-ECM区别：
RmNet获取公网IP，ECD-ECM获取局域网IP。
在高通平台上，rmnet driver 和标准的CDC-ECM是有区别的，rmnet 也是属于CDC-ECM
从模块角度出发，对USB命令的封装、使用的USB接口、端点定义方式不同；
使用rmnet,发起data call是通过QMI工具发的QMI命令，而通过标准的CDC-ECM发起data call，则是发送标准的ECM命令。
使用rmnet建立的data call，不走router的，IP地址的是公网IP。
而通过标准的CDC-ECM建立的data call，是走router的，获得的IP地址是私有的IP如192.168开头。

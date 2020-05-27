# Model Driven Programmability

*模型驱动的可编程性*

- ##### Introduction to Basic Programming with Python

  - [Python3教程](https://www.runoob.com/python3/python3-tutorial.html)

- ##### YANG, RESTCONF & NETCONF

  - Network Programmability==网络可编程性概念==

    **可编程的网络API接口**

    1. 网络控制器可编程性

       REST APIS 如*DNA-C*

    2. 设备级可编程性

       *YANG models* *NETCONF* *REATCONF*

    ***模型驱动的可编程性*主要用来干啥什么**

    1. 设备级可编程性

       配置一个新的设备，或升级新设备的配置

       事务——要么所有设备更改成功，要么整个事务回滚

    2. 清晰定义的网络设备和服务模型

       模型定义设备及其配置的功能和特性

       模型定义操作和统计数据

       模型定义模式——什么数据可用，什么参数，数据类型(字符串，数字，布尔值，…)，等等。

    

  - Programming Network Devices

    > Device Level APIs and Programmability
    >
    > Management over CLI (Telnet, SSH)

    详细见[总结](#总结)

    

  - YANG device models

    为托管设备和与设备通信的另一个系统之间交换的消息的内容定义确切的结构、数据类型、语法和验证规则。数据模型仅仅是一种被广泛理解和认可的描述“某物”的方法。举个简单的例子 接口的“数据模型”:

    ```
    interface 
    name [String, max 32 chars] 
    enabled [Boolean] 
    description [String, max 64 chars, optional] 
    IPv4 Address [Dotted Decimal] 
    IPv4 Netmask [Integer, prefix length] …
    ```

    • 模块，它是一个自包含的顶级节点层次结构

    • 使用容器对相关节点进行分组

    • 使用列表来标识按顺序存储的节点

    • 节点的每个单独属性由一个叶子表示每个叶子必须有一个相关联的类型

    ```json
    module ietf-interfaces { 
        import ietf-yang-types { 
       		prefix yang;
    	}
    	container interfaces { 
        	list interface { 
            	key "name"; 
            	leaf name {
            		type string;
        		}
    			leaf enabled { 
                    type boolean; default "true";
                }
    }
    ```

    ![image-20200527204833988](README.assets\image-20200527204833988.png)

    

  - ##### Experimenting with RESTCONF and NETCONF protocols

    ![image-20200527205117912](README.assets\image-20200527205117912.png)

#### 1.Network Virtualizaton 网络虚拟化

1. 介绍
2. 云主机
3. 虚拟化
4. 虚拟网络基础设施
5. 软件定义网络
6. 控制器
7. 练习

#### 2.Network Automation 网络自动化

1. 介绍
2. 自动化概述
3. 数据格式
4. apis
5. rest
6. 配置管理工具
7. IBN 和 DNAC
8. 练习



# 总结

##### 网络可编程性与管理

##### 1. 使用SSH，TELNEL配置管理 

- 使用SSH，TElNET客户端登录到路由进行配置管理

  ![image-20200527203100488](README.assets\image-20200527203100488.png)

- 使用netmiko包进行配置管理

  ```python
  from netmiko import ConnectHandler
  # create a variable object that represents the ssh cli session
  sshCli = ConnectHandler(
          device_type='cisco_ios', 
          host='192.168.1.5',
          port=22,
          username='root',
          password='root'
          )
  print("Sending 'sh ip int brief'.")
  output = sshCli.send_command("show ip int brief")
  ```

  

##### 2.使用NETCONF配置管理

1. 使用**ncclient**包 端口:830

```python
from ncclient import manager

# create a variable object that represents the NETCONF session
m = manager.connect(
         host="192.168.1.5",
         port=830,
         username="root",
         password="root",
         hostkey_verify=False
         )

# one connected to the remote device using NETCONF
# display the device capabilities (supported YANG models)
print("#Supported Capabilities (YANG models):")
for capability in m.server_capabilities:
    print(capability)
```

2. 使用.get_config(source="running", filter=netconf_filter)获取配置

```python
from ncclient import manager
import xml.dom.minidom

# create a variable object that represents the NETCONF session
m = manager.connect(
         host="192.168.56.101",
         port=830,
         username="cisco",
         password="cisco123!",
         hostkey_verify=False
         )

# use the "get_config" method of the NETCONF object
# to get back the current "source" configuration
netconf_reply = m.get_config(source="running")
print(netconf_reply)



input("###### Press Enter to continue to Step 3 ######")
print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())


input("###### Press Enter to continue to Step 4 ######")
# limit the output from the NETCONF reply using a NETCONF filter
# in this case, filtering only data defined in 
# the native cisco ios xe yang model
netconf_filter = """
<filter>
    <routing xmlns="urn:ietf:params:xml:ns:yang:ietf-routing" />
</filter>
"""

# use the "get_config" method of the NETCONF object
# to get back the current "source" configuration
# limiting the output using the filter
netconf_reply = m.get_config(source="running", filter=netconf_filter)
print( xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml() )
```

3. 使用m.edit_config(target="running", config=netconf_data)编辑配置

```python
from ncclient import manager
import xml.dom.minidom

# create a variable object that represents the NETCONF session
m = manager.connect(
         host="192.168.56.101",
         port=830,
         username="cisco",
         password="cisco123!",
         hostkey_verify=False
         )

netconf_data = """
<config><native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
     <hostname>NEWHOSTNAME</hostname>
</native></config>"""


netconf_reply = m.edit_config(target="running", config=netconf_data)



print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())



input("###### Press Enter to continue to Step 3 ######")

# create a new NETCONF config dataset
# it starts with the <config> element
# and inside includes the actual YANG config data
# It defines a Loopback 100 with 100.100.100.100/24
netconf_data = """
<config>
 <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
  <interface>
   <Loopback>
    <name>100</name>
    <description>TEST100</description>
    <ip>
     <address>
      <primary>
       <address>100.100.100.100</address>
       <mask>255.255.255.0</mask>
      </primary>
     </address>
    </ip>
   </Loopback>
  </interface>
 </native>
</config>
"""

# to edit the config, use the edit_config() method
# target config and source NETCONF dataset
# print the output (should include <ok/>)
netconf_reply = m.edit_config(target="running", config=netconf_data)
print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())


input("###### Press Enter to continue to Step 4 ######")

# create a new NETCONF config dataset
# it starts with the <config> element
# and inside includes the actual YANG config data
# It defines a Loopback 111 with 100.100.100.100/24
#  Note that it is the same IP as on the existing Lo100
netconf_data = """
<config>
 <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
  <interface>
   <Loopback>
    <name>111</name>
    <description>TEST111</description>
    <ip>
     <address>
      <primary>
       <address>100.100.100.100</address>
       <mask>255.255.255.0</mask>
      </primary>
     </address>
    </ip>
   </Loopback>
  </interface>
 </native>
</config>
"""

# to edit the config, use the edit_config() method
# target config and source NETCONF dataset
# print the output (in this case this transaction  should fail)
netconf_reply = m.edit_config(target="running", config=netconf_data)
print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())
```

4. 删除配置

```python
# Follow up of the lab 2.8 part 2 - NETCONF wPython Device Configuration
# Delete a Loopback interface

from ncclient import manager
import xml.dom.minidom

# create a variable object that represents the NETCONF session
m = manager.connect(
         host="192.168.56.101",
         port=830,
         username="cisco",
         password="cisco123!",
         hostkey_verify=False
         )

# the important XML tag parameters are:
#  define the he "nc" namespace: xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0"
#  call from the "nc" namespace the operation to delete  nc:operation="delete"
netconf_data = """
<config>
 <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
  <interface>
   <Loopback xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" nc:operation="delete">
    <name>100</name>
   </Loopback>
  </interface>
 </native>
</config>
"""

netconf_reply = m.edit_config(target="running", config=netconf_data)
print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())

```

5. 从xml中提取数据

```python
from ncclient import manager
import xml.dom.minidom

# create a variable object that represents the NETCONF session
m = manager.connect(
         host="192.168.56.101",
         port=830,
         username="cisco",
         password="cisco123!",
         hostkey_verify=False
         )

# define a NETCONF filter to get only the required data
# w/o this filter the NETCONF GET operation will try to 
# return everything and will crash (aka. similar to 'debug all')
# the filter defines that we want to get only data defined
# in the ietf-interfaces model in the interfaces-state container
netconf_filter = """
<filter>
 <interfaces-state xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"/>
</filter>
"""

# using the NETCONF get method, get data:
netconf_reply = m.get(filter = netconf_filter)
print(xml.dom.minidom.parseString(netconf_reply.xml).toprettyxml())


input("###### Press Enter to continue to Step 3 ######")
import xmltodict

# use the xmldict module to parse the NETCONF reply (in xml form)
# the retuned object is a Python dictionary
netconf_reply_dict = xmltodict.parse(netconf_reply.xml)

# loop over the Python dictionary object and print the interesting data
for interface in netconf_reply_dict["rpc-reply"]["data"]["interfaces-state"]["interface"]:
    print("Name: {} MAC: {} Input: {} Output {}".format(
                interface["name"],
                interface["phys-address"],
                interface["statistics"]["in-octets"],
                interface["statistics"]["out-octets"]
       )
    )

```



##### 3.使用RESTCONF配置管理

```python
import json
import requests
requests.packages.urllib3.disable_warnings()
#请求接口的配置信息
api_url = "https://192.168.1.5/restconf/data/ietf-interfaces:interfaces"

headers = { "Accept": "application/yang-data+json",
            "Content-type":"application/yang-data+json"
          }

basicauth = ("root", "root")

resp = requests.get(api_url, auth=basicauth, headers=headers, verify=False)

response_json = resp.json()
print(json.dumps(response_json, indent=4))
```

```python
import json
import requests
requests.packages.urllib3.disable_warnings()

api_url = "https://192.168.56.101/restconf/data/ietf-interfaces:interfaces/interface=Loopback99"

headers = { "Accept": "application/yang-data+json", 
            "Content-type":"application/yang-data+json"
          }

basicauth = ("cisco", "cisco123!")

yangConfig = {
    "ietf-interfaces:interface": {
        "name": "Loopback99",
        "description": "WHATEVER99",
        "type": "iana-if-type:softwareLoopback",
        "enabled": True,
        "ietf-ip:ipv4": {
            "address": [
                {
                    "ip": "99.99.99.99",
                    "netmask": "255.255.255.0"
                }
            ]
        },
        "ietf-ip:ipv6": {}
    }
}


resp = requests.put(api_url, data=json.dumps(yangConfig), auth=basicauth, headers=headers, verify=False)
if(resp.status_code >= 200 and resp.status_code <= 299):
    print("STATUS OK: {}".format(resp.status_code))
else:
    print("Error code {}, reply: {}".format(resp.status_code, resp.json()))

```


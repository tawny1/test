# OTA升级协议


## 1. 协议基本格式

|字节序号（相对）|字段长度|字段类型|协议内容|说明|
|-|-|-|-|-|
|**[0,1]**|**2**|**uint8_t**|**同步帧头**|**0xeb, 0x90**|
|**[2~5]**|**4**|**uint32_t**|**版本号**|**0xFFFFFFFF**|
|**[6~9]**|**4**|**uint32_t**|**源地址**|**发送方 0x00000000**|
|**[10~13]**|**4**|**uint32_t**|**目标地址**|**接收方 0x00000001**|
|**[14~15]**|**2**|**uint16_t**|**序列号Seq**|**初始化随机，单调递增回绕（各自为主体递增）**|
|**[16]**|**1**|**uint8_t**|**是否需要回应（确认位）**|**广播模式不用ACK回复**|
|**[17,18]**|**2**|**uint16_t**|**确认序列号ACK**|**回复确认赋值与收到的Seq一致即可。**|
|**[19,20]**|**2**|**uint16_t**|**数据体长度n**||
|**[21]**|**1**|**uint8_t**|**指令**||
|**[22~22+n-1]**|**n**| **—** |**指令数据**|**当数据体长度n=0时，该字段为空**|
|**[22+n]**|**1**|**uint8_t**|**校验和**|**[2~22+n-1]字节和**|
|**[23+n, 24+n]**|**2**|**uint8_t**|**帧尾**|**0x0d, 0x0a 2个连续字节**|

>标灰部分表示该数据可随机设置。
数据体长度n是指令数据部分的有效长度，不包括填充数据长度。

## 2. OTA相关协议集，指令ID为0x1f

### 2.1 命令字

| 字节序号（相对） | 字段长度 | 字段指令 | 协议内容                            | 说明                                    |      |
| ---------------- | -------- | -------- | ----------------------------------- | --------------------------------------- | ---- |
| **[21]**         | **1**    | **0x1f** | **ota相关指令集指令**               | **OTA 升级**                            |      |
| **[22]**         | **1**    | **0x01** | **获取版本号/响应**                 |                                         |      |
|                  |          | **0x02** | **请求启动升级**                    |                                         |      |
|                  |          | **0x03** | **请求bin文件分片/响应**            |                                         |      |
|                  |          | **0x04** | **升级状态上报/响应**               | **升级完成 跳转下位机app后发送**        |      |
|                  |          | **0x05** | **发送升级bin文件大小以及切片大小** | **回复升级文件相关信息  （app 回复 ）** |      |


### 2.2 数据22位协议细则-0x01

| 上位机主动请求 | 21位     |22位|23位|24-25|说明:版本号获取|
|-|-|-|-|-|-|
||**0x1f**|**0x01**|**0x01**||**23位（0x01ECU；0x02 VCU；0x03 电机；0x04 IMU；0x05 RTK；）**|
| **下位机响应** | **21位** |**22位**|**23位**|**24-27**|**说明**|
||**0x1f**|**0x01**|**0x01**|**0x01010101**|**0x01010101代表1.1.1.1**|

### 2.3 数据22位协议细则-0x02

| **上位机主动请求** | **21位** | **22位** | **23位** |            | **说明**：启动升级请求                                       |
| ------------------ | -------- | -------- | -------- | ---------- | ------------------------------------------------------------ |
|                    | **0x1f** | **0x02** | **0x01** |            | **23位（0x01ECU；0x02 VCU；0x03 电机；0x04 IMU；0x05 RTK；）** |
| **下位机响应**     | **21位** | **22位** | **23位** | **24-25**  | **说明**                                                     |
|                    | **0x1f** | **0x02** | **0x01** | **0x0001** | **0x0001代表成功<br/>0x0000代表失败<br/>其他状态待扩展**     |

### 2.4 数据22位协议细则-0x03

| **下位机主动请求** | **21位** | **22位** | **23位** | **24-25位** |              | **说明：请求bin文件分片/响应**                               |
| ------------------ | -------- | -------- | -------- | ----------- | ------------ | ------------------------------------------------------------ |
|                    | **0x1f** | **0x03** | **0x01** | **编号**    |              | **请求对应编号的分⽚（编号从0开始）<br/>23位（0x01ECU；0x02 VCU；0x03 电机；0x04 IMU；0x05 RTK；）** |
| **上位机响应**     | **21位** | **22位** | **23位** | **24-25位** | **24-27位**  | **说明**                                                     |
|                    | **0x1f** | **0x03** | **0x01** | **编号**    | **分⽚数据** | **最后⼀⽚的⼤⼩可能⼩于<br/>分⽚长度**                      |

### 2.4 数据22位协议细则-0x04

| **下位机主动上报** | **21位** | **22位** | **23位** | **24-25**  | **说明**：升级结果上报                                       |
| ------------------ | -------- | -------- | -------- | ---------- | ------------------------------------------------------------ |
|                    | **0x1f** | **0x04** | **0x01** | **0x0001** | **0x0001代表成功<br/>0x0000代表失败<br/>其他状态待扩展23位（0x01ECU；0x02 VCU；0x03 电机；0x04 IMU；0x05 RTK；）** |
| **上位机回复**     | **21位** | **22位** | **23位** |            | **说明**                                                     |
|                    | **0x1f** | **0x04** | **0x01** |            | **收到**                                                     |

### 2.5 数据22位协议细则-0x05  

| **下位机主动请求** | **21位** | **22位** | **23位** |                   |              |               | **说明**：升级固件包参数请求           |
| ------------------ | -------- | -------- | -------- | ----------------- | ------------ | ------------- | -------------------------------------- |
|                    | **0x1f** | **0x05** | **0x01** |                   |              |               |                                        |
| **上位机回复**     | **21位** | **22位** | **23位** | **24-27**         | **28-29**    | **30-41**     | **说明**: **下位机即将开始请求数据包** |
|                    | **0x1f** | **0x05** | **0x01** | **⽂件⼤⼩4字节** | **切⽚⼤⼩** | 固件包的MD5值 |                                        |



## 

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |

## 5. 交互流程

> - ```mermaid
>   sequenceDiagram
>   app->>下位机:获取版本号  指令(0x1F01)
>   app->>app:等待版本号ACK，3s超时，用户手动获取
>   下位机->>app:回复版本号
>   app->>下位机:启动升级请求  指令(0x1F02)
>   app->>app:等待升级回复ACK，超时重新请求（1s）
>   下位机->>app:升级请求ACK
>   Note right of 下位机: 下位机 跳转至bootloader
>   下位机->>app:下位机 进入bootloader （0x1F05）
>   下位机->>下位机:等待app回复ack，1s一次 ，<br>10s未收到app ack 升级失败 ，<br>下位机跳转至运行程序
>   app->>下位机:回复文件大小 切片信息 指令(0x1F05)
>   Loop 1
>   下位机->>app:请求升级bin文件切片 指令(0x1F03)
>   app->>下位机:切片文件下发 指令(0x1F03)
>   Note right of 下位机: bin文件烧写完成 <br>或者失败退出
>   end
>   Note right of 下位机: 下位机 跳转至 运行程序
>   下位机->>app:上报升级 指令(0x1F04)
>   app->>下位机:回复ACK
>   ```
>
> 
>
---
title: NFC基础
date: 2018-05-18 14:00:58
tags: code
---



### 卡片类型

[参考](https://blog.csdn.net/h3c4lenovo/article/details/8179879)

跟nfc有关的ISO标准：

| ISO 14443 | RFID卡标准（非接触IC卡），该标准又有很多子标准 |
| --------- | -------------------------- |
| ISO 7816  | 接触式IC卡标准                   |
| ISO 15693 | 某种射频卡标准吧，这个没查到资料           |
| ISO 18092 | NFC标准                      |



##### NfcA NfcB类型卡的区别

| Class            | Description                              |
| ---------------- | ---------------------------------------- |
| `TagTechnology`  | The interface that all tag technology classes must implement. |
| `NfcA`           | Provides access to NFC-A (ISO 14443-3A) properties and I/O operations. |
| `NfcB`           | Provides access to NFC-B (ISO 14443-3B) properties and I/O operations. |
| `NfcF`           | Provides access to NFC-F (JIS 6319-4) properties and I/O operations. |
| `NfcV`           | Provides access to NFC-V (ISO 15693) properties and I/O operations. |
| `IsoDep`         | Provides access to ISO-DEP (ISO 14443-4) properties and I/O operations. |
| `Ndef`           | Provides access to NDEF data and operations on NFC tags that have been formatted as NDEF. |
| `NdefFormatable` | Provides a format operations for tags that may be NDEF formattable. |

| Class              | Description                              |
| ------------------ | ---------------------------------------- |
| `MifareClassic`    | Provides access to MIFARE Classic properties and I/O operations, if this Android device supports MIFARE. |
| `MifareUltralight` | Provides access to MIFARE Ultralight properties and I/O operations, if this Android device supports MIFARE. |



#### 主动模式

NFC第一种通信模式计较主动模式，在主动模式下，nfc发起设备要发送数据给目标设备时，必须产生自己的射频场，被读nfc设备发送响应改发起设备时，也要产生自己的射频场。

#### 被动模式

在被动模式下，nfc发起设备的一方，在整个通信过程中提供射频场。

#### NFC工作模式和应用分类

读写模式，p2p模式，卡模拟模式（支付0模式）


















































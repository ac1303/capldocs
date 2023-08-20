## 通用

### `sendDiag()`

> 发送任意诊断请求

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| data       | byte []  | True         | 需要发送的数据                                               |
| length     | long     | False        | 数据的长度，若是不填写则默认为byte数组长度                   |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 byte q27[5]={0x27};
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
 sendDiag(handle,q27);	 //发送27 00 00 00 00
 sendDiag(handle,q27,1); //发送27
 sendDiag(handle,q27,3); //发送27 00 00
```



## 10服务

### `sendDiag1001()`

> 发送10 01诊断请求

详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



### `sendDiag1002()`

> 发送10 02诊断请求

详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



### `sendDiag1003()`

> 发送10 03诊断请求

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);

 sendDiag1001(handle); // 切换默认会话
 sendDiag1002(handle);
 sendDiag1003(handle);
```

**注意事项:**

切换会话后会等待500ms，因为可能会出现复位。可以自己修改代码取消等待



## 11服务

### `sendDiag1101()`

> 发送11 01复位诊断请求

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



## 14服务

### `sendDiag14FFFFFF()`

> 清除所有DTC故障

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



## 19服务

### `sendDiag190209()`

> 发送19 02 09诊断请求

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)

## 22服务

### `sendDiag22xxxx()`

> 读取DID

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| did        | word     | True         | 需要读取的DID                                                |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
 sendDiag22xxxx(handle,0xF90);	 //发送22 F1 90
```

## 27服务

### `sendDiag27_seed()`

> 27服务请求种子

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| lv         | byte     | True         | 安全等级                                                     |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
 sendDiag27_seed(handle,0x01);	 //发送27 01
```



### `sendDiag27_key()`

> 27服务请求种子

**Parameters:**

| **名称**   | **类型**      | **是否必选** | **说明**                                                     |
| ---------- | ------------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long          | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| lv         | byte          | True         | 安全等级                                                     |
| unlockKey  | dword/byte [] | True         | 数据类型二选一，可以传入dword类型也可以传入byte数组，byte数组只会取前四个byte作为密钥 |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
 sendDiag27_key(handle,0x01,0xA1B2C3D4);	 //发送27 02 A1 B2 C3 D4
```



###  `sendDiag27_UnlockSecurityLevel()`

> 27服务请求种子

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| lv         | byte     | True         | 安全等级                                                     |
| ecuName    | char []  | True         | ECU 名字，该数据要与工程中加载的CDD文件中一致，且工程需要配置CDD和解锁DLL，并且能在诊断控制台正常解锁 |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |

**代码示例:**

```c
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
 sendDiag1003(handle);
 sendDiag27_UnlockSecurityLevel(handle,0x01,"SBM");	 //解锁Lv 1 安全访问
```



## 28服务

### `sendDiag28xxxx()`

> 发送通信控制相关请求

用法、参数列表、返回值均与sendDiag22xxxx一样，详情请见 [sendDiag22xxxx()](/docs/Diag/Request?id=sendDiag22xxxx)



## 31服务

### `sendDiag31010203()`

> 发送31 01 02 03诊断请求

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



## 36服务

### `sendDiag36()`

> 36服务传输数据专用

**Parameters:**

| **名称**    | **类型** | **是否必选** | **说明**                                                     |
| ----------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle  | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| buffer      | byte []  | True         | 需要发送的数据                                               |
| bufferlen   | long     | True         | 数据的长度                                                   |
| bufferIndex | long     | True         | 数据开始下标                                                 |
| segmentSize | dword    | False        | 每个36传输多少个字节，若不填则为默认值130                    |

**return:**

| 类型 | 说明                                       |
| ---- | ------------------------------------------ |
| int  | 0表示传输过程出现非肯定应答，1表示传输完成 |

**代码示例:**

```
byte buffer[1000];
 long handle;
 handle = CreateCANConnection(diagTx,diagRx);
 
// 从buffer的第三个位开始传，总共传输300个byte，每次传输130个byte，需要注意的是这里的130表示（数据长度（128） + SID（1） + blockSequenceCounter（1））
sendDiag36(handle,buffer,300,3,130);
```



## 85服务

### `sendDiag8501()`

> 发送85 01诊断请求

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)



### `sendDiag8502()`

> 发送85 02诊断请求

用法与sendDiag1003一样，详情请见 [sendDiag1003()](/docs/Diag/Request?id=senddiag1003)

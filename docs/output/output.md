## 报文发送增强



### 示例

```c
long handle;
enableOutputPro();
handle = addCANMsg(0x123,100); //添加一条id为0x123，周期为100ms的报文
testwaitfortime(2000);

setOutPutProCycle(handle,200);//修改周期为200ms
setOutPutProFDF(handle,1);//将FDF位置一，代表该报文变成CANFD报文
setOutPutProMsgData(handle,0,0xFF)//将第一个byte设置为FF
testwaitfortime(2000);

delMsg(handle); // 删除该发送任务
clearMsg(handle); //清空所有发送任务
```



### enableOutputPro()

> 开启报文发送增强，若想使用该文件中的函数，必须调用该函数

**Parameters:**

| **名称** | **类型** | **是否必选** | **说明** |
| -------- | -------- | ------------ | -------- |
| 无参数   |          |              |          |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### addCANMsg

> 快速添加一条CAN报文到发送列表

添加长度为8的CAN报文，需要传入id和周期

**Parameters:**

| **名称** | **类型** | **是否必选** | **说明** |
| -------- | -------- | ------------ | -------- |
| id       | word     | 是           | 报文ID   |
| cycle    | word     | 是           | 报文周期 |

**return:**

| 类型 | 说明     |
| ---- | -------- |
| long | 任务句柄 |

### addCANFDMsg

> 快速添加一条CAN FD报文到发送列表

添加长度为8，FDF位和BRS位置一的CAN FD报文，需要传入id和周期

**Parameters:**

| **名称** | **类型** | **是否必选** | **说明** |
| -------- | -------- | ------------ | -------- |
| id       | word     | 是           | 报文ID   |
| cycle    | word     | 是           | 报文周期 |

**return:**

| 类型 | 说明     |
| ---- | -------- |
| long | 任务句柄 |

### long addMsg()

> 添加一条自定义报文

**Parameters:**

| **名称** | **类型**         | **是否必选** | **说明** |
| -------- | ---------------- | ------------ | -------- |
| msg      | struct OutPutMsg | 是           | 结构体   |

**return:**

| 类型 | 说明     |
| ---- | -------- |
| long | 任务句柄 |

```c
struct OutPutMsg{
    word msgId;
    word cycle;
    int dlc;
    int BRS;
    int FDF;
    byte msgData[64];
    word timeCount;
  };
```


## 报文发送增强



### 函数示例

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

### 故障老化代码实例

```c
testcase TG4_TC2_SC7_DTC_Aged(){       //故障老化示例
  enableOutputPro();
  long i,j,nm,ign,app1,app2,app3,app4;  
  for(i = 1;i<=43;i++){     // 老化43次
    nm = addCANMsg(0x500,20);     // 发送NM报文，周期为20ms
    setOutPutProMsgData(nm,1,0x40); // 设置NM报文第一个byte为40
    setOutPutProMsgData(nm,2,0x04);

    app1 = addCANFDMsg(0x20b,50);  // 发送节点丢失相关报文
    app2 = addCANFDMsg(0x20c,50);
    app3 = addCANFDMsg(0x20d,50);
    app4 = addCANFDMsg(0x20e,50);
    ign = addCANFDMsg(0x13C,20);  // 发送ign报文 
    testWaitForTimeout(100);
    setOutPutProMsgData(ign,3,0x02);  // 第三个byte是ign信号，02为ign on
    
    testWaitForTimeout(10000);
    
    setOutPutProMsgData(ign,3,0x00); // ign off
    testWaitForTimeout(200);
    delMsg(nm);                     // 停发NM
    testWaitForTimeout(200);
    delMsg(ign);		   // 停发ign
    testWaitForTimeout(100);
    delMsg(app1);		// 停发节点丢失相关报文
    delMsg(app2);
    delMsg(app3);
    delMsg(app4);
    testWaitForTimeout(8000);	//等待总线休眠
    write("第 %d 次老化 结束",i);
  }
}
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


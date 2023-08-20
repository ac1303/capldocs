# 诊断发送和接收相关
##  <font size=2>`Struct`</font> 数据结构<!-- {docsify-ignore} -->

> 几乎所有诊断功能都是在操作这个结构体，它是核心中的核心，你也可以直接从这个结构体中拿数据或者修改数据

```c
  struct Diag{
    long handle; // 句柄
    int diagReq; // // 0 无应答 ， 1有应答， 2流控帧，3 NRC 0x78
    dword setDelay; //延迟发送时间
    word p2server;
    word p2_server;// p2*server
    word timeout;
    
    long reqLength;// 发送的诊断服务数据
    byte reqData[4096];
    word respLength; // 诊断响应的实际长度
    byte respData[4096]; 
  }Diag;
  msTimer tSendAbortTimer_ms; // 停止发送计时器
  msTimer P2Timeout; //计算p2server和p2*server超时
```

## 发送过程<!-- {docsify-ignore} -->

`void sendDiag(long diagHandle, byte data[], int length)` 函数在发送过程中发挥了核心作用，其他的发送函数则可能是在这个函数的基础上进行进一步的封装，以适应不同的通信需求和协议要求。

```c
void sendDiag(long diagHandle,byte data[]){
  sendDiag(diagHandle, data, elCount(data));
}
void sendDiag(long diagHandle,byte data[],int length){
  Diag.diagReq = 0;
  Diag.reqLength = length;
  memcpy(Diag.reqData,data,length);
  CanTpSendData(diagHandle, data, length);
  setTimer(P2Timeout,Diag.p2server);
  waitDiagResp();
}
```

其他的发送函数可以根据不同的需求进行封装，例如：

```c
// 22服务
void sendDiag22xxxx(long diagHandle,word did){
  byte q22[3] = {0x22};
  q22[1] = did >> 8;
  q22[2] = did & 0xFF;
  sendDiag(diagHandle,q22);
}
// 36服务
int sendDiag36(long diagHandle,byte buffer[],long bufferlen,long buffertop,dword segmentSize){
  int i = 1,len;
  byte q36[4096] = {0x36};
  i = 1;
  do{
    q36[1] = i;
    if(bufferlen - buffertop < segmentSize-2){
      memcpy_off(q36,2,buffer,buffertop,bufferlen - buffertop);
      sendDiag(diagHandle,q36,(bufferlen - buffertop)+2);
      buffertop = bufferlen;
    }else{
      memcpy_off(q36,2,buffer,buffertop,segmentSize-2);
      buffertop += segmentSize-2;
      sendDiag(diagHandle,q36,segmentSize);
    }
    if(i == 0xFF){
      i=0;
    }else{
      i++;
    }
    if(Diag.reqData[0] + 0x40 != Diag.respData[0]){
      return 0;
    }
  }while(buffertop < bufferlen);
  return 1;
}
```

`void sendDiag(long diagHandle, byte data[], int length)` 函数主要做了以下几件事：

1. diagReq标志位置0

2. 拷贝数据到reqData中进行保存

3. 设置P2Timeout定时器

4. 通过CanTpSendData发送数据到总线

5. 调用waitDiagResp等待应答

   

## 接收过程<!-- {docsify-ignore} -->

接收诊断响应主要由两个函数共同完成

### <font size=2>`function`</font> CanTp_ReceptionInd<!-- {docsify-ignore} -->

CanTp_ReceptionInd是官方回调函数，当诊断出现应答时会调用这个函数，data为响应数据

```
CanTp_ReceptionInd(long handle, byte data[])
{
  int i;
  cancelTimer(P2Timeout); // 收到了应答，所以取消定时，如果该定时器被执行了，说明出现了超时
  i=elCount(data);
  if(data[0]==0x7f && data[2]==0x78){
    Diag.respLength = 0;
    Diag.diagReq = 3;  // 表示当前处于NRC 0x78状态
    setTimer(P2Timeout,Diag.p2_server); // 重新定时
    return;
  }
  memcpy(Diag.respData,data,i); //将获得的响应数据拷贝到Diag.respData
  Diag.respLength = i;
  Diag.diagReq = 1;// 将接收标志位置一，表示已经收到了响应
}
```

### <font size=2>`function`</font>waitDiagResp<!-- {docsify-ignore} -->

waitDiagResp函数的主要功能是等待，等待CanTp_ReceptionInd将应答标志位置一，若是在规定时间内应答标志位置没有置一，说明出现了超时，然后做一些善后处理

```
void waitDiagResp(){ // 总感觉这个函数会爆出来一个大坑。。。乱糟糟的
  word count;
  count = 0;
  while(Diag.diagReq == 0){ //未响应
    if(count == Diag.timeout){ 
      Diag.respLength = 0;
      Diag.setDelay = 0;
      break;
    }
    testWaitForTimeout(1);
    count++;
  };
  count = 0;
  // TODO 这里得加个跳出死循环的条件
  while(Diag.diagReq == 3){ // 收到 NRC 0x78
    testWaitForTimeout(1);
    count++;
  };
//  这里不再进行拷贝，响应数据和长度直接由CanTp_ReceptionInd回调函数拷贝到Diag.respData
  Diag.diagReq = 0;
}
```

当sendDiag发送诊断响应后，会调用waitDiagResp进行等待，有两种方式可以结束waitDiagResp的等待，一是CanTp_ReceptionInd收到应答，二是等待时间大于等待chao'shi

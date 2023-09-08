# 诊断发送和接收相关
##  <font size=2>`Struct`</font> 数据结构<!-- {docsify-ignore} -->

> 几乎所有诊断功能都是在操作这个结构体，它是核心中的核心，你也可以直接从这个结构体中拿数据或者修改数据。详情请看 [struct/DiagStruct.can](https://github.com/ac1303/capl/blob/main/V2.0/struct/DiagStruct.can)

```c

  enum DiagDir{
    TxMsg = 0,
    RxMsg = 1
  };
  enum RespType{
    Answer=1,
    diagTimeoutButAnswered=2,
    Sending=-1,
    SendFail=-2,
    WaitAnswer=-3,
    NoAnswer=-4,
    Pending=-5,   // 0x78
    FlowControl=-6,
    OverFlow=-7,
    diagTimeout=-8
  };
  // DiagMsg 用来存储发送和接收的数据
  struct DiagMsg{
    enum DiagDir dir;
    int dateLen;
    char date[61];
  } diagMsg;
  // Diag 存储连接的信息
  struct Diag{
    long handle; // 句柄
    enum RespType respInd; 
    
    long PendingCount;// NRC 0x78 等待次数
    long CFCount; // 连续帧计数
    long FCCount; // 流控帧计数
    dword delayTime; //延迟发送时间

    word p2server;
    word p2_server;// p2*server
    word timeout;  // 停止等待响应时间，超过改时间判定为诊断无应答

    long reqLength;// 发送的诊断服务数据
    byte reqData[4096];
    long respLength; // 诊断响应的实际长度
    byte respData[4096]; 
  };
  struct Diag _diagMap[long];
  struct DiagMsg _diagMsgMap[long,20];
  long _msgMapIndex = 0;
```

## 发送过程<!-- {docsify-ignore} -->

`void sendDiag(long diagHandle, byte data[], int length)` 函数在发送过程中发挥了核心作用，其他的发送函数则可能是在这个函数的基础上进行进一步的封装，以适应不同的通信需求和协议要求。

```c
void sendDiag(long diagHandle,byte data[]){
  sendDiag(diagHandle,data,elcount(data));
}
void sendDiag(long diagHandle,byte data[],long length){
  long count,CFCount,FCCount;
  count = 0;
  setDiagReqData(diagHandle,data,length);
  CanTpSendData(diagHandle, data, length);
  setDiagRespInd(diagHandle,Sending);
  while(getDiagRespInd(diagHandle) == Sending){
    testWaitForTimeout(1);
  }
  if(getDiagRespInd(diagHandle) == SendFail){
    return;
  }
  _waitResponse(diagHandle);
//  logging(debug,"sendDiag：诊断发送完成，当前状态是",getDiagRespInd(diagHandle));
}
```

其他的发送函数可以根据不同的需求进行封装，例如：

```c
int sendDiag36(long diagHandle,byte buffer[],long bufferlen,long buffertop){
  return sendDiag36(diagHandle,buffer,bufferlen,buffertop,130);
}
int sendDiag36(long diagHandle,byte buffer[],long bufferlen,long buffertop,dword segmentSize){
  int i = 1;
  byte q36[4096] = {0x36},respData[4096];
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
    if(respData[0] != 0x76){
      logging(warning,LOG_MESSAGE,"36服务收到否定应答");
      return 0;
    }
  }while(buffertop < bufferlen);
  return 1;
}

void sendDiag1001(long diagHandle){
  byte q1001[2]={0x10,0x01};
  sendDiag(diagHandle,q1001);
  testWaitForTimeout(500);
}
void sendDiag22xxxx(long diagHandle,word did){
  byte q22[3] = {0x22};
  q22[1] = did >> 8;
  q22[2] = did & 0xFF;
  sendDiag(diagHandle,q22);
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
void CanTp_ReceptionInd( long connHandle, byte data[])
{
  if(data[0]==0x7f && data[2]==0x78){
    setDiagPendingCount(connHandle,getDiagPendingCount(connHandle)+1);
    setDiagRespInd(connHandle,Pending);
    return;
  }
  setDiagRespData(connHandle,data,elcount(data));
  setDiagRespInd(connHandle,Answer);
  logging(debug,LOG_MESSAGE, "收到应答报文， connHandle = ",connHandle);
  logging(debug,LOG_MESSAGE,"应答：",data,elCount(data));
}
```

### <font size=2>`function`</font>waitDiagResp<!-- {docsify-ignore} -->

waitDiagResp函数的主要功能是等待，等待CanTp_ReceptionInd将应答标志位置一，若是在规定时间内应答标志位置没有置一，说明出现了超时，然后做一些善后处理

```

int _waitResponse(long diagHandle){
  return _waitResponse(diagHandle,0);
}
int _waitResponse(long diagHandle,long count){
  long PendingCount;
  long PendingTime;
  while(getDiagRespInd(diagHandle) == WaitAnswer){
    if(count >= getDiagP2server(diagHandle)){
      setDiagRespInd(diagHandle,diagTimeout);
      return _waitResponse(diagHandle,count);
    }
    count++;
    testWaitForTimeout(1);
  }
  while(getDiagRespInd(diagHandle) == Pending){
    if(getDiagPendingCount(diagHandle) != PendingCount){
      PendingCount = getDiagPendingCount(diagHandle);
      PendingTime = 0;
    }
    if(PendingTime >= getDiagP2server(diagHandle)){
      write(" NRC 0x78 诊断响应超时!!!");
      setDiagRespInd(diagHandle,NoAnswer);
      return diagTimeout;
    }
    PendingTime++;
    testWaitForTimeout(1);
  }
  while(getDiagRespInd(diagHandle) == FlowControl){
    // 暂时不做处理
    write("_waitResponse：收到流控帧，但是没有做相关处理，我认为不应该执行到这里，");
  }
  // 处理timeout
  while(getDiagRespInd(diagHandle) == diagTimeout){
    if(count >= getDiagTimeout(diagHandle)){
      setDiagRespInd(diagHandle,NoAnswer);
      logging(warning,LOG_MESSAGE,"诊断无应答！！！");
      return NoAnswer;
    }
    count++;
    testWaitForTimeout(1);
  }
  if(getDiagRespInd(diagHandle) == OverFlow){
    return OverFlow;
  }
  if(count <= getDiagP2server(diagHandle)){
    logging(debug,LOG_MESSAGE,"诊断响应成功，响应时间为 ",count," ms");
    setDiagRespInd(diagHandle,Answer);
    return diagTimeoutButAnswered;
  }else{
    logging(error,LOG_MESSAGE,"诊断响应超时，响应时间为 ",count ," ms");
    setDiagRespInd(diagHandle,diagTimeoutButAnswered);
    return diagTimeoutButAnswered;
  }
  write("_waitResponse出现一个错误，等待响应时间为 count = %d，当前状态为 %d",count,getDiagRespInd(diagHandle));
  return NoAnswer;
}
```

当sendDiag发送诊断响应后，会调用waitDiagResp进行等待


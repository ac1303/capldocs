## DiagCore中的接收函数

### `getDiagResp()`

> 获取响应数据，在getDiagRespDataAndLength的基础上进行封装，增加对响应状态的判定，避免获取到上一次的响应数据

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| data       | byte []  | True         | 接收响应数据的buffer                                         |

**return:**

| 类型 | 说明         |
| ---- | ------------ |
| long | 响应数据长度 |



## DiagStruct中的接收函数

### `getDiagRespDataAndLength()`

> 获取响应数据，不会判断响应是否超时，不建议使用

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| data       | byte []  | True         | 接收响应数据的buffer                                         |

**return:**

| 类型 | 说明         |
| ---- | ------------ |
| long | 响应数据长度 |


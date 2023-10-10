### `AssertDiagTrue()`

> 判定诊断的响应是否为正响应

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertDiagFalse()`

> 判定诊断的响应是否为负响应，

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| NRC        | byte     | False        | 可选参数，判定NRC是否与传入值一致                            |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertDiagNoResponse()`

> 判定诊断是否为无应答

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertDiagCompareBytes()`

> 判定响应数据是否与预期一致

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |

可选参数组合1：

| **名称** | **类型** | **是否必选** | **说明**           |
| -------- | -------- | ------------ | ------------------ |
| startIdx | long     | True         | 需要判定第几个byte |
| input    | byte     | True         | 判定值             |

可选参数组合2：

| **名称** | **类型** | **是否必选** | **说明**                     |
| -------- | -------- | ------------ | ---------------------------- |
| input    | byte[]   | True         | 判断响应数据是否与该数组一致 |
| length   | long     | True         | 传入数据的长度               |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertIsASCII()`

> 判定诊断响应从startIdx位开始到endIdx位的数据是否符合ASCII格式要求

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| startIdx   | long     | True         | 起始位置                                                     |
| endIdx     | long     | True         | 结束位置                                                     |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertIsBCD()`

> 判定诊断响应从startIdx位开始到endIdx位的数据是否符合BCD格式要求

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| startIdx   | long     | True         | 起始位置                                                     |
| endIdx     | long     | True         | 结束位置                                                     |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |



### `AssertIsZero()`

> 判定诊断响应从startIdx位开始到endIdx位的数据是否为0

**Parameters:**

| **名称**   | **类型** | **是否必选** | **说明**                                                     |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| diagHandle | long     | True         | 诊断句柄，通过CreateCANConnection获得，或者CanTpCreateConnection获取 |
| startIdx   | long     | True         | 起始位置                                                     |
| endIdx     | long     | True         | 结束位置                                                     |

**return:**

| 类型 | 说明       |
| ---- | ---------- |
| Void | 没有返回值 |


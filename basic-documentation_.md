# (Basic) Robot Management API Documentation

| 版本      | 日期       | 修改摘要                                                     |
| --------- | ---------- | ------------------------------------------------------------ |
| 1.00.0000 | 2020/09/01 | API 初版                                                     |
| 1.00.1100 | 2020/10/27 | 1.  新增modbus function  <br />2.  修改http url              |
| 1.00.1200 | 2020/11/12 | 調整input output 資料格式                                    |
| 1.00.1300 | 2020/11/19 | 1. 修改Modbus回傳欄位:Readbyte->ReadByte  <br />2. 修改Modbus function code 代號  <br />3. 修改http 控制相關command url |
| 1.00.1400 | 2020/11/24 | 1.  修改Login function name to GetToken<br />2.  新增訂閱方法訂閱訊息說明<br />3. 修改手臂資訊欄位名稱 |

  * [APIs Overview](#apis-overview)
  * [Socket API](#socket-api)
    + [Get Authorization Token](#get-authorization-token)
    + [Subscribe TM Robot Information](#subscribe-tm-robot-information)
    + [Unsubscribe TM Robot Information](#unsubscribe-tm-robot-information)
    + [Subscribe TM Robot Variable](#subscribe-tm-robot-variable)
    + [Unsubscribe TM Robot Variable](#unsubscribe-tm-robot-variable)
    + [Get TM Robots Information](#get-tm-robots-information)
    + [Get TM Robots Project Information](#get-tm-robots-project-information)
    + [Get TM Robot Variable](#get-tm-robot-variable)
    + [Get TM Robot Variable Value](#get-tm-robot-variable-value)
    + [Get Modbus](#get-modbus)
  * [Web API](#web-api)
    + [Get Authorization Token](#get-authorization-token-1)
    + [Get TM Robot Information](#get-tm-robot-information)
    + [Get TM Robot Project Information](#get-tm-robot-project-information)
    + [Get TM Robot Variable](#get-tm-robot-variable-1)
    + [Get TM Robot Variable Value](#get-tm-robot-variable-value-1)
    + [Get Modbus Value](#get-modbus-value)
  * [RabbitMQ Client](#rabbitmq-client)
  * [Socket Object Properties](#socket-object-properties)
    + [Reponse](#reponse)
    + [ServiceActionReturn](#serviceactionreturn)
    + [TMRobotActionReturn](#tmrobotactionreturn)
    + [TMRobotInfo](#tmrobotinfo)
    + [TMRobotProject](#tmrobotproject)
    + [TMRobotVariable](#tmrobotvariable)
    + [TMRobotVariableValue](#tmrobotvariablevalue)
    + [ModbusActionReturn](#modbusactionreturn)
  * [Field Specification](#field-specification)
    + [ActionTarget](#actiontarget)
    + [ServiceActionType](#serviceactiontype)
    + [RobotActionType](#robotactiontype)
    + [ModbusFunction](#modbusfunction)
    + [TMRobotLightColor](#tmrobotlightcolor)
    + [TMRobotStatus](#tmrobotstatus)

## APIs Overview

(Basic) Robot Management API 僅提供讀取TM Robot相關資訊。

透過TM Robot HMI 設定連線功能，可連線 Robot Management API，使用者便可透過Robot Management API取得所有線上連線中的TM Robot 資訊。

APIs 提供方式有

- Socket API
- Web API
- RabbitMQ Client

**Method Matrix**

|  #    | Method                    | Socket API             | Web API |
| ---- | --------------------------- | :------------------: | :--: |
| 1 | Get Authorization Token | O | O |
| 2 | Get TM Robots Information | O  | O |
| 3   | Get TM Robots Project Information | O  | O |
| 4   | Get TM Robot Variable | O  | O |
| 5 | Get TM Robot Variable Value | O | O |
| 6  | Subscribe TM Robot Information | O       | X |
| 7  | Subscribe TM Robot Variable | O        | X |
| 8  | Unsubscribe TM Robot Information | O       | X |
| 9  | Unsubscribe TM Robot Variable | O        | X |
| 10 | Get Modbus | O | O |

**RabbitMQ Queues**

| #    | Queue Description            |
| ---- | ---------------------------- |
| 1    | Publish TM Robot Information |
| 2    | Publish TM Robot Variable    |

Ports

| #    | API TYPE   | Port |
| ---- | ---------- | ---- |
| 1    | Socket API | 9834 |
| 2    | Web API    | 9832 |

## Socket API

利用TCP/IP Socket通道存取資料，資料格式為 JSON 進行傳輸，結尾符號為換行符號 `\n`

### Get Authorization Token

根據API Config定義的使用者名稱/密碼，取得API存取Token，以便API存取資料時驗證使用。

**Arguments**

| 參數名稱               | 是否必填 | 參數格式 | 說明                                             |
| ---------------------- | -------- | -------- | ------------------------------------------------ |
| ActionTarget           | 是       | int      | Socket 動作命令對象，帶入 `0` 表示對象為 Service |
| ServiceActionParameter |          | object   |                                                  |
| ServiceActionType      | 是       | int      | Service 類型動作識別碼，此方法帶入值為 `0`       |
| UserName               | 是       | string   | 使用者名稱                                       |
| Password               | 是       | string   | 使用者密碼                                       |

```json
{
  "ActionTarget": 0,
  "ServiceActionParameter": {
    "ServiceActionType": 0,
    "UserName": "Admin",
    "Password": "123"
  }
}
```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 0,
  "ServiceActionReturn": {
    "serviceActionType": 0,
    "Token": "123456"
  }
}
```

### Subscribe TM Robot Information

根據輸入的參數內容，訂閱手臂資訊。當連線手臂的資訊改變時，會主動發佈手臂資訊至訂閱對象

手臂資訊請參照[取得手臂資訊](#_bookmark60)

**Arguments**

| 參數名稱               | 是否必填 | 參數格式 | 說明                                           |
| ---------------------- | -------- | -------- | ---------------------------------------------- |
| Token                  | 是       | string   | API 存取Token                                  |
| ActionTarget           | 是       | int      | Socket 動作命令對象，帶入 `1` 表示對象為 Robot |
| TMRobotActionParameter |          | object   |                                                |
| RobotActionType        | 是       | 整數     | Robot 類型動作識別碼，此方法帶入值為 `4`       |
| TMRobotName            | 是       | 字串     | 手臂名稱                                       |

 ```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 4,
    "TMRobotNames": [
      "TM123456"
    ]
  }
}
 ```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 4
  }
}
```

### Unsubscribe TM Robot Information

根據輸入的參數內容，取消訂閱手臂資訊

**Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                           |
| ---------------------- | -------- | ------------ | ---------------------------------------------- |
| Token                  | 是       | string       | API 存取Token                                  |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot |
| TMRobotActionParameter |          | object       |                                                |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `5`       |
| TMRobotNames           | 是       | String array | 手臂名稱清單                                   |

```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 5,
    "TMRobotNames": [
      "TM123456"
    ]
  }
}
```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 5
  }
}
```

### Subscribe TM Robot Variable

根據輸入的參數內容，訂閱手臂變數，當連線中手臂的變數值改變時，會主動發佈手臂資訊至訂閱對象

手臂資訊請參照取得[手臂變數值](#_取得手臂變數值)

 **Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                                         |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | 是       | string       | API 存取Token                                                |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot               |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `6`                     |
| TMRobotName            | 是       | string       | 手臂名稱                                                     |
| VariableNames          | 是       | string array | 變數名稱清單；若欄位為`null` 或 `[]` 或不帶欄位，則會訂閱該手臂所有的變數 |

```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 6,
    "TMRobotName ": "TM123456",
    "VariableNames": [
      "var_a"
    ]
  }
}
```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 6
  }
}
```

### Unsubscribe TM Robot Variable

根據輸入的參數內容，取消訂閱手臂變數

**Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                                         |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | 是       | string       | API 存取Token                                                |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot               |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `7`                     |
| TMRobotName            | 是       | string       | 手臂名稱                                                     |
| ProjectName            | 是       | string       | 專案名稱                                                     |
| VariableNames          | 是       | string array | 變數名稱清單；若欄位為`null` 或 `[]` 或不帶欄位，則取消該手臂所有的變數訂閱 |

 ```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 7,
    "TMRobotName ": "TM123456",
    "ProjectName": "Test",
    "VariableNames": [
      "var_a"
    ]
  }
}
 ```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 6
  }
}
```

### Get TM Robots Information

根據輸入的參數內容，取得單台或多台連線中的手臂資訊

**Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                                         |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | 是       | string       | API 存取Token                                                |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot               |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `0`                     |
| TMRobotNames           | 否       | string array | 手臂名稱清單；若欄位為`null` 或 `[]` 或不帶欄位，則回傳所有連線中的手臂 |

 ```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 0,
    "TMRobotNames": [
        "TM123456"
    ]
  }
}
 ```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 0,
    "TMRobotInfos": [
      {
        "TMRobotName": "TM123456",
        "IP": "192.168.132.11",
        "Port": 63327,
        "Model": "TM12",
        "EnabledHMIControl ": true,
        "HasRightControl ": false,
        "EnabledAutoRemoteMode": false,
        "EnabledSpeedControl": false,
        "TMRobotLightColor": 3,
        "TMRobotStatus": 0,
        "Speed": 0,
        "HMIVersion": "",
        "DefaultProjectName": "Test",
        "StatusUpdateTime": "2020-11-12T20:08:26.9991835+08:00",
        "IsSubscribed": false,
        "ErrorCode": ""
      }
    ]
  }
}
```

### Get TM Robots Project Information

根據輸入的參數內容取得連線中手臂的專案名稱清單

**Arguments**

| 參數名稱               | 是否必填 | 參數格式 | 說明                                           |
| ---------------------- | -------- | -------- | ---------------------------------------------- |
| Token                  | 是       | string   | API 存取Token                                  |
| ActionTarget           | 是       | int      | Socket 動作命令對象，帶入 `1` 表示對象為 Robot |
| TMRobotActionParameter |          | object   |                                                |
| RobotActionType        | 是       | int      | Robot 類型動作識別碼，此方法帶入值為 `1`       |
| TMRobotName            | 是       | string   | 手臂名稱                                       |

 ```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 1,
    "TMRobotName": "TM123456"
  }
}
 ```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 1,
    "TMRobotProjects": [
      {
        "TMRobotName": "TM123456",
        "ProjectName": "Test"
      }
    ]
  }
}
```

### Get TM Robot Variable

根據輸入的參數內容取得連線中手臂的變數屬性資訊，變數種類包含有全域變數以及TM Robot運行中的專案變數

**Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                                    |
| ---------------------- | -------- | ------------ | ------------------------------------------------------- |
| Token                  | 是       | string       | API 存取Token                                           |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot          |
| TMRobotActionParameter |          | object       |                                                         |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `2`                |
| TMRobotName            | 是       | string       | 手臂名稱                                                |
| VariableNames          | 否       | string array | 若欄位為`null` 或 `[]` 或不帶欄位，則回傳所有的手臂變數 |

```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 2,
    "TMRobotName": "TM123456",
    "VariableNames": [
        "var_a"
    ]
  }
}
```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 2,
    "TMRobotVariables": [
      {
        "Name": "var_a",
        "ProjectName": "Test",
        "IsGlobalVariable": true,
        "IsSubscribed": false
      }
    ]
  }
}
```

### Get TM Robot Variable Value

根據輸入的參數內容取得連線中手臂的變數值

**Arguments**

| 參數名稱               | 是否必填 | 參數格式     | 說明                                                    |
| ---------------------- | -------- | ------------ | ------------------------------------------------------- |
| Token                  | 是       | string       | API 存取Token                                           |
| ActionTarget           | 是       | int          | Socket 動作命令對象，帶入 `1` 表示對象為 Robot          |
| TMRobotActionParameter |          | object       |                                                         |
| RobotActionType        | 是       | int          | Robot 類型動作識別碼，此方法帶入值為 `3`                |
| TMRobotName            | 是       | string       | 手臂名稱                                                |
| VariableNames          | 否       | string array | 若欄位為`null` 或 `[]` 或不帶欄位，則回傳所有的專案變數 |

 ```json
{
  "Token": "123456",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 3,
    "TMRobotName": "TM123456",
    "VariableNames": [
      "var_a"
    ]
  }
}
 ```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 3,
    "TMRobotVariableValues": [
      {
        "TMRobotName": "TM123456",
        "ProjectName": "Test ",
        "vType": "string",
        "vName": "var_a",
        "Value": "Test",
        "PastValue": ""
      }
    ]
  }
}
```

### Get Modbus

根據輸入的參數內容讀取手臂 Modbus Address 並取得 Value

手臂Modbus Table請參閱 HMI Specification

 **Arguments**

| 參數名稱              | 是否必填 | 參數格式 | 說明                                                         |
| --------------------- | -------- | -------- | ------------------------------------------------------------ |
| Token                 | 是       | string   | API 存取Token                                                |
| ActionTarget          | 是       | int      | Socket 動作命令對象，帶入 `2` 表示對象為 Modbus              |
| ModbusActionParameter |          | object   |                                                              |
| ModbusFunction        | 是       | int      | Modbus function code 請參閱 [Field Specification](#modbusfunction) |
| DeviceIP              | 是       | string   | 設備 IP Address                                              |
| Port                  | 是       | int      | Modbus Port 502                                              |
| SlaveAddress          | 是       | int      | Modbus Slave Id                                              |
| StartAddress          | 是       | int      | Modbus 起始位置                                              |
| NumberOfPoint         | 是       | int      | Modbus Address 讀取數量                                      |

```json
{
  "Token": "123456",
  "ActionTarget": 2,
  "ModbusActionParameter": {
    "ModbusFunction": 1,  // 3或4的回應結果為Response第二段 
    "DeviceIP": "192.168.132.169",
    "Port": 502,
    "SlaveAddress": 1,
    "StartAddress": 0,
    "NumberOfPoint": 3
  }
}
```

**Response**

- Typical success response

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 2,
  "ModbusActionReturn": {
    "ModbusFunction": 1,
    "ReadBools": [
      false,
      false,
      false
    ]
  }
}
```
```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 2,
  "ModbusActionReturn": {
    "ModbusFunction": 3,
    "ReadRegisters": [
        3,
        4
    ]
  }
}
```


## Web API

利用 HTTP 通道存取資料，請求標頭需帶Authorization存取Token，根據 URL 以及所帶入的 JSON 格式參數查詢相對應的手臂資料

### Get Authorization Token

根據API Config定義的使用者名稱/密碼，取得API存取Token，以便API存取資料時驗證使用。

Method URL: `http://{server_ip}:9832/HttpServer/GetToken`

HTTP method: `POST`

Headers

```http
Content-Type: application/json
```

Body

```http
{"username":"Admin","password":"123"}
```

**Arguments**

| 參數名稱 | 是否必填 | 參數格式 | 說明       |
| -------- | -------- | -------- | ---------- |
| username | 是       | string   | 使用者名稱 |
| password | 是       | string   | 使用者密碼 |

 **Response**

- Typical success response

```json
{
  "userName": "Admin",
  "password": "123",
  "token": "123456"
}
```

### Get TM Robot Information

取得單台或多台連線中的手臂資訊

Method URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobot/{ TMRobotName }`

| 參數名稱    | 是否必填 | 參數格式 | 說明                                     |
| ----------- | -------- | -------- | ---------------------------------------- |
| TMRobotName | 否       | string   | 手臂名稱，如果未填則回傳所有連線中的手臂 |

HTTP method: `GET` `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

 **Response**

- Typical success response，回應內容可參考[TMRobotInfo](#tmrobotinfo)


```json
[
  {
    "tmRobotName": "TM123456",
    "ip": "192.168.132.11",
    "port": 63327,
    "model": "TM12",
    "enabledHMIControl": true,
    "hasRightControl": false,
    "tmRobotLightColor": 3,
    "tmRobotStatus": 3,
    "speed": 0,
    "hmiVersion": "",
    "defaultProjectName": "Test",
    "statusUpdateTime": "2020-11-12T20:24:52.617899+08:00",
    "isSubscribed": false,
    "errorCode": ""
  }
]
```

### Get TM Robot Project Information

取得連線中的手臂專案名稱清單

Method URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotProject/{ TMRobotName }`

| 參數名稱    | 是否必填 | 參數格式 | 說明     |
| ----------- | -------- | -------- | -------- |
| TMRobotName | 是       | string   | 手臂名稱 |

HTTP method: `GET` `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

**Response**

- Typical success response，回應內容可[參考](#tmrobotproject)

```json
[
  {
    "tmRobotName": "TM123456",
    "projectName": "Test"
  }
]
```

### Get TM Robot Variable

取得手臂專案使用變數的屬性資訊，變數種類包含有全域變數以及TM Robot運行中的專案變數清單

Method URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotVariable/{ TMRobotName }/{ProjectName}`

| 參數名稱    | 是否必填 | 參數格式 | 說明               |
| ----------- | -------- | -------- | ------------------ |
| TMRobotName | 是       | string | 手臂名稱 |
| ProjectName | 是     | string | 專案名稱 |

HTTP method: `GET` `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

**Response**

- Typical success response，回應內容可[參考](#tmrobotvariabl)

```json
[
  {
    "name": "var_a",
    "projectName": "Test",
    "isGlobalVariable": true,
    "isSubscribed": false
  }
]
```

### Get TM Robot Variable Value

取得手臂專案使用變數的值，變數種類包含有全域變數與專案變數，內容包含有當下以及前一次的值

Method URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotVariableValue/{ RobotName }/{ ProjectName }/{VariableName}`

| 參數名稱     | 是否必填 | 參數格式 | 說明     |
| ------------ | -------- | -------- | -------- |
| TMRobotName  | 是       | string   | 手臂名稱 |
| VariableName | 是       | string   | 變數名稱 |


HTTP method: `GET` `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

**Response**

- Typical success response，回應內容可[參考](#tmrobotvariablevalue)

```json
[
  {
    "tmRobotName": "TM123456",
    "projectName": "Test ",
    "vType": "string",
    "vName": "var_a",
    "value": "Test",
    "pastValue": ""
  }
]
```

### Get Modbus Value

取得連線中手臂的 Modbus Address Value，手臂Modbus Table請參閱 HMI Specification

Method URL: `http://{server_ip}:9832/HttpServerModbus/ReadModbus`


HTTP method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Body

```json
{
  "ModbusFunction": 1,
  "DeviceIP": "192.168.132.169",
  "Port": 502,
  "SlaveAddress": 1,
  "StartAddress": 0,
  "NumberOfPoint": 3
}
```

| 參數名稱       | 是否必填 | 參數格式 | 說明         |
| -------------- | -------- | -------- | ------------ |
| ModbusFunction        | 是       | int      | Modbus Read Function Code 請參考 [Field Specification](#modbusfunction) |
| DeviceIP              | 是       | string   | 設備 IP                   |
| Port                  | 是       | int      | Modbus Port 502           |
| SlaveAddress          | 是       | int      | Modbus Slave Id           |
| StartAddress          | 是       | int      | Modbus 起始位置           |
| NumberOfPoint         | 是       | int      | Modbus Address 讀取數量   |

**Response**

- Typical success response，回應內容可[參考](#modbusactionreturn)

```json
{
  "modbusFunction": 1,
  "readBools": [
    false,
    false,
    false
  ]
}
```

## RabbitMQ Client

Linux 請在 docker-compose.yml 設定

Windows 請至 appsetting.config 設定

目前提供 Publish 如下

1. TMRobotInfoRoute 推送手臂資訊，內容請參閱 [TMRobotInfo](#tmrobotinfo)
2. TMRobotVariableRoute 推送變數值，內容請參閱[TMRobotVariableValue](#tmrobotvariablevalue)

---

## Socket Object Properties

### Reponse

| 參數名稱            | 參數型別 | 參數說明                                                     |
| ------------------- | -------- | ------------------------------------------------------------ |
| Success             | bool     | 請求成功結果                                                 |
| Message             | string   | 請求失敗訊息顯示內容，成功則為空                             |
| ActionTarget        | int      | 對應 Socket API Method 對象，請參考 [File Specification](#actiontarget) |
| ServiceActionReturn | object   | Service 類型動作回傳物件                                     |
| TMRobotActionReturn | object   | Robot 類型動作回傳物件                                       |
| ModbusActionReturn  | object   | Modbus 類型動作回傳物件                                      |

### ServiceActionReturn

| 參數名稱          | 參數型別 | 參數說明                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| ServiceActionType | string   | Service 類型動作識別碼，請參考 [File Specification](#serviceactiontype) |
| Token             | string   | API 存取Token                                                |

### TMRobotActionReturn

| 參數名稱              | 參數型別     | 參數說明                  |
| --------------------- | ------------ | ------------------------- |
| RobotActionType       | int          | 對應 Socket API 提供的 Robot 類型 Function，請參考 [File Specification](#robotactiontype) |
| TMRobotInfos          | array object | 手臂資訊物件清單          |
| TMRobotProjects | array object | 手臂專案資訊物件清單 |
| TMRobotVariables | array object | 手臂變數屬性資訊物件清單 |
| TMRobotVariableValues | array object | 手臂變數值資訊物件清單 |


### TMRobotInfo

| 參數名稱              | 參數型別     | 參數說明                  |
| --------------------- | ------------ | ------------------------- |
| TMRobotName           | string       | 手臂名稱                  |
| IP                    | string       | 手臂 IP Address     |
| Port                  | int          | 手臂 Port                |
| Model                 | string       | 手臂型號                  |
| EnabledHMIControl     | bool         | HMI是否已登入取得 Control Permission |
| EnabledAutoRemoteMode | bool         | HMI是否已開啟遠程控制     |
| EnabledSpeedControl   | bool         | HMI是否已開啟加速控制     |
| HMIVersion            | string       | HMI版本                   |
| HasRightControl       | bool         | API使用者是否有存取控制權 |
| TMRobotLightColor     | int          | 手臂燈號代碼，請參考 [File Specification](#tmrobotlightcolor) |
| TMRobotStatus         | int          | 手臂狀態代碼                |
| StatusUpdateTime      | datetime   | 手臂狀態變動更新時間      |
| Speed                 | int          | 手臂速度(%)               |
| DefaultProjectName    | string       | 預設專案名稱       |
| ErrorCode             | string       | 錯誤代碼，說明請參閱 TMflow 說明書附錄 C |
| IsSubscribed         | bool         | 是否已訂閱               |

### TMRobotProject

| 參數名稱       | 參數型別     | 參數說明             |
| -------------- | ------------ | -------------------- |
| TMRobotName | string | 手臂名稱 |
| ProjectName           | bool         | 專案名稱                  |

### TMRobotVariable

| 參數名稱       | 參數型別     | 參數說明             |
| -------------- | ------------ | -------------------- |
| Name          | string       | 手臂變數名稱              |
| ProjectName | string   | 專案名稱      |
| IsGlobalVariable | bool | 是否為全域變數 |
| IsSubscribed   | bool | 是否已訂閱   |

### TMRobotVariableValue

| 參數名稱       | 參數型別     | 參數說明             |
| -------------- | ------------ | -------------------- |
| TMRobotName | string | 手臂名稱 |
| ProjectName           | bool         | 專案名稱                  |
| VariableType | string | 變數型別 |
| VariableName | string | 變數名稱 |
| VariableValue | string | 變數值 |

### ModbusActionReturn

| 參數名稱       | 參數型別     | 參數說明                                                     |
| -------------- | ------------ | ------------------------------------------------------------ |
| ModbusFunction | int          | Modbus function code 請參考 [File Specification](#modbusfunction) |
| ReadBools      | bool array   | ReadCoils或ReadInputs讀取多筆的回傳值                        |
| ReadRegisters  | ushort array | ReadHoldingRegisters或ReadInputRegisters讀取多筆的回傳值     |



## Field Specification

### ActionTarget

| 值   | 說明                                  |
| ---- | ------------------------------------- |
| 0    | Socket API 用來辨識命令對象為 Service |
| 1    | Socket API 用來辨識命令對象為 Robot   |
| 2    | Socket API 用來辨識命令對象為 Modbus  |

### ServiceActionType

Service 類型動作識別碼，對應 Socket API 提供的 Service 類型方法

| 值   | 說明                          |
| ---- | ----------------------------- |
| 0    | 取得存取Token                 |

### RobotActionType 

Robot 類型動作識別碼，對應 Socket API 提供的 Robot 類型方法

| 值   | 說明                 |
| ---- | -------------------- |
| 0    | 取得手臂資訊         |
| 1    | 取得手臂專案資訊     |
| 2    | 取得手臂變數屬性資訊 |
| 3    | 取得手臂變數值       |
| 4    | 訂閱手臂資訊         |
| 5    | 取消訂閱手臂資訊     |
| 6    | 訂閱手臂變數         |
| 7    | 取消訂閱手臂變數     |

### ModbusFunction

Modbus function code 對應識別碼

| 值   | 說明                 |
| ---- | -------------------- |
| 1    | ReadCoils            |
| 2    | ReadInputs           |
| 3    | ReadHoldingRegisters |
| 4    | ReadInputRegisters   |

### TMRobotLightColor

TM Robot 燈號識別碼，燈號意義請參閱 TMflow 說明書附錄 B

| 值   | 說明                      |
| ---- | ------------------------- |
| 0    | Light off                 |
| 1    | Solid Red                 |
| 2    | Flashing Red              |
| 3    | Solid Blue                |
| 4    | Flashing Blue             |
| 5    | Solid Green               |
| 6    | Flashing Green            |
| 9    | Alternating Blue-Red      |
| 10   | Alternating Green-Red     |
| 13   | Alternating  Purple-Green |
| 14   | Alternating  Purple-Blue  |
| 17   | Alternating  White-Green  |
| 18   | Alternating  White-Blue   |
| 19   | Flashinglightblue         |

### TMRobotStatus

TM Robot 狀態識別碼

| 值   | 說明  |
| ---- | ----- |
| 0    | Play  |
| 1    | Error |
| 2    | Pause |
| 3    | Stop  |
| 4    | EStop |


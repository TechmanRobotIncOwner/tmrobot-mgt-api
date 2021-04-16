# Robot Management API Documentation (Beta)

## 1. Overview

TM Robot Management API is an API service for system developers to integrate TM Robot. It supports  communication protocols of TCP/IP, HTTP, and RabbitMQ with the complement API methods such as  Socket API, WebAPI, RabbitMQ Client, and gRPC. 

For utilizations of the TM Robot data in user applications, users can  make multiple connections by the available multi-client connection of basic functions.

For System developers also can control multiple TM Robots through advance functions and implement the robotic arm center or robotic arm IOT unit in the factory.

Supported TM Robot software version: 1.72/1.76/1.80 and above

![image-20210414181303906](images/image-20210414181303906.png)

## 2. TM Robot Settings

- Turn on the Modbus function

- Option and limitation

  To do this using the Robot Management API: project list, variable manifest, VarSync variable delivery, project upload/download, it cannot be shared with TMmanager software. Launch HMI TMmanager function and complete the settings.

  | IP Address                      | Port |
  | ------------------------------- | ---- |
  | TM Robot Management API Host IP | 9835 |

## 3. TM Robot Management API Host Preparation

- Static IP address

### Linux

> IMPORTANT: This is beta version so that please contact teachman team for `docker-compose.yml` if needed.

- (Preferred) Ubuntu Bionic 18.04 (LTS) 
- Install Docker  
- Install Docker Compose

**Start via Docker**

1. Open terminal and paste the command as below to create the service folder.

   ```bash
   mkdir robot-management-api
   ```

2. Paste the file `docker-compose.yml` into `robot-management-api` folder.

3. Refers to  [ServiceSettings](#servicesettings.json) and [ModbusDetail](#modbusdetail.json) column description and customize `docker-compose.yml`. 

4. Open terminal and change the directory to the folder `robot-management-api` and run the command as below.

   ```bash
   docker-compose up -d
   ```

5. Launch the URL `http://{server_ip}:9832/HttpServer/TestRobotApiConnect` for the Success as  the service of TM Robot Management API started successfully. 

### Windows

- Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later). 

- .NET Desktop Runtime 3.1.14 from Microsoft

- ASP.NET Core Runtime 3.1.14 from Microsoft

**Start**

1. Download TM Robot Management API.
2. Change directory to RobotManagementAPI.
3. Customize configuration files and refers to [ServiceSettings](#servicesettings.json) and [ModbusDetail](#modbusdetail.json)
   1. `RobotManagementAPI\Settings\ServiceSettings.json`
   2. `RobotManagementAPI\Settings\ModbusSetting.json`
4. Execute TM Robot Management API service.
5. Launch the URL `http://{server_ip}:9832/HttpServer/TestRobotApiConnect` for `Connection true` as  the service of TM Robot Management API started successfully.

### ServiceSettings.json

TM Robot Management API configuration files come with `ServiceSettings.json` environment variables. The column descriptions are as follows:

| Column Name                      | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| MQSetting.IP                     | Rabbit MQ Server IP                                          |
| MQSetting.Port                   | Rabbit MQ Server Port                                        |
| MQSetting.UserName               | Rabbit MQ Server UserName                                    |
| MQSetting.Password               | Rabbit MQ Server Password                                    |
| MQSetting.VHost                  | Rabbit MQ Server Virtual Host Name                           |
| MQSetting.Exchange               | Rabbit MQ Server Exchange Name                               |
| MQSetting.TMRobotInfoRoute       | Rabbit MQ Server Routing Key TM Robot Information            |
| MQSetting.TMRobotVariableRoute   | Rabbit MQ Server Routing Key TM Robot Variable Value         |
| MQSetting.TMRobotInfoDetailRoute | Rabbit MQ Server Routing Key TM Robot Detail Information     |
| USERINFO.UserName                | Robot API User Name (customize)                              |
| USERINFO.Password                | Robot API User Password (customize)                          |
| RobotFilePath                    | Download Upload File Folder                                  |
| RobotDataMonitorTime             | Data Monitoring Time (Send to client side data)              |
| RobotDataUpdateTime              | TM Robot Data Updated Time (Update real-time data in memory) |
| ApiService                       | * API Service Host IP Address                                |
| SslEnable                        | Enable SSL Certificate                                       |
| SslpfxPath                       | SSL Certificate Path                                         |
| SslPassword                      | SSL Certificate Password                                     |



### ModbusDetail.json

TM Robot Management API Modbus data configuration files come with `ModbusDetail.json` environment variables. The column descriptions are as follows:

| Column Name   | Description                      |
| ------------- | -------------------------------- |
| startaddress  | Modbus Register Starting Address |
| enable        | Enable or not                    |
| numberofpoint | The Number of Modbus Column      |
| funcitoncode  | Modbus Function Code             |
| datatype      | Data Type                        |

## 4. API Considerations

| #    | API TYPE  | Service Port |
| ---- | --------- | ------------ |
| 1    | Socket    | 9834         |
| 2    | Socket(s) | 9829         |
| 3    | Http      | 9832         |
| 4    | Http(s)   | 9833         |
| 5    | gRPC      | 9831         |

### Socket(s) API

Use the TCP/IP Socket as a channel to access data with the data format in JSON for transmission. EOL  (end of line) is `\n`.

### Http(s) Web API

Use HTTP(S) as the channel to access data with the request header of the authorization access token. Go by  the URL along with the arguments in JSON format to query the respective TM Robot data.

### gRPC API

### RabbitMQ Client

- Use docker-compose.yml to configure in Linux.
- Use `ServiceSettings.json` configure in Windows.


## 5. Basic Functions

### Get Authorization Token

Obtain the API access token by the username/password defined in API Config for authentication when  accessing data with the API.

#### Http(s)

**Arguments**

| Name     | Required | Data Type | Description   |
| -------- | -------- | --------- | ------------- |
| username | Yes      | string    | User name     |
| password | Yes      | string    | User password |


**Request**

URL: `http://{server_ip}:9832/HttpServer/GetToken`

Method: `POST`

Headers

```html
Content-Type: application/json
```

Content

```json
{"username":"Admin","password":"123"}
```

**Response**

```json
{
  "userName": "Admin",
  "password": "123",
  "token": "123456"
}
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use 0 for the  target as a service. |
| ServiceActionParameter |          | object    |                                                              |
| ServiceActionType      | Yes      | int       | Service action type ID. The method brings in the  value `0`. |
| UserName               | Yes      | string    | User name                                                    |
| Password               | Yes      | string    | User password                                                |

**Request**

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

### Search TM Robot Information by IP List

Obtain single or multiple TM Robot information in connection.

#### Http(s)

**Arguments**

| Name        | Required | Data Type | Description                                                  |
| ----------- | -------- | --------- | ------------------------------------------------------------ |
| TMRobotIPs | No       | string array | TM Robot IP addresses. If not filled, it returns every TM Robot in  connection. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotsByIP`

Method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
["192.168.132.70","192.168.132.200"]
```

 **Response**

- Typical success response


```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a TM Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `21`.  |
| TMRobotIPs             | No       | string array | TM Robot IP address list. If the field is null, an  empty array, or no field, it returns every TM  Robot in connection. |

**Request**

```json
{
  "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6IjUxOTQ2NjMzLTM2NTgtNDYzNC1hZTYzLTQzNmRjMDIzMjFkNyIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA3OTA4MDY0LCJleHAiOjE2MDc5MTE2NjQsImlhdCI6MTYwNzkwODA2NCwiaXNzIjoiVE1NIn0.YfbC4sa3ICavXHPCD82r2B3kcUFYRtZ2SOGH_k1tGH8",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 21,
   "TMRobotIPs":["192.168.132.200"]
  }
}

```

**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 0,
    "TMRobotInfos": [
      {
        "TMRobotName": "TM000000",
        "IP": "192.168.132.200",
        "Mode": 2,
        "ControlBoxSN": "BC1924002",
        "Model": "TM5-700ACA",
        "EnabledHMIControl": true,
        "HasRightControl": false,
        "EnabledAutoRemoteMode": false,
        "EnabledSpeedControl": false,
        "TMRobotLIghtColor": 5,
        "TMRobotStatus": 3,
        "ProjectSpeed": 100,
        "HMIVersion": "1.80.5400",
        "DefaultProjectName": "API_Test_1_80",
        "StatusUpdateTime": "2021/4/6 上午 02:56:09",
        "ErrorCode": ""
      }
    ]
  }
}
```

#### grpc

**Arguments**
| Name       | Required | Data Type    | Description                                                  |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| tmrobotips | No       | string array | TM Robot IP address list. If the field is an  empty array, it returns every TM  Robot in connection. |

**Request**

Url:/GrpcServerRobot/GetTMRobotsByIP 

ContentType: json 

Method: Post 

Content: 

```json
{
  "TMRobotIPs": ["192.168.132.70","192.168.132.200"]
}
```
**Response**

```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

### Search TM Robot Information by TM Robot Names

Obtain single or multiple TM Robot information in connection.

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotNames | No       | string array | TM Robot name list. If the field is an empty array, it will return all TM Robot information in connection. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobots`

Method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
["NEX613-20200518","TM000000"]
```

**Response**

```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `0`.   |
| TMRobotNames           | No       | string array | TM Robot name list. If the field is null, an  empty array, or no field, it returns every TM  Robot in connection. |

**Request**

```json
{
  "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6IjUxOTQ2NjMzLTM2NTgtNDYzNC1hZTYzLTQzNmRjMDIzMjFkNyIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA3OTA4MDY0LCJleHAiOjE2MDc5MTE2NjQsImlhdCI6MTYwNzkwODA2NCwiaXNzIjoiVE1NIn0.YfbC4sa3ICavXHPCD82r2B3kcUFYRtZ2SOGH_k1tGH8",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 0,
   "TMRobotNames":["TM000000"]
  }
}
```

**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 0,
    "TMRobotInfos": [
      {
        "TMRobotName": "TM000000",
        "IP": "192.168.132.200",
        "Mode": 2,
        "ControlBoxSN": "BC1924002",
        "Model": "TM5-700ACA",
        "EnabledHMIControl": true,
        "HasRightControl": false,
        "EnabledAutoRemoteMode": false,
        "EnabledSpeedControl": false,
        "TMRobotLIghtColor": 5,
        "TMRobotStatus": 3,
        "ProjectSpeed": 100,
        "HMIVersion": "1.80.5400",
        "DefaultProjectName": "API_Test_1_80",
        "StatusUpdateTime": "2021/4/6 上午 02:56:09",
        "ErrorCode": ""
      }
    ]
  }
}
```

#### grpc

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotNames | No       | string array | TM Robot name list. If the field is an  empty array, it returns every TM  Robot in connection. |

**Request**

Url:/GrpcServerRobot/GetTMRobots 

ContentType: json 

Method: Post 

Content: 

```json
{
  "TMRobotNames": [
    "TM000000",
    "NEX613-20200518"
  ]
}
```

**Response**

```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

### Search Custom  TM Robot Information by IP List

Obtain single or multiple TM Robot  custom information.

> Custom information follow settings' configuration

#### Http(s)

**Arguments**

| Name       | Required | Data Type    | Description                                                  |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotIPs | No       | string array | TM Robot IP addresses. If not filled, it returns every TM Robot in  connection. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotsDetailByIP`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Method: `POST`

Content

```json
["192.168.132.70"]
```

**Response**

- Typical success response

```json
[
  {
    "tmRobotInfo": {
      "tmRobotName": "NEX613-20200518",
      "ip": "192.168.132.70",
      "mode": 2,
      "controlBoxSN": "",
      "model": "TM5-700-CPR1-B",
      "enabledHMIControl": true,
      "hasRightControl": true,
      "enabledAutoRemoteMode": false,
      "enabledSpeedControl": false,
      "tmRobotLIghtColor": 5,
      "tmRobotStatus": 3,
      "projectSpeed": 5,
      "hmiVersion": "1.80.3300",
      "defaultProjectName": "Q123",
      "statusUpdateTime": "2021/4/6 下午 05:03:15",
      "errorCode": ""
    },
    "robotStatus2": {
      "autoRemoteModeEnable": true,
      "autoRemoteModeActive": false,
      "speedAdjustment": false
    },
    "controlBoxDIO": {
      "dI0": false,
      "dI1": false,
      "dI2": false,
      "dI3": false,
      "dI4": false,
      "dI5": false,
      "dI6": false,
      "dI7": false,
      "dI8": false,
      "dI9": false,
      "dI10": false,
      "dI11": false,
      "dI12": false,
      "dI13": false,
      "dI14": false,
      "dI15": false
    },
    "controlBoxAIO": {
      "aO0": -0
    },
    "robotCoordinateCurrentBaseWithoutTool": {
      "xBasewithouttool": -120.41257,
      "yBasewithouttool": -128.21477,
      "zBasewithouttool": 839.75586,
      "rxBasewithouttool": 108.38899,
      "ryBasewithouttool": -21.980446,
      "rzBasewithouttool": -93.81739
    },
    "robotCoordinateTouchStopCurrentBaseWithoutTool": {
      "xBasewithouttool": 0,
      "yBasewithouttool": 0,
      "zBasewithouttool": 0,
      "rxBasewithouttool": 0,
      "ryBasewithouttool": 0,
      "rzBasewithouttool": 0
    },
    "robotCoordinateTouchStopCurrentBaseWithTool": {
      "xBasewithtool": 0,
      "yBasewithtool": 0,
      "zBasewithtool": 0,
      "rxBasewithtool": 0,
      "ryBasewithtool": 0,
      "rzBasewithtool": 0
    },
    "robotCoordinateTouchStopJoint": {
      "joint1": 0,
      "joint2": 0,
      "joint3": 0,
      "joint4": 0,
      "joint5": 0,
      "joint6": 0
    },
    "robotCoordinateTouchStopRobotBaseWithoutTool": {
      "xBasewithtool": 0,
      "yBasewithtool": 0,
      "zBasewithtool": 0,
      "rxBasewithtool": 0,
      "ryBasewithtool": 0,
      "rzBasewithtool": 0
    },
    "robotCoordinateTouchStopRobotBaseWithTool": {
      "xBasewithouttool": 0,
      "yBasewithouttool": 0,
      "zBasewithouttool": 0,
      "rxBasewithouttool": 0,
      "ryBasewithouttool": 0,
      "rzBasewithouttool": 0
    },
    "collaborativeMode": {
      "tcpSpeed": 0.1,
      "tcpSpeedUnderHandGuideMode": 1.5,
      "tcpForce": 130,
      "joint1Speed": 190,
      "joint2Speed": 190,
      "joint3Speed": 190,
      "joint4Speed": 235,
      "joint5Speed": 235,
      "joint6Speed": 235,
      "joint1Torque": 65,
      "joint2Torque": 65,
      "joint3Torque": 65,
      "joint4Torque": 15,
      "joint5Torque": 15,
      "joint6Torque": 15,
      "motionSpeed": 0,
      "ptpSpeed": 0,
      "minimumPossibleContactArea": 0.74,
      "gSensor": false,
      "decelerationTime": 150
    },
    "others1": {
      "currentTime": "2021-4-6 17:25:48",
      "lastErrorTime": "0-0-0 0:0:0",
      "ipcConnectNumber": 48
    }
  }
]
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a TM Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `23`.  |
| TMRobotIPs             | No       | string array | TM Robot IP address list. If the field is null, an  empty array, or no field, it returns every TM  Robot in connection. |

**Request**

```json
{
  "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6IjUxOTQ2NjMzLTM2NTgtNDYzNC1hZTYzLTQzNmRjMDIzMjFkNyIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA3OTA4MDY0LCJleHAiOjE2MDc5MTE2NjQsImlhdCI6MTYwNzkwODA2NCwiaXNzIjoiVE1NIn0.YfbC4sa3ICavXHPCD82r2B3kcUFYRtZ2SOGH_k1tGH8",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 23,
    "TMRobotIPs":["192.168.132.200"]
  }
}

```

**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 22,
    "TMRobotInfosDetail": [
      {
        "TMRobotInfo": {
          "TMRobotName": "TM000000",
          "IP": "192.168.132.200",
          "Mode": 1,
          "ControlBoxSN": "BC1924002",
          "Model": "TM5-700ACA",
          "EnabledHMIControl": true,
          "HasRightControl": true,
          "EnabledAutoRemoteMode": true,
          "EnabledSpeedControl": false,
          "TMRobotLIghtColor": 3,
          "TMRobotStatus": 3,
          "ProjectSpeed": 100,
          "HMIVersion": "1.80.5400",
          "DefaultProjectName": "API_Test_1_80",
          "StatusUpdateTime": "2021/4/6 銝?? 04:26:22",
          "ErrorCode": ""
        },
        "RobotStatus2": {
          "AutoRemoteModeEnable": true,
          "AutoRemoteModeActive": true,
          "SpeedAdjustment": false
        },
        "ControlBoxDIO": {
          "DI0": false,
          "DI1": false,
          "DI2": false,
          "DI3": false,
          "DI4": false,
          "DI5": false,
          "DI6": false,
          "DI7": false,
          "DI8": false,
          "DI9": false,
          "DI10": false,
          "DI11": false,
          "DI12": false,
          "DI13": false,
          "DI14": false,
          "DI15": false
        },
        "ControlBoxAIO": {
          "AO0": -10.0
        },
        "RobotCoordinateCurrentBaseWithoutTool": {
          "XBasewithouttool": 184.00572,
          "YBasewithouttool": -376.85797,
          "ZBasewithouttool": 310.54944,
          "RxBasewithouttool": 177.61002,
          "RyBasewithouttool": -2.0264668,
          "RzBasewithouttool": -8.918276
        },
        "RobotCoordinateTouchStopCurrentBaseWithoutTool": {
          "XBasewithouttool": 0.0,
          "YBasewithouttool": 0.0,
          "ZBasewithouttool": 0.0,
          "RxBasewithouttool": 0.0,
          "RyBasewithouttool": 0.0,
          "RzBasewithouttool": 0.0
        },
        "RobotCoordinateTouchStopCurrentBaseWithTool": {
          "XBasewithtool": 0.0,
          "YBasewithtool": 0.0,
          "ZBasewithtool": 0.0,
          "RxBasewithtool": 0.0,
          "RyBasewithtool": 0.0,
          "RzBasewithtool": 0.0
        },
        "RobotCoordinateTouchStopJoint": {
          "Joint1": 0.0,
          "Joint2": 0.0,
          "Joint3": 0.0,
          "Joint4": 0.0,
          "Joint5": 0.0,
          "Joint6": 0.0
        },
        "RobotCoordinateTouchStopRobotBaseWithoutTool": {
          "XBasewithtool": 0.0,
          "YBasewithtool": 0.0,
          "ZBasewithtool": 0.0,
          "RxBasewithtool": 0.0,
          "RyBasewithtool": 0.0,
          "RzBasewithtool": 0.0
        },
        "RobotCoordinateTouchStopRobotBaseWithTool": {
          "XBasewithouttool": 0.0,
          "YBasewithouttool": 0.0,
          "ZBasewithouttool": 0.0,
          "RxBasewithouttool": 0.0,
          "RyBasewithouttool": 0.0,
          "RzBasewithouttool": 0.0
        },
        "CollaborativeMode": {
          "TCPSpeed": 0.1,
          "TCPSpeedUnderHandGuideMode": 1.5,
          "TCPForce": 130.0,
          "Joint1Speed": 190.0,
          "Joint2Speed": 190.0,
          "Joint3Speed": 190.0,
          "Joint4Speed": 235.0,
          "Joint5Speed": 235.0,
          "Joint6Speed": 235.0,
          "Joint1Torque": 65.0,
          "Joint2Torque": 65.0,
          "Joint3Torque": 65.0,
          "Joint4Torque": 15.0,
          "Joint5Torque": 15.0,
          "Joint6Torque": 15.0,
          "MotionSpeed": 180.0,
          "PTPSpeed": 8.1,
          "MinimumPossibleContactArea": 0.74,
          "GSensor": true,
          "DecelerationTime": 150
        },
        "Others1": {
          "CurrentTime": "2021-4-6 16:30:54",
          "LastErrorTime": "2021-4-6 11:37:56",
          "IPCConnectNumber": 54
        }
      }
    ]
  }
}
```

#### gRPC

**Arguments**
| Name       | Required | Data Type    | Description                                                  |
| ---------- | -------- | ------------ | ------------------------------------------------------------ |
| tmrobotips | No       | string array | TM Robot IP address list. If the field is an empty array, it will return all TM Robot custom information. |

**Request**

Url:/GrpcServerRobot/GetTMRobotsDetailByIP 

```json
{
  "TMRobotIps": [
    "192.168.132.70",
    "192.168.132.200"
  ]
}
```
**Response**

```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

### Search Custom  TM Robot Information by TM Robot Names

Obtain single or multiple TM Robot  custom information.

> Custom information follow settings' configuration

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotNames | No       | string array | TM Robot name(stick serial number, unique value). If the field not filled, it returns every TM Robot in  connection. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotsDetail`

Method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
["TM123456"]
```

**Response**

- Typical success response

```json
[
  {
    "tmRobotInfo": {
      "tmRobotName": "TM123456",
      "ip": "192.168.132.70",
      "mode": 2,
      "controlBoxSN": "",
      "model": "TM5-700-CPR1-B",
      "enabledHMIControl": true,
      "hasRightControl": true,
      "enabledAutoRemoteMode": false,
      "enabledSpeedControl": false,
      "tmRobotLIghtColor": 5,
      "tmRobotStatus": 3,
      "projectSpeed": 5,
      "hmiVersion": "1.80.3300",
      "defaultProjectName": "Q123",
      "statusUpdateTime": "2021/4/6 下午 05:03:15",
      "errorCode": ""
    },
    "robotStatus2": {
      "autoRemoteModeEnable": true,
      "autoRemoteModeActive": false,
      "speedAdjustment": false
    },
    "controlBoxDIO": {
      "dI0": false,
      "dI1": false,
      "dI2": false,
      "dI3": false,
      "dI4": false,
      "dI5": false,
      "dI6": false,
      "dI7": false,
      "dI8": false,
      "dI9": false,
      "dI10": false,
      "dI11": false,
      "dI12": false,
      "dI13": false,
      "dI14": false,
      "dI15": false
    },
    "controlBoxAIO": {
      "aO0": -0
    },
    "robotCoordinateCurrentBaseWithoutTool": {
      "xBasewithouttool": -120.41257,
      "yBasewithouttool": -128.21477,
      "zBasewithouttool": 839.75586,
      "rxBasewithouttool": 108.38899,
      "ryBasewithouttool": -21.980446,
      "rzBasewithouttool": -93.81739
    },
    "robotCoordinateTouchStopCurrentBaseWithoutTool": {
      "xBasewithouttool": 0,
      "yBasewithouttool": 0,
      "zBasewithouttool": 0,
      "rxBasewithouttool": 0,
      "ryBasewithouttool": 0,
      "rzBasewithouttool": 0
    },
    "robotCoordinateTouchStopCurrentBaseWithTool": {
      "xBasewithtool": 0,
      "yBasewithtool": 0,
      "zBasewithtool": 0,
      "rxBasewithtool": 0,
      "ryBasewithtool": 0,
      "rzBasewithtool": 0
    },
    "robotCoordinateTouchStopJoint": {
      "joint1": 0,
      "joint2": 0,
      "joint3": 0,
      "joint4": 0,
      "joint5": 0,
      "joint6": 0
    },
    "robotCoordinateTouchStopRobotBaseWithoutTool": {
      "xBasewithtool": 0,
      "yBasewithtool": 0,
      "zBasewithtool": 0,
      "rxBasewithtool": 0,
      "ryBasewithtool": 0,
      "rzBasewithtool": 0
    },
    "robotCoordinateTouchStopRobotBaseWithTool": {
      "xBasewithouttool": 0,
      "yBasewithouttool": 0,
      "zBasewithouttool": 0,
      "rxBasewithouttool": 0,
      "ryBasewithouttool": 0,
      "rzBasewithouttool": 0
    },
    "collaborativeMode": {
      "tcpSpeed": 0.1,
      "tcpSpeedUnderHandGuideMode": 1.5,
      "tcpForce": 130,
      "joint1Speed": 190,
      "joint2Speed": 190,
      "joint3Speed": 190,
      "joint4Speed": 235,
      "joint5Speed": 235,
      "joint6Speed": 235,
      "joint1Torque": 65,
      "joint2Torque": 65,
      "joint3Torque": 65,
      "joint4Torque": 15,
      "joint5Torque": 15,
      "joint6Torque": 15,
      "motionSpeed": 0,
      "ptpSpeed": 0,
      "minimumPossibleContactArea": 0.74,
      "gSensor": false,
      "decelerationTime": 150
    },
    "others1": {
      "currentTime": "2021-4-6 17:25:48",
      "lastErrorTime": "0-0-0 0:0:0",
      "ipcConnectNumber": 48
    }
  }
]
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a TM Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `22`.  |
| TMRobotNames           | No       | string array | TM Robot name list. If the field is null, an  empty array, or no field, it returns every TM  Robot in connection. |

**Request**

```json
{
  "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6IjUxOTQ2NjMzLTM2NTgtNDYzNC1hZTYzLTQzNmRjMDIzMjFkNyIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA3OTA4MDY0LCJleHAiOjE2MDc5MTE2NjQsImlhdCI6MTYwNzkwODA2NCwiaXNzIjoiVE1NIn0.YfbC4sa3ICavXHPCD82r2B3kcUFYRtZ2SOGH_k1tGH8",
  "ActionTarget":1,
  "TMRobotActionParameter":{
    "RobotActionType":22,
    "TMRobotNames":["TM000000"]
    }
}

```

**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 22,
    "TMRobotInfosDetail": [
      {
        "TMRobotInfo": {
          "TMRobotName": "TM000000",
          "IP": "192.168.132.200",
          "Mode": 1,
          "ControlBoxSN": "BC1924002",
          "Model": "TM5-700ACA",
          "EnabledHMIControl": true,
          "HasRightControl": true,
          "EnabledAutoRemoteMode": true,
          "EnabledSpeedControl": false,
          "TMRobotLIghtColor": 3,
          "TMRobotStatus": 3,
          "ProjectSpeed": 100,
          "HMIVersion": "1.80.5400",
          "DefaultProjectName": "API_Test_1_80",
          "StatusUpdateTime": "2021/4/6 銝?? 04:26:22",
          "ErrorCode": ""
        },
        "RobotStatus2": {
          "AutoRemoteModeEnable": true,
          "AutoRemoteModeActive": true,
          "SpeedAdjustment": false
        },
        "ControlBoxDIO": {
          "DI0": false,
          "DI1": false,
          "DI2": false,
          "DI3": false,
          "DI4": false,
          "DI5": false,
          "DI6": false,
          "DI7": false,
          "DI8": false,
          "DI9": false,
          "DI10": false,
          "DI11": false,
          "DI12": false,
          "DI13": false,
          "DI14": false,
          "DI15": false
        },
        "ControlBoxAIO": {
          "AO0": -10.0
        },
        "RobotCoordinateCurrentBaseWithoutTool": {
          "XBasewithouttool": 184.00572,
          "YBasewithouttool": -376.85797,
          "ZBasewithouttool": 310.54944,
          "RxBasewithouttool": 177.61002,
          "RyBasewithouttool": -2.0264668,
          "RzBasewithouttool": -8.918276
        },
        "RobotCoordinateTouchStopCurrentBaseWithoutTool": {
          "XBasewithouttool": 0.0,
          "YBasewithouttool": 0.0,
          "ZBasewithouttool": 0.0,
          "RxBasewithouttool": 0.0,
          "RyBasewithouttool": 0.0,
          "RzBasewithouttool": 0.0
        },
        "RobotCoordinateTouchStopCurrentBaseWithTool": {
          "XBasewithtool": 0.0,
          "YBasewithtool": 0.0,
          "ZBasewithtool": 0.0,
          "RxBasewithtool": 0.0,
          "RyBasewithtool": 0.0,
          "RzBasewithtool": 0.0
        },
        "RobotCoordinateTouchStopJoint": {
          "Joint1": 0.0,
          "Joint2": 0.0,
          "Joint3": 0.0,
          "Joint4": 0.0,
          "Joint5": 0.0,
          "Joint6": 0.0
        },
        "RobotCoordinateTouchStopRobotBaseWithoutTool": {
          "XBasewithtool": 0.0,
          "YBasewithtool": 0.0,
          "ZBasewithtool": 0.0,
          "RxBasewithtool": 0.0,
          "RyBasewithtool": 0.0,
          "RzBasewithtool": 0.0
        },
        "RobotCoordinateTouchStopRobotBaseWithTool": {
          "XBasewithouttool": 0.0,
          "YBasewithouttool": 0.0,
          "ZBasewithouttool": 0.0,
          "RxBasewithouttool": 0.0,
          "RyBasewithouttool": 0.0,
          "RzBasewithouttool": 0.0
        },
        "CollaborativeMode": {
          "TCPSpeed": 0.1,
          "TCPSpeedUnderHandGuideMode": 1.5,
          "TCPForce": 130.0,
          "Joint1Speed": 190.0,
          "Joint2Speed": 190.0,
          "Joint3Speed": 190.0,
          "Joint4Speed": 235.0,
          "Joint5Speed": 235.0,
          "Joint6Speed": 235.0,
          "Joint1Torque": 65.0,
          "Joint2Torque": 65.0,
          "Joint3Torque": 65.0,
          "Joint4Torque": 15.0,
          "Joint5Torque": 15.0,
          "Joint6Torque": 15.0,
          "MotionSpeed": 180.0,
          "PTPSpeed": 8.1,
          "MinimumPossibleContactArea": 0.74,
          "GSensor": true,
          "DecelerationTime": 150
        },
        "Others1": {
          "CurrentTime": "2021-4-6 16:30:54",
          "LastErrorTime": "2021-4-6 11:37:56",
          "IPCConnectNumber": 54
        }
      }
    ]
  }
}
```

#### gRPC

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotNames | No       | string array | TM Robot name list. If the field is an empty array, it will return all TM Robot custom information. |

**Request**

Url:/GrpcServerRobot/GetTMRobotsDetail

```json
{
  "TMRobotNames": [
    "TM000000",
    "NEX613-20200518"
  ]
}
```
**Response**

```json
[
  {
    "tmRobotName": "NEX613-20200518",
    "ip": "192.168.132.70",
    "mode": 2,
    "controlBoxSN": "",
    "model": "TM5-700-CPR1-B",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": false,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 5,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.3300",
    "defaultProjectName": "Q123",
    "statusUpdateTime": "2021/4/6 下午 05:03:15",
    "errorCode": ""
  },
  {
    "tmRobotName": "TM000000",
    "ip": "192.168.132.200",
    "mode": 1,
    "controlBoxSN": "BC1924002",
    "model": "TM5-700ACA",
    "enabledHMIControl": true,
    "hasRightControl": true,
    "enabledAutoRemoteMode": true,
    "enabledSpeedControl": false,
    "tmRobotLIghtColor": 3,
    "tmRobotStatus": 3,
    "projectSpeed": 5,
    "hmiVersion": "1.80.5400",
    "defaultProjectName": "000_0",
    "statusUpdateTime": "2021/4/6 下午 05:08:57",
    "errorCode": ""
  }
]
```

### Search TM Robot Project Name

Check and obtain the project name list of the TM Robot in connection.

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotName  | Yes      | string       | TM Robot name (stick serial number, unique value).           |
| ProjectNames | Yes      | string array | TM Robot project name list. If the field is an empty array, it will returns all project names of TM Robot. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotProjects/{TMRobotName}`

Method: `POST`

Content

```json
["000_0","0704_700_vision"]
```

**Response**

- Typical success response

```json
[
  {"tmRobotName":"TM000000","projectName":"000_0"},
  {"tmRobotName":"TM000000","projectName":"0704_700_vision"}
]
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a TM Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `1`.   |
| TMRobotName            | Yes      | string       | TM Robot name(stick serial number, unique value).            |
| ProjectNames           | No       | string array | TM Robot project name list. If the field is null, an  empty array, or no field, it returns TM  Robot's all project names. |

**Request**

 ```json
{
  "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6IjUxOTQ2NjMzLTM2NTgtNDYzNC1hZTYzLTQzNmRjMDIzMjFkNyIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA3OTA4MDY0LCJleHAiOjE2MDc5MTE2NjQsImlhdCI6MTYwNzkwODA2NCwiaXNzIjoiVE1NIn0.YfbC4sa3ICavXHPCD82r2B3kcUFYRtZ2SOGH_k1tGH8",
  "ActionTarget": 1,
  "TMRobotActionParameter": {
    "RobotActionType": 1,
   "TMRobotName":"TM000000",
   "ProjectNames":["000_0","0704_700_vision"]
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
        "TMRobotName": "TM000000",
        "ProjectName": "000_0"
      },
      {
        "TMRobotName": "TM000000",
        "ProjectName": "0704_700_vision"
      }
    ]
}
```

#### gRPC

**Arguments**

| Name         | Required | Data Type    | Description                                                  |
| ------------ | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotName  | Yes      | string       | TM Robot name(stick serial number, unique value).            |
| ProjectNames | No       | string array | TM Robot project name list. If the field is an  empty array, it returns TM  Robot's all project names. |

**Request**

Url:/GrpcServerRobot/GetTMRobotProjects

```json
{
    "TMRobotName":"TM000000",
    "ProjectNames":["000_0","0704_700_vision"]
}
```
**Response**

```json
[
  {"tmRobotName":"TM000000","projectName":"000_0"},
  {"tmRobotName":"TM000000","projectName":"0704_700_vision"}
]
```

### Search TM Robot Variable

Obtain the variable property information in connection. The variables include global variables and  project variables of the TM Robot in operation.

#### Http(s)

| Name    | Required | Data Type | Description      |
| ----------- | -------- | -------- | ------------------ |
| TMRobotName | Yes    | string | TM Robot names((stick serial number, unique value)). |
| ProjectName | Yes  | string | Project name |
| VariableNames | No | string array | TM Robot variable name list. If the field is an  empty array, it returns TM  Robot's all project names. |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotVariable/{ TMRobotName }/{ProjectName}`

HTTP method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
["var_a"]
```

**Response**

- Typical success response

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

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as a TM Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `2`.   |
| TMRobotName            | Yes      | string       | TM Robot name                                                |
| VariableNames          | No       | string array | Variable name list. If the field is null, an  empty array, or no field, it returns every TM  Robot in connection. |

**Request**

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

#### gRPC

**Arguments**

| Name          | Required | Data Type    | Description                                                  |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotName   | Yes      | string       | TM Robot names((stick serial number, unique value)).         |
| ProjectName   | Yes      | string       | Project name                                                 |
| VariableNames | No       | string array | TM Robot variable name list. If the field is an  empty array, it returns TM  Robot's all variables. |

**Request**

Url:/GrpcServerRobot/GetTMRobotVariables

```json
{
    "TMRobotName":"TM000000",
    "ProjectName":"Test",
    "VariableNames":["var_a"]
}
```
**Response**

```json
[
  {
    "name": "var_a",
    "projectName": "Test",
    "isGlobalVariable": true,
  }
]
```

### Search TM Robot Variable Value

Obtain one or more variable values of the TM Robot in connection. The content contains the current  and the last value.

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description                                        |
| ------------ | -------- | ------------ | -------------------------------------------------- |
| TMRobotName  | Yes      | string       | TM Robot names(stick serial number, unique value). |
| VariableName | No       | string array | Variable name                                      |

**Request**

URL: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotVariablesValue/{TMRobotName}`


Method:  `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
["var_a"]
```

**Response**

- Typical success response

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

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for  the target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `3`.   |
| TMRobotName            | Yes      | string       | TM Robot name                                                |
| VariableNames          | No       | string array | Variable name list. If the field is null, an empty array, or no field,  it returns all project variables. |

**Request**

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

#### gRPC

**Arguments**
| Name          | Required | Data Type    | Description                                                  |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| TMRobotName   | Yes      | string       | TM Robot name                                                |
| VariableNames | No       | string array | Variable name list. If the field is an empty array,  it returns all project variables. |

**Request**

Url:/GrpcServerRobot/GetTMRobotVariablesValue

```json
{
    "variableNames":["var_a"]
}
```
**Response**

```json
[
    {
        "TMRobotName": "TM123456",
        "ProjectName": "Test ",
        "vType": "string",
        "vName": "var_a",
        "Value": "Test",
        "PastValue": ""
      }
]
```

### Read Modbus

Obtain the TM Robot Modbus Address value. For TM Robot Modbus Table, refer to the respective  version of TMflow user manual.

#### Http(s)

**Arguments**

| Name       | Required | Data Type | Description |
| -------------- | -------- | -------- | ------------ |
| ModbusFunction        | Yes    | int      | Modbus Read Function Code, refer to [Field Specification](#modbusfunction). |
| DeviceIP              | Yes    | string   | TM Robot or other device IP address. |
| Port                  | Yes    | int      | Modbus port 502          |
| SlaveAddress          | Yes    | int      | Modbus slave id         |
| StartAddress          | Yes    | int      | Modbus register starting address |
| NumberOfPoint         | Yes    | int      | Modbus address amount to read |

**Request**

URL: `http://{server_ip}:9832/HttpServerModbus/ReadModbus`


Method: `POST`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
{
  "ModbusFunction": 1, //  Refer to the second part of the Response for the response result of 3 or 4.
  "DeviceIP": "192.168.132.169",
  "Port": 502,
  "SlaveAddress": 1,
  "StartAddress": 0,
  "NumberOfPoint": 3
}
```

**Response**

- Request content is 1, Typical success response

```json
[false,false,false]
```

- Request content is 3 or 4, typical success respose.

```json
[15547,62248,33374]
```

#### Socket(s)

**Arguments**

| Name                  | Required | Data Type | Description                                                  |
| --------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                 | Yes      | string    | API access token                                             |
| ActionTarget          | Yes      | int       | Socket command target in effect. Use `2` for the  target as Modbus. |
| ModbusActionParameter |          | object    |                                                              |
| ModbusFunction        | Yes      | int       | Refer to 11. Field Specification for Modbus  function codes. |
| DeviceIP              | Yes      | string    | Device IP address                                            |
| Port                  | Yes      | int       | Modbus port 502                                              |
| SlaveAddress          | Yes      | int       | Modbus slave id                                              |
| StartAddress          | Yes      | int       | Modbus register starting address                             |
| NumberOfPoint         | Yes      | int       | Modbus address amount to read                                |

```json
{
  "Token": "123456",
  "ActionTarget": 2,
  "ModbusActionParameter": {
    "ModbusFunction": 1,  //  Refer to the second part of the Response for the response result of 3 or 4.
    "DeviceIP": "192.168.132.169",
    "Port": 502,
    "SlaveAddress": 1,
    "StartAddress": 0,
    "NumberOfPoint": 3
  }
}

```

**Response**

- Request content is 1, Typical success response

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
- Request content is 3 or 4, typical success respose.

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 2,
  "ModbusActionReturn": {
    "ModbusFunction": 3,
    "ReadRegisters": [
       15547,62248,33374
    ]
  }
}
```

#### grpc

**Arguments**

| Name           | Required | Data Type | Description                                                  |
| -------------- | -------- | --------- | ------------------------------------------------------------ |
| ModbusFunction | Yes      | int       | Modbus Read Function Code, refer to [Field Specification](#modbusfunction). |
| DeviceIP       | Yes      | string    | TM Robot or other device IP address.                         |
| Port           | Yes      | int       | Modbus port 502                                              |
| SlaveAddress   | Yes      | int       | Modbus slave id                                              |
| StartAddress   | Yes      | int       | Modbus register starting address                             |
| NumberOfPoint  | Yes      | int       | Modbus address amount to read                                |

**Request**

Url:/GrpcServerRobot/ReadModbus

```json
{
  "ModbusFunction": 1, //  Refer to the second part of the Response for the response result of 3 or 4.
  "DeviceIP": "192.168.132.169",
  "Port": 502,
  "SlaveAddress": 1,
  "StartReadAddress": 0,
  "NumberOfPoint": 3
}
```
**Response**

- Request content is 1, Typical success response

```json
{
    "ReadBools": [
      false,
      false,
      false
    ]
}
```
- Request content is 3 or 4, typical success respose.

```json
{
    "ReadRegisters": [
       15547,62248,33374
    ]
}
```

### Subscribe TM Robot Information

Subscribe TM Robot information by the contents of the input arguments. When the connected TM  Robot information changes, the system actively publishes TM Robot information to subscribers. For TM  Robot information, refer to 8.4 TMRobotInfo.

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                  | Yes      | string    | API access token                                             |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object    |                                                              |
| RobotActionType        | Yes      | int       | Robot action type ID. The method brings in the  value `4`.   |
| TMRobotName            | Yes      | string    | TM Robot name                                                |

**Request**

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

Unsubscribe TM Robot information by the contents of the input arguments.

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for  the target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `5`.   |
| TMRobotNames           | Yes      | string array | TM Robot names                                               |

**Request**

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

Response

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

Subscribe TM Robot variables by the contents of the input arguments. When the connected TM Robot  variables change, the system actively publishes TM Robot information to subscribers.

#### Socket(s) 

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for  the target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in  the value `6`.   |
| TMRobotName            | Yes      | string       | TM Robot name                                                |
| VariableNames          | Yes      | string array | Variable list. If the field is null, an empty  array, or no field, it subscribes all variables in  the TM Robot. |

**Request**

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

Unsubscribe TM Robot variables by the contents of the input arguments.

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type ID. The method brings in the  value `7`.   |
| TMRobotName            | Yes      | string       | TM Robot name                                                |
| ProjectName            | Yes      | string       | Project name                                                 |
| VariableNames          | Yes      | string array | Variable list. If the field is null, an empty array, or no field, it unsubscribes all variables in the  TM Robot. |

**Request**

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

## 6. Advance Functions

### Get API Client Control of TM Robot

The logged-in user gets  API client control of TM Robot. Ensure that a single TM Robot can only be controlled by one API Client at the same time.

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description        |
| ------------ | -------- | ------------ | ------------------ |
| TMRobotNames | No       | string array | TM Robot name list |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/GetTMRobotControl`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content:

```json
["NEX613-20200518","TM000000"]
```
**Response**

```json
{
  "MultipleResult":{
    "NEX613-20200518":{
      "success":true,
      "message":""
      },
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type id. The method brings in the  value `8`.   |
| TMRobotNames           | Yes      | string array | TM Robot name list                                           |

**Request**

```json
{
   "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":8,
      "TMRobotNames":[
         "TM000000",
         "NEX613-20200518"
      ]
   }
}
```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 8,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      },
      "NEX613-20200518": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name         | Required | Data Type    | Description        |
| ------------ | -------- | ------------ | ------------------ |
| TMRobotNames | No       | string array | TM Robot name list |

**Request**

Url:/GrpcServerRobot/GetTMRobotControl

```json
{
  "TMRobotNames": [
    "NEX613-20200518",
    "TM000000"
  ]
}

```
**Response**

```json
{
  "MultipleResult":{
    "NEX613-20200518":{
      "success":true,
      "message":""
      },
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Release API Client Control of TM Robot

The logged-in user releases API client control of TM Robot.

#### Http(s)

**Arguments**

| Name         | Required | Data Type    | Description        |
| ------------ | -------- | ------------ | ------------------ |
| TMRobotNames | Yes      | string array | TM Robot name list |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/ReleaseTMRobotControl`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content:

```json
["NEX613-20200518","TM000000"]
```

**Response**

```json
{
  "NEX613-20200518": {
    "success": true,
    "message": ""
  },
  "TM000000": {
    "success": true,
    "message": ""
  }
}
```
#### Socket(s)

**Arguments**

| Name                   | Required | Data Type    | Description                                                  |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Token                  | Yes      | string       | API access token                                             |
| ActionTarget           | Yes      | int          | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object       |                                                              |
| RobotActionType        | Yes      | int          | Robot action type id. The method brings in the  value `9`.   |
| TMRobotNames           | No       | string array | TM Robot name list；If the field is null, an empty  array, or no field, it release all TM Robot in connection. |

**Request**

```json
{
  "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":9,
      "TMRobotNames":[
         "TM000000"
      ]
   }
}

```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 9,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name         | Required | Data Type    | Description        |
| ------------ | -------- | ------------ | ------------------ |
| TMRobotNames | Yes      | string array | TM Robot name list |

**Request**

Url:/GrpcServerRobot/ReleaseTMRobotControl

```json
{
  "TMRobotNames": [
    "NEX613-20200518",
    "TM000000"
  ]
}
```
**Response**

```json
{
  "MultipleResult":{
    "NEX613-20200518":{
      "success":true,
      "message":""
      },
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Change TM Robot Default Project

Required

- Get API Client Control of TM Robot.
- TM Robot must be in `Auto mode`.

#### Http(s)

**Arguments**

| Name                    | Required | Data Type  | Description                               |
| ----------------------- | -------- | ---------- | ----------------------------------------- |
| TMRobotName:projectName | Yes      | dictionary | TM Robot name correspond to project name. |

**Request**

Url:`http://{server_ip}:9832/HttpServerRobot/ChangeRobotDefaultProject`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content:

```json
{
  "TM000000":"000_0000",
  "NEX613-20200518":"Test"
}
```
**Response**

```json
{
  "TM000000": {
    "success": true,
    "message": ""
  },
  "NEX613-20200518": {
    "success": false,
    "message": " NEX613-20200518 :you can do this action under AutoRemoteMode"
  }
}
```
#### Socket(s)

**Arguments**

| Name                    | Required | Data Type  | Description                                                  |
| ----------------------- | -------- | ---------- | ------------------------------------------------------------ |
| Token                   | Yes      | string     | API access token                                             |
| ActionTarget            | Yes      | int        | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter  |          | object     |                                                              |
| RobotActionType         | Yes      | int        | Robot action type ID. The method brings in the  value `10`.  |
| TMRobotName:projectName | Yes      | dictionary | TM Robot name correspond to project name.                    |

**Request**

```json
{    "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":10,
      "RobotProjectMap":{
         "TM000000":"000_0"
      }
   }
}

```

**Response**
```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 10,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name                    | Required | Data Type  | Description                               |
| ----------------------- | -------- | ---------- | ----------------------------------------- |
| TMRobotName:projectName | Yes      | dictionary | TM Robot name correspond to project name. |

**Request**

Url:/GrpcServerRobot/ChangeRobotDefaultProject

```json
{
  "RobotProjectMap": {
    "TM000000": "000_0000",
    "NEX613-20200518": "Test"
  }
}
```
**Response**

```json
{
  "MultipleResult":{
    "NEX613-20200518":{
      "success":true,
      "message":""
      },
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Change TM Robot Stick Execution

Required

- Get API Client Control of TM Robot.
- TM Robot HMI version 1.72, 1.76  must be in `Auto mode`.
- TM Robot HMI version 1.80, 1.82  must be in `AutoRemote mode`.

#### Http(s)

**Arguments**

| Name                        | Required | Data Type  | Description                                                  |
| --------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| TMRobotName:TMRobotExcution | Yes      | dictionary | TM Robot name correspond to execution code(refer to Field Specification). |

**Request**

Url:`http://{server_ip}:9832/HttpServerRobot/ChangeRobotExecution`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

Content

```json
{
  "TM000000":0,
  "NEX613-20200518":0
}
```
**Response**
```json
{
  "TM000000": {
    "success": true,
    "message": ""
  },
  "NEX613-20200518": {
    "success": false,
    "message": "NEX613-20200518 not connect(Modbus)"
  }
}
```

#### Socket(s)

**Arguments**

| Name                        | Required | Data Type  | Description                                                  |
| --------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| Token                       | Yes      | string     | API access token                                             |
| ActionTarget                | Yes      | int        | Socket command target in effect. Use `1` for the  target as TM Robot. |
| TMRobotActionParameter      |          | object     |                                                              |
| RobotActionType             | Yes      | int        | Robot action type ID. The method brings in the  value `11`.  |
| TMRobotName:TMRobotExcution | Yes      | dictionary | TM Robot name correspond to execution code(refer to Field Specification). |
**Request**
```json
{
   "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":11,
      "RobotExcutionMap":{
         "TM000000":2,
         "NEX613-20200518":0
      }
   }
}

```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 11,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      },
      "NEX613-20200518": {
        "Success": false,
        "Message": " NEX613-20200518 :you can do this action under AutoRemoteMode"
      }
    }
  }
}
```

#### gRPC

**Arguments**

| Name                        | Required | Data Type  | Description                                                  |
| --------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| TMRobotName:TMRobotExcution | Yes      | dictionary | TM Robot name correspond to execution code(refer to Field Specification). |

**Request**

Url:/GrpcServerRobot/ChangeRobotExecution

```json
{
  "RobotProjectMap": {
    "TM000000": 0,
    "NEX613-20200518": 0
  }
}
```
**Response**

```json
{
  "MultipleResult":{
    "NEX613-20200518":{
      "success":true,
      "message":""
      },
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Set TM Robot Variable Value

Required

- Get API Client Control of TM Robot.

#### Http(s)

**Arguments**

| Name         | Required | Data Type | Description    |
| ------------ | -------- | --------- | -------------- |
| TMRobotName  | Yes      | string    | TM Robot name  |
| ProjectName  | Yes      | string    | Project name   |
| VariableName | Yes      | string    | Variable name  |
| Value        | Yes      | string    | Variable value |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/SetRobotVariable/{tmRobotName}/{projectName}/{variableName}/{value}`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```


#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                  | Yes      | string    | API access token                                             |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object    |                                                              |
| RobotActionType        | Yes      | int       | Robot action type ID. The method brings in the  value `12`.  |
| TMRobotName            | Yes      | string    | TM Robot name                                                |
| ProjectName            | Yes      | string    | Project name                                                 |
| VariableName           | Yes      | string    | Variable name                                                |
| Value                  | Yes      | string    | Variable value                                               |

**Request**
```json
{
   "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":12,
      "TMRobotName":"TM000000",
      "ProjectName":"API_Test_1_80",
      "VariableName":"var_qty1",
      "Value":"0"
   }
}

```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 12,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name         | Required | Data Type | Description    |
| ------------ | -------- | --------- | -------------- |
| TMRobotName  | Yes      | string    | TM Robot name  |
| ProjectName  | Yes      | string    | Project name   |
| VariableName | Yes      | string    | Variable name  |
| Value        | Yes      | string    | Variable value |

**Request**

Url:/GrpcServerRobot/SetRobotVariable

```json
{
  "TMRobotName": "TM000000",
  "ProjectName": "000_0",
  "VariableName": "test",
  "Value": "1"
}
```
**Response**

```json
{
  "MultipleResult":{
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Pull TM Robot Project into API Host

Default TM Robot project will be download to the folder:  `apiserver/Files/(TMRobotName)`, the name of the Files (default) folder can be changed in the Settings.json configuration.

Required

- Get API Client Control of TM Robot.
- TM Robot must be in `Manual mode`.
- TM Robot must be stopped.

#### Http(s)

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/pullproject/{TMRobotName}/{projectName}`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

**Response**

```json
{
  "TM000000": {
    "success": true,
    "message": ""
  }
}
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                  | Yes      | string    | API access token                                             |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object    |                                                              |
| RobotActionType        | Yes      | int       | Robot action type id. The method brings in the  value `13`.  |
| TMRobotName            | Yes      | string    | TM Robot name                                                |
| ProjectName            | Yes      | string    | Project name                                                 |


**Request**
```json
{   "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":13,
      "TMRobotName":"NEX613-20200518",
      "ProjectName":"Test"
   }
}

```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 13,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url:/GrpcServerRobot/pullproject

```json
{
  "TMRobotName": "TM000000",
  "ProjectName": "000_0"
}
```
**Response**

```json
{
  "MultipleResult":{
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Download TM Robot Project from API Host

User can download TM Robot project from API host into client.

Required

- TM Robot project must be downloaded to the API host first.

#### Http(s)

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/DownloadProject/{TMRobotName}/{projectName}`

Method: `Post`

Headers

```http
Content-Type: application/zip
Authorization: Bearer {token}
```

**Response**

TM Robot project `.zip`  file.

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type  | Description                                                  |
| ---------------------- | -------- | ---------- | ------------------------------------------------------------ |
| Token                  | Yes      | string     | API access token                                             |
| ActionTarget           | Yes      | int        | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object     |                                                              |
| RobotActionType        | Yes      | int        | Robot action type ID. The method brings in the  value `14`.  |
| TMRobotProjectFileInfo | Yes      | object     |                                                              |
| ProjectName            | Yes      | string     | Project name                                                 |
| FileSize               | Yes      | string     | Project file size                                            |
| Content                | Yes      | byte array | Project content                                              |
| CheckSum               | Yes      | string     | File check code. (MD5, remove '-' and lower case)            |
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 14,
    "TMRobotProjectFileInfo": {
      "projectName": "Test",
      "fileSize": 100,
      "content": "project content",
      "checkSum": "0bcafe5e30ad8c87aeb38936d6a5f3c4"
    }
  }
}
```

#### gRPC

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url:/GrpcServerRobot/DownloadProject

```json
{
  "TMRobotName": "TM000000",
  "ProjectName": "000_0"
}
```
**Response**

```json
[
  {
    "projectName": "Test",
    "fileSize": 100,
    "content": "project content",
    "checkSum": "0bcafe5e30ad8c87aeb38936d6a5f3c4"
  }
]
```
### Push TM Robot Project to TM Robot

User can upload TM Robot project from API server host into TM Robot.

Required

- Only the project owned by the API server host can be uploaded.

#### Http(s)

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/pushproject/{TMRobotName}/{projectName}`

Method: `Post`

Headers

```http
Content-Type: application/json
Authorization: Bearer {token}
```

**Response**

```json
{
  "TM000000": {
    "success": true,
    "message": ""
  }
}
```

#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                  | Yes      | string    | API access token                                             |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object    |                                                              |
| RobotActionType        | Yes      | int       | Robot action type ID. The method brings in the  value `15`.  |
| TMRobotName            | Yes      | string    | TM Robot name                                                |
| ProjectName            | Yes      | string    | Project name                                                 |

**Request**
```json
{   "Token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBZG1pbiIsImp0aSI6ImQ4NjBhYjMzLTdmZTEtNDFmZi1iNDE1LTE3MTI4NzYxYzViYiIsInJvbGVzIjpbIkFkbWluIiwiVXNlcnMiXSwibmJmIjoxNjA1MDg5NzM5LCJleHAiOjE2MDUwOTMzMzksImlhdCI6MTYwNTA4OTczOSwiaXNzIjoiVE1NIn0.PL0LfipqGfbHP3Y_wucbXSrhUNL3VgJH4boWeeUrn6M",
   "ActionTarget":1,
   "TMRobotActionParameter":{
      "RobotActionType":15,
      "TMRobotName":"NEX613-20200518",
      "ProjectName":"Test"
   }
}

```
**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 15,
    "MultipleResult": {
      "TM000000": {
        "Success": true,
        "Message": ""
      }
    }
  }
}
```
#### gRPC

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url:/GrpcServerRobot/pushproject

```json
{
  "TMRobotName": "TM000000",
  "ProjectName": "000_0"
}
```
**Response**

```json
{
  "MultipleResult":{
      "TM000000":{
        "success":true,
        "message":""
      }
}
```
### Upload TM Robot Project to API Host
#### Http(s)

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url: `http://{server_ip}:9832/HttpServerRobot/UploadProject/{TMRobotName}/{projectName}`

Method: `Post`

Headers

```http
Content-Type: application/zip
Authorization: Bearer {token}
```

Content: FileStream content

**Response**

HttpStatus 200
#### Socket(s)

**Arguments**

| Name                   | Required | Data Type | Description                                                  |
| ---------------------- | -------- | --------- | ------------------------------------------------------------ |
| Token                  | Yes      | string    | API access token                                             |
| ActionTarget           | Yes      | int       | Socket command target in effect. Use `1` for the  target as the Robot. |
| TMRobotActionParameter |          | object    |                                                              |
| RobotActionType        | Yes      | int       | Robot action type ID. The method brings in the  value `16`.  |
| TMRobotName            | Yes      | string    | TM Robot name                                                |
| ProjectName            | Yes      | string    | Project name                                                 |

**Response**

```json
{
  "Success": true,
  "Message": "",
  "ActionTarget": 1,
  "TMRobotActionReturn": {
    "RobotActionType": 16,
  }
}
```
#### gRPC

**Arguments**

| Name        | Required | Data Type | Description   |
| ----------- | -------- | --------- | ------------- |
| TMRobotName | Yes      | string    | TM Robot name |
| ProjectName | Yes      | string    | Project name  |

**Request**

Url:/GrpcServerRobot/UploadProject

```json
[
  {
    "projectName": "Test",
    "fileSize": 100,
    "content": "TM Robot project fileStream content",
    "checkSum": "0bcafe5e30ad8c87aeb38936d6a5f3c4"
  }
]
```
**Response**

```json
{
    "success": true,
    "messege": ""
}
```

## Appendix

### Socket Response

| Name                | Data Type | Description                                                  |
| ------------------- | --------- | ------------------------------------------------------------ |
| Success             | bool      | Result of the successful request                             |
| Message             | string    | Message content of the failed request. Empty for the successful  request. |
| ActionTarget        | int       | Socket API Method target in effect. 請參考 [File Specification](#actiontarget) |
| ServiceActionReturn | object    | Service action object returned                               |
| TMRobotActionReturn | object    | Robot action object returned                                 |
| ModbusActionReturn  | object    | Modbus action object returned                                |

### ServiceActionReturn

| Name              | Data Type | Description                                                  |
| ----------------- | --------- | ------------------------------------------------------------ |
| ServiceActionType | string    | Service action type.，請參考 [File Specification](#serviceactiontype) |
| Token             | string    | API access token                                             |

### TMRobotActionReturn

| Name              | Data Type | Description         |
| --------------------- | ------------ | ------------------------- |
| RobotActionType       | int          | Socket API Robot function in effect. 請參考 [File Specification](#robotactiontype) |
| TMRobotInfos          | array object | TM Robot information object list                             |
| TMRobotProjects | array object | TM Robot project object list |
| TMRobotVariables | array object | TM Robot variable property object list |
| TMRobotVariableValues | array object | TM Robot variable value object list |


### TMRobotInfo

| Name | Data Type | Description |
| --------------------- | -------- | ---------------------------------------- |
| TMRobotName | string | TM Robot name (stick serial number, unique value) |
| IP | string | TM Robot IP address |
| ControlBoxSN | string | Control box serial number (unique value) |
|Mode |int | Running mode (1: AutoMode 2:ManulMode) |
| Model | string | TM Robot model |
| EnabledHMIControl | bool | Is HMI login and get control permission or not |
| EnabledAutoRemoteMode | bool | Is HMI enabled Auto Remote Mode or not |
| EnabledSpeedControl | bool | Is HMI Enabled Speed Control or not |
| HmiVersion | string | HMI Version |
| HasRightControl | bool | Is user get access control right or not |
| TMRobotLightColor | int | TM Robot light indicator code. Refer to Field  Specification. |
| TMRobotStatus | int | TM Robot status code. Refer to Field  Specification. |
| StatusUpdateTime | datetime | TM Robot status update time |
| ProjectSpeed | int | TM Robot speed(%) |
| DefaultProjectName | string | Default project name |
| ErrorCode | string | Error code. For description, refer to TMflow appendix  description |

### TMRobotProject

| Name       | Data Type | Description    |
| -------------- | ------------ | -------------------- |
| TMRobotName | string | TM Robot name (stick serial number, unique value) |
| ProjectName | bool         | TM Robot project name |

### TMRobotVariable

| Name       | Data Type | Description    |
| -------------- | ------------ | -------------------- |
| Name          | string       | TM Robot variable name |
| ProjectName | string   | Project name |
| IsGlobalVariable | bool | 是否為全域變數 |
| IsSubscribed   | bool | 是否已訂閱   |

### TMRobotVariableValue

| Name       | Data Type | Description    |
| -------------- | ------------ | -------------------- |
| TMRobotName | string | TM Robot name (stick serial number, unique value) |
| ProjectName | bool         | Project name      |
| VariableType | string | Variable data type |
| VariableName | string | Variable name |
| VariableValue | string | Variable value |

### TMRobotProjectInfo

| Name        | Data Type  | Description                                      |
| ----------- | ---------- | ------------------------------------------------ |
| ProjectName | string     | Project name                                     |
| FileSize    | int        | File size                                        |
| Content     | byte array | File content                                     |
| CheckSum    | string     | File check code (MD5, remove '-' and lower case) |

### ModbusActionReturn

| Name           | Data Type    | Description                                                  |
| -------------- | ------------ | ------------------------------------------------------------ |
| ModbusFunction | int          | Modbus function code. Refer to Field  Specification.         |
| ReadBools      | bool array   | ReadCoils or ReadInputs for reading multiple values  returned. |
| ReadRegisters  | ushort array | ReadHoldingRegisters or ReadInputRegisters for  reading multiple values returned. |



## Field Specification

### ActionTarget

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 0    | For Socket API to identify the target to command as a service |
| 1    | For Socket API to identify the target to command as TM Robot |
| 2    | For Socket API to identify the target to command as Modbus   |

### ServiceActionType

Service action type ID matches to the Socket API Service function in effect.

| Code | Description           |
| ---- | --------------------- |
| 0    | Obtained access token |

### RobotActionType 

Robot action type ID matches to the Socket API Robot function in effect.

| Code | Description                                     |
| ---- | ----------------------------------------------- |
| 0    | Obtained TM Robot information                   |
| 1    | Obtained TM Robot project information           |
| 2    | Obtained TM Robot variable property information |
| 3    | Obtained TM Robot variable values               |
| 4    | Subscribed TM Robot information                 |
| 5    | Unsubscribed TM Robot information               |
| 6    | Subscribed TM Robot variables                   |
| 7    | Unsubscribed TM Robot variables                 |

### ModbusFunction

| Code | Description          |
| ---- | -------------------- |
| 1    | ReadCoils            |
| 2    | ReadInputs           |
| 3    | ReadHoldingRegisters |
| 4    | ReadInputRegisters   |

### TMRobotLightColor

TM Robot light indicator ID. Refer to Appendix B in the respective version of TMflow user manual for details. 

| Code | Description               |
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

| Code | Description |
| ---- | ----------- |
| 0    | Play        |
| 1    | Error       |
| 2    | Pause       |
| 3    | Stop        |
| 4    | EStop       |

### TMRobotExcution

| Code | Description |
| ---- | ----------- |
| 0    | Play        |
| 1    | Pause       |
| 2    | Stop        |
| 3    | Speed UP    |
| 4    | Speed Down  |

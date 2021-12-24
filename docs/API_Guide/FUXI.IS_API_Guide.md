# FUXI IS CA API文档

FUXI IS（伏羲数据互操作系统）中认证授权层API的说明文档。

API文档的格式主要包括以下几个部分:

- **请求路径**
  该API的请求路径，[请求URL] 加上该请求路径为完整请求路径。如果路径中有大括号扩起来的路径，表示了该部分为路径参数，例如`/app/datatable/{table_name}`中`{table_name}`为路径参数，需要你自己去传入。

- **请求协议**
  目前都是HTTP协议。

- **请求方法**
  HTTP协议的请求方法，包括常用的 `GET, POST, PUT, DELETE PATCH`。 一般来讲：`GET`表示查询获取数据，`POST`表示新增数据或者进行一些逻辑操作，`PATCH  PUT`表示更新数据，`DELETE`表示删除数据。

- **请求头部**
  HTTP请求头部，将该参数放入`HTTP Header`中。

- 请求参数

  请求的参数，常见有`路径参数（Path）、查询参数（Query）、请求体参数（Body`，其中`请求体参数（Body`可以是`表单参数 或 Json参数`

  `GET`请求中只有Path参数和Query参数，`POST PATCH`请求中三种参数都有。

  - Path参数：请求的资源路径紧跟请求域名的后面，服务器通过解析路径参数获取资源位置。路径参数是用`/`分隔的一系列字符串，其意义在于对服务器资源进行统一定位。如果请求的参数是id，在HTTP请求中为`/path/id`。
  - Query参数：是拼接到路径中的参数，用`?`连接，比如路径为`/app/datatable/{table_name}`的请求，有多个`pageSize`和`pageNum`的Query参数，在HTTP中就会拼接到路径后，`/app/datatable/{table_name}?pageSize=10&pageNum=1`
  - 表单参数：表示有HTTP请求体的网络请求，会将参数以Form表单形式放在HTTP请求体中，以`username`和`email`的表单参数为例子，就会在请求体中发送内容为：`username=XXX&email=XXX`
  - Json参数：表示有HTTP请求体的网络请求，会将参数以Json形式放在HTTP请求体中，以`username`和`email`的Json Object参数为例子，就会在请求体中发送内容为：

```
http http://localhost:4444/.well-known/openid-configuration
```

## 用户账号 API

​	用户账号操作相关的API说明。

### 创建Client

**说明**

申请创建一个 client 账号。

用户必须先创建一个Client作为自己的账号，有了账号后用户才能进行其他操作。申请时需要输入自己定义的`client_name、client_secret`参数，还需要输入用户所在组织`organization`，以及用户的`email、mobile`基本信息。

账号创建成功后会返回`client_id`等相关信息。

**请求路径**

```
/clients
```

**HTTP Method**

POST

**请求参数**

JSON 参数

| 参数名        | 类型   | 必填 | 说明         |
| ------------- | ------ | ---- | ------------ |
| client_name   | string | 是   | 用户名       |
| client_secret | string | 是   | 用户密钥     |
| organization  | string |      | 用户所在组织 |
| email         | string |      | 邮箱         |
| mobile        | string |      | 电话         |

**响应结果**



**范例**

```
http POST http://localhost:4445/clients client_secret=abc123 organization=fuxi mobile=10010270253 client_name=fuyufan13 email=fuyufan13@fuxiinstitution.org
```

![image-20211210134927366](FUXI.IS CA API Guide v0.3.assets/image-20211210134927366.png)

```powershell
http POST http://localhost:4445/clients client_secret=abc123 organization=fuxi mobile=20010270253 client_name=zhangyouyi2 email=zhangyouyi2@fuxiinstitution.org
```

### 创建License

**说明**

用户创建一个 license。

为自己的用户账号申请创建一个 license 作用其他操作的 token，创建成功后返回一个 `apiKey`，用户进行其他操作时需要使用 `apiKey `以验证 license。

注：apiKey 需要用户自己保管，一个用户可创建多个 license。

**请求路径**

`POST`

```
/clients/{id}/licenses
```

路径中的 id 为创建用户后返回的 client_id。

**请求参数**

| 参数名        | 类型   | 必填 | 说明                                          | 格式                                 |
| ------------- | ------ | ---- | --------------------------------------------- | ------------------------------------ |
| id            | string | 是   | Path参数                                      |                                      |
| Authorization | string | 是   | Json参数；<br>创建Client时输入的client_secret | Authorization:"bearer client_secret" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为 `id、clientID、apiKey`，`id`是 license 的id<br>用户进行其他操作时，需要使用`apiKey`用于验证 license |
| 400    | 参数错误；client_id 获取用户失败                             |
| 403    | client 身份验证失败                                          |
| 500    | 创建 license 失败                                            |

**示例**

```powershell
http POST http://localhost:4445/clients/did:fuxi:004d0e14fc43558baee34f15d4d0aad1a9e0217ab37ee8963d48cd192d8dbc57/licenses Authorization:"bearer abc123"
```

![image-20211210135416418](FUXI.IS CA API Guide v0.3.assets/image-20211210135416418.png)

```powershell
http POST http://localhost:4445/clients/did:fuxi:b9da7f3d70685270b49d5004e9e9f6997fb5a11ba84371674ee77cba7ec1f2a2/licenses Authorization:"bearer abc123"
```

### 查询License

**说明**

查询用户所有 license 的信息。

输入参数 `client_id` 和 `client_secret`，对该用户账号下的所有 license进行查询。

注：只有本用户才能查询，需要用 client_secret 认证用户身份。

**请求路径**

`GET`

```
/clients/{id}/licenses
```

路径中的 id 为创建用户后返回的 client_id。

**请求参数**

| 参数名        | 类型   | 必填 | 说明                                            | 格式                                 |
| ------------- | ------ | :--: | ----------------------------------------------- | ------------------------------------ |
| client_id     | string |  是  | Path参数                                        |                                      |
| Authorization | string |  是  | Json参数<br>创建 client 时的输入的client_secret | Authorization:"bearer client_secret" |

**响应结果**

| 响应码 | 说明                                                 |
| ------ | :--------------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 用户所有 license 的相关信息 |
| 400    | 参数错误                                             |
| 403    | client 身份验证失败                                  |
| 500    | 查询失败                                             |

**示例**

```powershell
http http://localhost:4445/clients/did:fuxi:004d0e14fc43558baee34f15d4d0aad1a9e0217ab37ee8963d48cd192d8dbc57/licenses Authorization:"bearer abc123"
```

### 查询可用的名字空间

**说明**

查询所有可用的名字空间。

输入参数 `client_id` 和 `apiKey`，对该用户账号下的所有可用的名字空间进行查询，一个名字空间（namespace）对应一个组织（organization），例如 fuxi.id 对应 fuxi 。

注：apiKey 是成功创建 license 的返回值。

**请求路径**

`GET`

```
/clients/{id}/namespaces
```

路径中的 id 为创建用户后返回的 client_id。

**请求参数**

| 参数名        | 类型   | 必填 | 说明                                    | 格式                          |
| ------------- | ------ | :--: | --------------------------------------- | ----------------------------- |
| client_id     | string |  是  | Path参数                                |                               |
| Authorization | string |  是  | Json参数<br>创建 license 后返回的apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                         |
| ------ | :------------------------------------------- |
| 200    | 查询成功；<br>返回内容为所有可用的 namespace |
| 500    | 查询失败                                     |

**示例**

```powershell
http http://localhost:4445/clients/did:fuxi:9299bf2084b3e28a9f63314c12bdc37ac0b0521dc0b5d4cf625016c5084b147d/namespaces Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTowMDRkMGUxNGZjNDM1NThiYWVlMzRmMTVkNGQwYWFkMWE5ZTAyMTdhYjM3ZWU4OTYzZDQ4Y2QxOTJkOGRiYzU3IiwiaXNzIjoiZnV4aSJ9.u1qVNUA03CWoc41niZVAqlii8dU9V6SpimzFw0BjQaw"
```

![image-20211207104623099](FUXI.IS CA API Guide v0.3.assets/image-20211207104623099.png)

```powershell
http http://localhost:4445/clients/did:fuxi:9da7f3d70685270b49d5004e9e9f6997fb5a11ba84371674ee77cba7ec1f2a2/namespaces Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs"
```



## 身份标识 API

### 创建身份标识

**说明**

创建一个身份标识（Identity identifier）。

申请创建一个身份标识。需要使用之前创建 license 后返回的 apiKey 进行身份验证。

注：用户可以为自己创建一个身份标识，拥有身份标识后可以进行数据标识的管理操作以及订阅其他数据。

**请求路径**

`POST`

```
/identity
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名 | 类型   | 必填 | 说明                                              | 格式                                                         |
| ------ | ------ | ---- | ------------------------------------------------- | ------------------------------------------------------------ |
| apiKey | string | 是   | Json参数<br>license 的 apiKey                     | Authorization:"bearer apiKey"                                |
| id     | string | 是   | Json参数；<br>id 自定义，但需要在可用的名字空间里 | domain format<br>cat.fuxi.id<br>其中fuxi.id为可用的namespace |
| name   | string | 是   | Json参数                                          | general name<br>可自定义                                     |
| owner  | string | 否   | Json参数；<br>该身份标识的拥有者                  |                                                              |
| email  | string | 是   | Json参数                                          |                                                              |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为 CreateIdentity，如下所示<br>`PublicKey、PrivateKey`是为该身份标识创建的公私钥对 |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 429    | 创建失败，该身份标识在domain中已存在                         |
| 500    | 创建身份标识失败；创建 domain cert rr 失败                   |

```json
CreateIdentity {
    	Id:               id,
		Name:             name,
		Email:            email,
    	Owner:            owner,
		PublicKey:        PublicKey,
    	PrivateKey:       PrivateKey,
		CreationTime:     time.Time,
		LastModifiedTime: time.Time
}
```

**示例**

```powershell
http POST http://localhost:4444/identity Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTowMDRkMGUxNGZjNDM1NThiYWVlMzRmMTVkNGQwYWFkMWE5ZTAyMTdhYjM3ZWU4OTYzZDQ4Y2QxOTJkOGRiYzU3IiwiaXNzIjoiZnV4aSJ9.u1qVNUA03CWoc41niZVAqlii8dU9V6SpimzFw0BjQaw" id=fvan12.fuxi.id name=fvan12.fuxi.id email=fvan12@fuxiinstitution.org owner=fuxi
```

![image-20211211191311179](FUXI.IS CA API Guide v0.3.assets/image-20211211191311179.png)

```
http POST http://localhost:4444/identity Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs" id=zyy2.fuxi.id name=zyy2 email=zyy2@fuxiinstitution.org 
```

### 查询身份标识

**说明**

根据 ID 查询身份标识。

根据身份标识创建时定义的ID进行查询。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identity/:id
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                              |
| ------ | ------ | ---- | ----------------------------- | --------------------------------- |
| id     | string | 是   | Path参数；<br>身份标识的 ID   | domain format<br>news.cat.fuxi.id |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey"     |

**响应结果**

| 响应码 | 说明                                           |
| ------ | ---------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 GetIdentity，如下所示 |
| 400    | 参数错误                                       |
| 404    | 没有查询到该数据标识                           |
| 500    | 查询失败                                       |

```json
GetIdentity {
    	Id:               id,
		Name:             name,
		Email:            email,
		PublicKey:        PublicKey,
		CreationTime:     time.Time,
		LastModifiedTime: time.Time
}
```

**示例**

```powershell
http GET http://localhost:4444/identity/fvan12.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTowMDRkMGUxNGZjNDM1NThiYWVlMzRmMTVkNGQwYWFkMWE5ZTAyMTdhYjM3ZWU4OTYzZDQ4Y2QxOTJkOGRiYzU3IiwiaXNzIjoiZnV4aSJ9.u1qVNUA03CWoc41niZVAqlii8dU9V6SpimzFw0BjQaw"
```

![image-20211211202942970](FUXI.IS CA API Guide v0.3.assets/image-20211211202942970.png)

```
http GET http://localhost:4444/identity/zyy2.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs"
```

### 删除身份标识

**说明**

根据 ID 删除该身份标识。

根据身份标识的 ID 进行删除。需要使用 apiKey 进行身份验证。

**请求路径**

`DELETE`

```
/identity/:id 
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                          |
| ------ | ------ | ---- | ----------------------------- | ----------------------------- |
| id     | string | 是   | Path参数；<br>身份标识的 ID   | domain format<br>cat.fuxi.id  |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                              |
| ------ | --------------------------------- |
| 204    | 删除成功，返回 No Content 和 Date |
| 400    | 参数错误                          |
| 500    | 删除失败                          |

**示例**

```powershell
http DELETE http://localhost:4444/identity/fvan12.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTowMDRkMGUxNGZjNDM1NThiYWVlMzRmMTVkNGQwYWFkMWE5ZTAyMTdhYjM3ZWU4OTYzZDQ4Y2QxOTJkOGRiYzU3IiwiaXNzIjoiZnV4aSJ9.u1qVNUA03CWoc41niZVAqlii8dU9V6SpimzFw0BjQaw"
```

<img src="FUXI.IS CA API Guide v0.3.assets/image-20211212114510325.png" alt="image-20211212114510325" style="zoom:50%;" />

### 查询某个用户名下的身份标识

**说明**

根据 client_id 查询所有身份标识。

根据client_id ，查询该 client_id 下的所有身份标识。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identity?client_id=?
```

**请求参数**

| 参数名    | 类型   | 必填 | 说明                                 | 格式                          |
| --------- | ------ | ---- | ------------------------------------ | ----------------------------- |
| client_id | string | 是   | Query参数<br>创建 client 时返回的 id |                               |
| apiKey    | string | 是   | Json参数<br>license 的 apiKey        | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 查询成功；<br>返回内容为 GetIdentity list，GetIdentity如下所示 |
| 400    | 参数错误                                                     |
| 404    | 根据 client_id 没有查询到身份标识                            |
| 500    | 查询失败                                                     |

```json
GetIdentity {
    	Id:               id,
		Name:             name,
		Email:            email,
		PublicKey:        PublicKey,
		CreationTime:     time.Time,
		LastModifiedTime: time.Time
}
```

**示例**

```
http GET http://localhost:4444/identity?client_id=did:fuxi:004d0e14fc43558baee34f15d4d0aad1a9e0217ab37ee8963d48cd192d8dbc57  Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTowMDRkMGUxNGZjNDM1NThiYWVlMzRmMTVkNGQwYWFkMWE5ZTAyMTdhYjM3ZWU4OTYzZDQ4Y2QxOTJkOGRiYzU3IiwiaXNzIjoiZnV4aSJ9.u1qVNUA03CWoc41niZVAqlii8dU9V6SpimzFw0BjQaw"
```

![image-20211213153353696](FUXI.IS CA API Guide v0.3.assets/image-20211213153353696.png)

```
http GET http://localhost:4444/identity?client_id=did:fuxi:b9da7f3d70685270b49d5004e9e9f6997fb5a11ba84371674ee77cba7ec1f2a2  Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQL
```

## 数据标识 API

### 创建数据标识

**说明**

创建一个身份标识（Data identifier）。

申请创建一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

创建时需要使用 apiKey 进行身份验证。

**请求路径**

`POST`

```
/identifier
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名           | 类型   | 必填 | 说明                                                         | 格式                                                         |
| ---------------- | ------ | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| apiKey           | string | 是   | Json参数<br>license 的 apiKey                                | Authorization:"bearer apiKey"                                |
| id               | string | 是   | Json参数；<br>id 自定义，但需要在可用的身份标识下            | domain format<br>news.cat.fuxi.id<br>其中cat.fuxi.id是license下对应的身份标识，即该数据标识的拥有者 |
| name             | string | 是   | Json参数                                                     | general name<br>可自定义                                     |
| owner            | string | 是   | Json参数；<br>该数据标识的拥有者，可用身份标识表示           | cat.fuxi.id                                                  |
| AuthAddress      | string | 否   | Json参数；<br>默认为 http://localhost:4444                   | URL                                                          |
| dataAddress      | string | 是   | Json参数；<br>数据标识所表示的数据的存放地址                 | URL<br>"https://example/data/"                               |
| dataDigest       | string | 是   | Json参数；<br>自定义                                         |                                                              |
| subscriptionType | string | 否   | Json参数；<br>默认为 Free                                    |                                                              |
| categoryID       | string | 否   | Json参数；<br/>在已有的分类中选择一个进行填写<br>若不填写，默认为 other |                                                              |
| tags             | 数组   | 否   | Json参数；<br/>数据的 tag，自定义                            | '["News","Home"]'                                            |
| metadata         | map    | 否   | Json参数；<br/>元数据，自定义，提供元数据用于查询            | '{"type":"test", "site":"zhihu"}'                            |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

```json
CreateDataIdentifier {
    	Id:               id,
		Name:             name,
		Email:            email,
    	Owner:            owner,
		AuthAddress:      AuthAddress,
    	dataAddress:      URL,
		dataDigest:       dataDigest,
		subscriptionType: "Free",
    	categoryID:       categoryID,
    	tags: 			  tags,
   		metadata: 		  metadata
}
```

**示例**

```
http POST localhost:4444/identifier Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs" id=news.zyy4.fuxi.id name=zyy4_news owner=zyy4.fuxi.id dataAddress="https://scholars-stage.org/xi-jinpings-war-on-spontaneous-order/" dataDigest="abc" subscriptionType="Free" categoryID="education" tags:='["News","Home"]' metadata:='{"type":"test", "site":"zhihu"}'
```

![image-20211218150226351](FUXI.IS CA API Guide v0.3_12.13.assets/image-20211218150226351.png)



### 查询数据标识

**说明**

根据 ID 查询数据标识。

根据数据标识创建时定义的ID进行查询。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identifier/:id 
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                          |
| ------ | ------ | ---- | ----------------------------- | ----------------------------- |
| id     | string | 是   | Path参数；<br>数据标识的 ID   | domain format<br>cat.fuxi.id  |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                |
| ------ | ----------------------------------- |
| 200    | 查询成功；<br>返回内容为 ，如下所示 |
| 400    | 参数错误                            |
| 404    | 没有查询到该身份标识                |
| 500    | 查询失败                            |

**示例**

```
http GET localhost:4444/identifier/news.zyy4.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs"
```





### 删除数据标识

**说明**

根据 ID 删除该数据标识。

根据数据标识的 ID 进行删除。需要使用 apiKey 进行身份验证。

**请求路径**

`DELETE`

```
/identity/:id 
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                              |
| ------ | ------ | ---- | ----------------------------- | --------------------------------- |
| id     | string | 是   | Path参数；<br>数据标识的 ID   | domain format<br>news.cat.fuxi.id |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey"     |

**响应结果**

| 响应码 | 说明                              |
| ------ | --------------------------------- |
| 204    | 删除成功，返回 No Content 和 Date |
| 400    |                                   |
| 500    | 删除失败                          |

**示例**



```
http DELETE localhost:4444/identifier/news.zyy2.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTpiOWRhN2YzZDcwNjg1MjcwYjQ5ZDUwMDRlOWU5ZjY5OTdmYjVhMTFiYTg0MzcxNjc0ZWU3N2NiYTdlYzFmMmEyIiwiaXNzIjoiZnV4aSJ9.glhE-fqjt4y3JhLgZPKpuhOOvt7tIaFRQLFEUBDBQqs"
```

<img src="FUXI.IS CA API Guide v0.3_12.13.assets/image-20211218152650834.png" alt="image-20211218152650834" style="zoom:50%;" />





### 查询某个用户名下的数据标识

**说明**

根据 client_id 查询所有数据标识。

根据client_id ，查询该 client_id 下的所有数据标识。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identifier?client_id=?
```

**请求参数**

| 参数名    | 类型   | 必填 | 说明                                 | 格式                          |
| --------- | ------ | ---- | ------------------------------------ | ----------------------------- |
| client_id | string | 是   | Query参数<br>创建 client 时返回的 id |                               |
| apiKey    | string | 是   | Json参数<br>license 的 apiKey        | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 查询成功；<br>返回内容为 GetIdentity list，GetIdentity如下所示 |
| 400    | 参数错误                                                     |
| 404    | 根据 client_id 没有查询到身份标识                            |
| 500    | 查询失败                                                     |

**示例**

```
 http GET http://localhost:4444/identifier\?client_id\=cat.fuxi.id
```

## 订阅 API

### 订阅数据标识

**说明**

订阅一个数据标识。

申请订阅一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

创建时需要使用 apiKey 进行身份验证。

**请求路径**

`POST`

```
/subscriptions
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名     | 类型   | 必填 | 说明                          | 格式                          |
| ---------- | ------ | ---- | ----------------------------- | ----------------------------- |
| apiKey     | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |
| identifier | string | 是   | Json参数<br>                  | Authorization:"bearer apiKey" |
| recipient  | string | 是   |                               |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**

```
http POST localhost:4444/subscriptions Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzOTMzMDAwLCJpYXQiOjE2MzM2NzM4MDAsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6ImNkMjU3MTJiLThmN2QtNGMyOC05NWMzLTM1YTNiZjRhNzgwNiIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiY2F0LmZ1eGkuaWQifQ.poFvStG8F5kn9dg6Mq6nmqqUCaZEsnIyeFg_yvgn8AKVoy-jGIksmmuwzM-qLimNGo0-fS2ktumPbvYXl-Ykw_9w6E8zvCA4FIlEfvDIFgCyoNp8LDlGRHrk6dfrW86mEIjjdABIcxzjGCk5hXIjP91-bSvOj1A-yyxGJpLdlDFjopeR_hfDQ2_3d6ZOoLXAZliAGel99HjtV6RO7UrEz6C3-7HLIwvft01xLItSkbldg80XFhDr0WA1kRO5iVS-zNJLiu1SPSEEHVGOum9-idEq6Y4eZe59ojZolVDWenq7jvCLBddm40omJzQ7IFhO-xAjTo_EncCBktiH3d5NFrkn9puZ37c5vIugMwm28iW1ohqMpFdyUXmx9zcWvqFbI4NTlm1wIqlt9xqLQavS_YllCS09Nkczn5Qu4EDIRSfJekK5pl5-SPrKqTTSo_6ZW6Hk8WtXJAD3IoMFLJe9D23hx9uCDPQoSMxpDNnFQxsaG9Ckip32Wb79Qx3wpCDX7CMt9IIRKCHZ4TopMTRGO4fmf0ssq_4ViVy0Yj9ot9YycrX5yz261vaSijUHhm3TjtaSmR4Wcmy5QydlGV10QOsqiqtdFJZtJK9QdfGvfpaFQbh-tU2W8SJwwZOfSbaG3Hii-IXaojpuU6L-0hKbNQ-R1M0bi7S1AXgyUPFys_k" identifier=de1.fuxi.id recipient=cat.fuxi.id
```



### 查询某个订阅

**说明**

查询某个订阅。

申请订阅一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

创建时需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/subscriptions/:id
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名    | 类型   | 必填 | 说明                          | 格式                          |
| --------- | ------ | ---- | ----------------------------- | ----------------------------- |
| apiKey    | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |
| id        | string | 是   | Path参数<br>                  | Authorization:"bearer apiKey" |
| recipient | string | 是   |                               |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**

As data requestor

```
http GET http://localhost:4444/subscriptions/75208d8c58aa5312a4d0ea86053c216c.d2.elephant.fuxi.id Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzOTMzMDAwLCJpYXQiOjE2MzM2NzM4MDAsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6ImNkMjU3MTJiLThmN2QtNGMyOC05NWMzLTM1YTNiZjRhNzgwNiIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiY2F0LmZ1eGkuaWQifQ.poFvStG8F5kn9dg6Mq6nmqqUCaZEsnIyeFg_yvgn8AKVoy-jGIksmmuwzM-qLimNGo0-fS2ktumPbvYXl-Ykw_9w6E8zvCA4FIlEfvDIFgCyoNp8LDlGRHrk6dfrW86mEIjjdABIcxzjGCk5hXIjP91-bSvOj1A-yyxGJpLdlDFjopeR_hfDQ2_3d6ZOoLXAZliAGel99HjtV6RO7UrEz6C3-7HLIwvft01xLItSkbldg80XFhDr0WA1kRO5iVS-zNJLiu1SPSEEHVGOum9-idEq6Y4eZe59ojZolVDWenq7jvCLBddm40omJzQ7IFhO-xAjTo_EncCBktiH3d5NFrkn9puZ37c5vIugMwm28iW1ohqMpFdyUXmx9zcWvqFbI4NTlm1wIqlt9xqLQavS_YllCS09Nkczn5Qu4EDIRSfJekK5pl5-SPrKqTTSo_6ZW6Hk8WtXJAD3IoMFLJe9D23hx9uCDPQoSMxpDNnFQxsaG9Ckip32Wb79Qx3wpCDX7CMt9IIRKCHZ4TopMTRGO4fmf0ssq_4ViVy0Yj9ot9YycrX5yz261vaSijUHhm3TjtaSmR4Wcmy5QydlGV10QOsqiqtdFJZtJK9QdfGvfpaFQbh-tU2W8SJwwZOfSbaG3Hii-IXaojpuU6L-0hKbNQ-R1M0bi7S1AXgyUPFys_k"
```



As data owner

```
http GET http://localhost:4444/subscriptions/f4b26d7859c6a891404508a7df21cc41.d1.elephant.fuxi.id  Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzNjcwNTAwLCJpYXQiOjE2MzM0MTEzMDAsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6IjE1ZGJjOTdkLTk4YWUtNDY4MS1iZGVmLTlmYjZmYTE1MDE5ZiIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiZWxlcGhhbnQuZnV4aS5pZCJ9.oX_9hL-ls7gQEKtiJjFviD2fcHskEUM0s4Ub0fkHhbuyTTK-fiCROwkJ2grmPwJ2JfhG2Zll6P1gHogT-S_KCb7IgRS7AbrhwocyFQddzwYOsTn6DECxOR1_KEbEGMZJHGfa_pfZseG7HWhnkr9wwWkT5Q46mdBtZEvvorBKhroap2ADV-Va-LLC7nYszAU1CL8IFtZ04amnHNq4barTNvgTa9jW4AG_naRDzZrBU3Yz7d74dhouP93oijtZouq-eQndd6XrvHYbPw3-JX2PQcQacJAcLMDD56yO5z_xwMGI3qyF80L0FLqEcKOIp0o9hodyUu6CnH65WX-Cxs9-Q9SjBS6GFh6hPnKQ6_8FOHSOcykxP8Tr11ChMBmhdsHpWB8PiN8sTr6Szd74yS0EO-8N1WZwDPiZqyS0IgzL2ipfW4D_UGxCOWEG4AemTALNXlcN-7gm-EOBGpc7IlsopPcM0H5LFz1s91KjXCAmUi2eP5nVLmG09j-y4p0tFHso9LXguFtJsxwGeiQrIyy2PZeJFBzM0fAdcLlYyP5uto6kZQbVv9PVJCGSj-nwLm39LbcVh8Jjxfc7Qklavf-dX5Or9iuHOfXm9U_fjLSbbMa3y6V18vXUwp8I4beDvheYc21wamBlT0cgjvjVYq6VKSltSJs_j4qqZ6-Rb7j74Oo" 
```

### 删除订阅

**说明**

删除订阅。

申请订阅一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

创建时需要使用 apiKey 进行身份验证。

**请求路径**

`DELETE`

```
/subscriptions/:id
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名    | 类型   | 必填 | 说明                          | 格式                          |
| --------- | ------ | ---- | ----------------------------- | ----------------------------- |
| apiKey    | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |
| id        | string | 是   | Path参数<br>                  | Authorization:"bearer apiKey" |
| recipient | string | 是   |                               |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**



```
http DELETE localhost:4444/subscriptions/f4b26d7859c6a891404508a7df21cc41.d1.elephant.fuxi.id Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzNjAwODYxLCJpYXQiOjE2MzMzNDE2NjEsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6ImVhMjcxNTYxLTlkMGYtNDk5YS05YWY1LTMzMGI2ODhlZjdmYyIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiY2F0LmZ1eGkuaWQifQ.QCYfQU1kdfGdzfI2wPRR3bW5lCAGjU9f7CVX__WeY3hme7fPrfPx91YdG-ioVDpBNr-xW71zYjIJelM0BDWwGcKup1qRtvOwIOfTSYU4TAltj59V-CPw1taFXLg84P7kBqBRvZwD02l8p3utKtxPn0Qcnyg_wVo-lBgurXhZ8Oct8ihe6ARzx8Ug3W4Eqqjx0cC5R9OVJAzQ75L_h4nJ_wMsOr8GxVsiDqo63gfXpUx1a4clvGKva4OCSFh4CVQDlK2zPRKNEV4amqVvM5mSqv2BUiMVEeDTD_qVfZEWVF9zGXxI_o_WIyIdJt__iqKpo_AcDSYr7K0L-5t4A5TK5HzQ39QNMlmHTtTiMjHrdJgCsGgygKjoi901E0wF8b0urgZIXb8ety1iLhX7OzNAzus1shYnbRjgCoJQudOwebw8dk0fkdVsBHbT0q0yo1UvwhGX9H1dnC2_daK_GbzqOx6l77J483PeW7CSrO6PY1rz3M5dZ7Eu9BxpSFrScQ6zGv0kH6dJw5Wzu_h0vWM6swwr337nfTdkLqNojDctIXnmj78nfqFdkL_NI_pSJt0I9BeshOEXHB5xvNTxVA4MhMGmTjQzcfQMYunK44ZJI0fkFz8g7saoKCJjsKW1mFWiKFVz4_lm1uEgGfqc5l5kS9YrHagROAbyiaQ2bEm-0UA"
```



### 查询订阅

**说明**

查询某个订阅。

申请订阅一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

创建时需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/subscriptions?role=?
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名 | 类型   | 必填 | 说明                                 | 格式                          |
| ------ | ------ | ---- | ------------------------------------ | ----------------------------- |
| apiKey | string | 是   | Json参数<br>license 的 apiKey        | Authorization:"bearer apiKey" |
| role   | string | 是   | Query参数<br>取值为requesor或者owner | Authorization:"bearer apiKey" |
| offset | string | 是   |                                      |                               |
| limit  | string | 是   |                                      |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**

As data requestor

```
http GET http://localhost:4444/subscriptions?role=requestor Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzNjAwODYxLCJpYXQiOjE2MzMzNDE2NjEsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6ImVhMjcxNTYxLTlkMGYtNDk5YS05YWY1LTMzMGI2ODhlZjdmYyIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiY2F0LmZ1eGkuaWQifQ.QCYfQU1kdfGdzfI2wPRR3bW5lCAGjU9f7CVX__WeY3hme7fPrfPx91YdG-ioVDpBNr-xW71zYjIJelM0BDWwGcKup1qRtvOwIOfTSYU4TAltj59V-CPw1taFXLg84P7kBqBRvZwD02l8p3utKtxPn0Qcnyg_wVo-lBgurXhZ8Oct8ihe6ARzx8Ug3W4Eqqjx0cC5R9OVJAzQ75L_h4nJ_wMsOr8GxVsiDqo63gfXpUx1a4clvGKva4OCSFh4CVQDlK2zPRKNEV4amqVvM5mSqv2BUiMVEeDTD_qVfZEWVF9zGXxI_o_WIyIdJt__iqKpo_AcDSYr7K0L-5t4A5TK5HzQ39QNMlmHTtTiMjHrdJgCsGgygKjoi901E0wF8b0urgZIXb8ety1iLhX7OzNAzus1shYnbRjgCoJQudOwebw8dk0fkdVsBHbT0q0yo1UvwhGX9H1dnC2_daK_GbzqOx6l77J483PeW7CSrO6PY1rz3M5dZ7Eu9BxpSFrScQ6zGv0kH6dJw5Wzu_h0vWM6swwr337nfTdkLqNojDctIXnmj78nfqFdkL_NI_pSJt0I9BeshOEXHB5xvNTxVA4MhMGmTjQzcfQMYunK44ZJI0fkFz8g7saoKCJjsKW1mFWiKFVz4_lm1uEgGfqc5l5kS9YrHagROAbyiaQ2bEm-0UA"
```



As data owner

```
http GET http://localhost:4444/subscriptions?role=owner&status=Applied Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzNjcwNTAwLCJpYXQiOjE2MzM0MTEzMDAsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6IjE1ZGJjOTdkLTk4YWUtNDY4MS1iZGVmLTlmYjZmYTE1MDE5ZiIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiZWxlcGhhbnQuZnV4aS5pZCJ9.oX_9hL-ls7gQEKtiJjFviD2fcHskEUM0s4Ub0fkHhbuyTTK-fiCROwkJ2grmPwJ2JfhG2Zll6P1gHogT-S_KCb7IgRS7AbrhwocyFQddzwYOsTn6DECxOR1_KEbEGMZJHGfa_pfZseG7HWhnkr9wwWkT5Q46mdBtZEvvorBKhroap2ADV-Va-LLC7nYszAU1CL8IFtZ04amnHNq4barTNvgTa9jW4AG_naRDzZrBU3Yz7d74dhouP93oijtZouq-eQndd6XrvHYbPw3-JX2PQcQacJAcLMDD56yO5z_xwMGI3qyF80L0FLqEcKOIp0o9hodyUu6CnH65WX-Cxs9-Q9SjBS6GFh6hPnKQ6_8FOHSOcykxP8Tr11ChMBmhdsHpWB8PiN8sTr6Szd74yS0EO-8N1WZwDPiZqyS0IgzL2ipfW4D_UGxCOWEG4AemTALNXlcN-7gm-EOBGpc7IlsopPcM0H5LFz1s91KjXCAmUi2eP5nVLmG09j-y4p0tFHso9LXguFtJsxwGeiQrIyy2PZeJFBzM0fAdcLlYyP5uto6kZQbVv9PVJCGSj-nwLm39LbcVh8Jjxfc7Qklavf-dX5Or9iuHOfXm9U_fjLSbbMa3y6V18vXUwp8I4beDvheYc21wamBlT0cgjvjVYq6VKSltSJs_j4qqZ6-Rb7j74Oo" 
```



### 审批订阅

**说明**

审批某个订阅。

如果审批通过，订阅的状态将变为Granted，订阅的Content字段将包括生成的Access Token；否则，订阅的状态为Refused，Content字段或者为拒绝原因，或者为其他任何内容。

**请求路径**

`PATCH`

```
/subscriptions/:id
```

**请求参数**

以 Json 的格式放在请求体中。

| 参数名 | 类型   | 必填 | 说明                                | 格式                          |
| ------ | ------ | ---- | ----------------------------------- | ----------------------------- |
| apiKey | string | 是   | Json参数<br>license 的 apiKey       | Authorization:"bearer apiKey" |
| id     | string | 是   | Path参数<br>取值为requesor或者owner | Authorization:"bearer apiKey" |
| status | string | 是   |                                     |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**

```
http PATCH localhost:4444/subscriptions/d0c3d423d1d1c3f5013a34c30aec7bdd.de1.fuxi.id status=Granted Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiaWRlbnRpdHkiLCJkYXRhIGlkZW50aWZpZXIiXSwiZXhwIjoxNjMzOTMxNjUyLCJpYXQiOjE2MzM2NzI0NTIsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6ImEyZTZhNGE5LTAxMmEtNDdlMi04MTdjLWZmY2M2NTFlZmE1NyIsInNjcCI6WyJkYXRhIGV4Y2hhbmdlIiwiaWRlbnRpdHkiXSwic3ViIjoiZWxlcGhhbnQuZnV4aS5pZCJ9.T-Zrxl12tPbNGzCIVoeWXBAZHALtcF4YYsba67NUN8VpiKeQlV-xKT6L5SPMuLrLdJ98-pkTb2WZ64pt3FyBXORz7wEygPR2svTLFoHPWvkejLx1jd9d6BON6gd8IJIW2rr8LABujNEPfbsa70v7W0XvndzITTfg6Pg02jSQop1kOWxrSsuXfd2nAF55tpED3FCMlrGvSx8wV2GCoRngHBYR3tSKu_CIpPB1CVjIdkvm6k95YuTijDMc7J_kEiDDpSX62zJYFL6fy5qjnu9BvDoEJdNxBcJizFmnX2kxt6PlaNU_v9z39IcOT8hhuK_smRwL8sxY5T5IAk1p5u4lseJTzzcf3QSi_L22_wg335mj2WtyVp9FP715ja2biThgxgsGJliMGbxkxTnINgpZufUxK7xA3vj0qK1pi-wyCgyPzFQuTrTMbp8DG8EdGWSCO1lCCux-1fEp8gvlaPf2r1i_13htNrs8Ujebu2VGSnfc3yBXa8EgoQo-AEl1NEIkA6UHJoauXDVAmvm3fHLFaa09H_8Pg31AHnoit9m92vtKXThS6LQ2BTlxmZsjeEqI1OIsht7SKwW7jg4ODfFIKXWN0EydP45iHioSYD4NXQREKJAZ5SK4advRxf0yIDr4vqZAC2PQCfX9mOVNMZ-8iF6vWfgT0JYcXcLlJ8cUkfg" 
```



### 检验订阅后生成的Access Token

**说明**

审批某个订阅。

如果审批通过，订阅的状态将变为Granted，订阅的Content字段将包括生成的Access Token；否则，订阅的状态为Refused，Content字段或者为拒绝原因，或者为其他任何内容。

**请求路径**

`GET`

```
/oauth2/authenticate
```

**请求参数**

以 Json 的格式放在请求体中。

HTTP Header: Authorization

| 参数名 | 类型   | 必填 | 说明                          | 格式                          |
| ------ | ------ | ---- | ----------------------------- | ----------------------------- |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的client 与 owner身份标识所在的 client 不一致，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**



```
http GET http://localhost:4445/oauth2/authenticate Authorization:"bearer eyJhbGciOiJSUzI1NiJ9.eyJhdWQiOlsiY2F0LmZ1eGkuaWQiXSwiZXhwIjoxNjMzOTM0Nzc1LCJpYXQiOjE2MzM2NzU1NzUsImlzcyI6Ik9SWSBIeWRyYSIsImp0aSI6Ijc3YzZhYzAwLWRjMDctNDZkZi05YTYyLTk3MDA0OTkzZmFlZiIsInJlcXVlc3RvciI6ImNhdC5mdXhpLmlkIiwic2NwIjpbImRlMS5mdXhpLmlkIl0sInN1YiI6ImVsZXBoYW50LmZ1eGkuaWQiLCJzdWJzY3JpcHRpb24iOiJkMGMzZDQyM2QxZDFjM2Y1MDEzYTM0YzMwYWVjN2JkZC5kZTEuZnV4aS5pZCJ9.Hay5HiKMuicPFjNyXgZKS5h9RIqDmQHtnrCMg1gemCLDCn_CPPMT100Dj8gHAA3wPSCd57KghHFV27iGFBA50_nSyUJ46bhBIGN0IsX_z9XSjWOf1K5GuhuD-1MMaA5w5yG9NSGbH4NAZ9MpPDvs1SjvAI661Q0hedUpgw68uuwqCNiLdlbRksVH3WddxMX-IaW8488JDeXhC1nbTeiLdl5GGyn1G7M5RAPzj9Otk7kDZqtoTx79EPF6jQ7RU5KnykDLpkTr4jBzq8EHXUUxhaa7rLiVMITD3WOQh1c9IAfSYtcpWF3np24P4pDO60Iv2JqwESPc_MrN0nbc-3lr2H5susA0H5HhVdGMvMdgexx3-OIiFj2pMZvM8RwTf5jSXUQZA7TcNDDsZTF0snZRgV2_8i9szT4j4-ittjGmjPI_huiy30JI9mfBw1WsFcMPQ2fXxTJy0y8cIg4uitkEHZiGFza2BVymKlpttGoL1TF21vgnS3JzgVBX8bO60wuZ3LoAjFbVeruTjgx67D7O9VEJ-g6ifdVY8E8wP2uTfWwjjflRfOC2tIg7fXghduUJgWzvBKrRzC78ldn_DUcQLn_8nwYOvLgrgbvC6fqnMomwQtglCBWG4MVPRz-EKS2MFCxcAY5EVY3OHYwL6qldmmnVORFBTuNmCblCIlnhy2Y"
```


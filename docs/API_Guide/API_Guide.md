

# FUXI IS CA API文档

[下载 API 文档](https://github.com/zhangyouyi125/wiki/raw/gh-pages/API_Guide/API_Guide.pdf ){ .md-button }

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

## 1、用户账号 API

​	用户账号操作相关的API说明。

### 1.1 创建Client

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
| organization  | string | 是   | 用户所在组织 |
| email         | string | 是   | 邮箱         |
| mobile        | string | 是   | 电话         |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 201    | 创建成功；<br>返回内容为 `id、clientID、apiKey`，`id`是 license 的id<br>用户进行其他操作时，需要使用`apiKey`用于验证 license |
| 400    | 参数错误                                                     |
| 409    | 创建失败，该用户已存在                                       |
| 500    | 创建失败                                                     |

**范例**

```powershell
http POST http://localhost:4445/clients client_secret=abc123 organization=fuxi mobile=10013270953 client_name=botao email=botao@fuxiinstitution.org
```

![image-20211224172659683](FUXI.IS CA API Guide.assets/image-20211224172659683.png)

### 1.2 创建License

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
| client_secret | string | 是   | Json参数；<br>创建Client时输入的client_secret | Authorization:"bearer client_secret" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为 `id、clientID、apiKey`，`id`是 license 的id<br>用户进行其他操作时，需要使用`apiKey`用于验证 license |
| 400    | 参数错误；client_id 获取用户失败                             |
| 403    | client secret身份验证失败                                    |
| 500    | 创建 license 失败                                            |

**示例**

```powershell
http POST http://localhost:4445/clients/did:fuxi:53409d7bac7cb206aef53b4565cab8250284d98c5e608b82a81ff79f4c21b486/licenses Authorization:"bearer abc123"
```

![image-20211224172742449](FUXI.IS CA API Guide.assets/image-20211224172742449.png)

### 1.3 查询License

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
| client_secret | string |  是  | Json参数<br>创建 client 时的输入的client_secret | Authorization:"bearer client_secret" |

**响应结果**

| 响应码 | 说明                                                 |
| ------ | :--------------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 用户所有 license 的相关信息 |
| 400    | 参数错误                                             |
| 403    | client secret身份验证失败                            |
| 500    | 查询失败                                             |

**示例**

```powershell
http http://localhost:4445/clients/did:fuxi:53409d7bac7cb206aef53b4565cab8250284d98c5e608b82a81ff79f4c21b486/licenses Authorization:"bearer abc123"
```

### 1.4 查询可用的名字空间

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

| 参数名    | 类型   | 必填 | 说明                                    | 格式                          |
| --------- | ------ | :--: | --------------------------------------- | ----------------------------- |
| client_id | string |  是  | Path参数                                |                               |
| apiKey    | string |  是  | Json参数<br>创建 license 后返回的apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                         |
| ------ | :------------------------------------------- |
| 200    | 查询成功；<br>返回内容为所有可用的 namespace |
| 500    | 查询失败                                     |

**示例**

```powershell
http http://localhost:4445/clients/did:fuxi:53409d7bac7cb206aef53b4565cab8250284d98c5e608b82a81ff79f4c21b486/namespaces Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo2Yzk1OGM4NTIzNzcwNTAxZDA3MjY0NWUwZGJkYmUwODdhMzNmMzViYWNkMGY2OWRjZmYxYzg0ZmU2ZmVhZGVjIiwiaXNzIjoiZnV4aSJ9.OgLpCzrd_-XS3j6ULXC-xJRnfpDjlonR_J9Nmg_xWBg"
```

![image-20211207104623099](FUXI.IS CA API Guide.assets/image-20211207104623099.png)

## 2、身份标识 API

### 2.1 创建身份标识

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

Json参数，以 Json 的格式放在请求体Body中。

| 参数名 | 类型   | 必填 | 说明                                              | 格式                                                         |
| ------ | ------ | ---- | ------------------------------------------------- | ------------------------------------------------------------ |
| apiKey | string | 是   | Json参数<br>license 的 apiKey                     | Authorization:"bearer apiKey"                                |
| id     | string | 是   | Json参数；<br>id 自定义，但需要在可用的名字空间里 | domain format<br>cat.fuxi.id<br>其中fuxi.id为可用的namespace |
| name   | string | 是   | Json参数                                          | general name<br>可自定义                                     |
| owner  | string | 是   | Json参数；<br>该身份标识的拥有者                  |                                                              |
| email  | string | 是   | Json参数                                          |                                                              |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 201    | 创建成功；<br>返回内容为 CreateIdentity，如下所示<br>`PublicKey、PrivateKey`是为该身份标识创建的公私钥对 |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 429    | 创建失败，该身份标识在domain中已存在                         |
| 500    | 创建失败                                                     |

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
http POST http://localhost:4444/identity Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA" id=bt.fuxi.id name=bt email=bt@fuxiinstitution.org owner=fuxi
```

![image-20211224172942076](FUXI.IS CA API Guide.assets/image-20211224172942076.png)

### 2.2 查询身份标识

**说明**

根据 ID 查询身份标识。

根据身份标识创建时定义的ID进行查询。需要使用 apiKey 进行身份验证（**注：**apikey对应的client可以不是该标身份标识对应的client）。

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
http GET http://localhost:4444/identity/bt.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211224173041441](FUXI.IS CA API Guide.assets/image-20211224173041441.png)

### 2.3 删除身份标识

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
http DELETE http://localhost:4444/identity/bt.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211222133449710](FUXI.IS CA API Guide.assets/image-20211222133449710.png)

### 2.4 查询某个用户名下的身份标识

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

```powershell
http GET http://localhost:4444/identity?client_id=did:fuxi:53409d7bac7cb206aef53b4565cab8250284d98c5e608b82a81ff79f4c21b486  Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211224173358168](FUXI.IS CA API Guide.assets/image-20211224173358168.png)

## 3、数据标识 API

在进行数据标识管理操作时，需要用 apiKey 进行身份认证，该 apiKey 是数据标识拥有者的 license。

### 3.1 创建数据标识

**说明**

创建一个身份标识（Data identifier）。

申请创建一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

注：创建时需要使用 apiKey 进行身份验证，如果 apiKey 验证失败或者不是该身份标识所对应的身份，则无法创建数据标识。

**请求路径**

`POST`

```
/identifier
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名           | 类型   | 必填 | 说明                                                         | 格式                                                         |
| ---------------- | ------ | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| apiKey           | string | 是   | Json参数<br>与 identity 对应的 apiKey                        | Authorization:"bearer apiKey"                                |
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
| 201    | 创建成功；<br>返回内容为相应的注册信息                       |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与 owner身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限为该身份创建数据标识 |
| 500    | 创建数据标识失败                                             |

**示例**

```powershell
http POST localhost:4444/identifier Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA" id=news3.bt.fuxi.id name=bt_news3 owner=bt.fuxi.id dataAddress="https://scholars-stage.org/xi-jinpings-war-on-spontaneous-order/" dataDigest="abc" subscriptionType="Free" categoryID="education" tags:='["News","Home"]' metadata:='{"type":"test", "site":"zhihu"}'

```

![image-20211224173508519](FUXI.IS CA API Guide.assets/image-20211224173508519.png)

### 3.2 查询数据标识

**说明**

根据 ID 查询数据标识。

根据数据标识创建时定义的 ID 进行查询。需要使用 apiKey 进行身份验证

注：需要使用拥有该数据标识的 apiKey 才能进行查询，即 apiKey 的身份与拥有该数据标识的身份相对应 。

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

| 响应码 | 说明                                              |
| ------ | ------------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 DataIdentifier，如下所示 |
| 401    | 参数错误                                          |
| 404    | 没有查询到该数据标识                              |
| 500    | 查询失败                                          |

```
DataIdentifier {
    	Id:               id,
		Name:             name,
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

示例**

```powershell
http GET localhost:4444/identifier/news.bt.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211224173618050](FUXI.IS CA API Guide.assets/image-20211224173618050.png)

### 3.3 删除数据标识

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

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 204    | 删除成功，返回 No Content 和 Date                            |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与 owner身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限删除该数据标识 |
| 404    | 不存在该数据标识                                             |
| 500    | 删除失败                                                     |

**示例**

```powershell
http DELETE localhost:4444/identifier/news.bt.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

<img src="FUXI.IS CA API Guide.assets/image-20211223100449033.png" alt="image-20211223100449033" style="zoom:67%;" />

### 3.4 查询某个身份标识下的数据标识

**说明**

根据 owner 查询所有数据标识，这里的 owner 是指拥有该数据标识的身份标识。

根 据 owner ，查询该 owner 下的所有数据标识。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identifier?owner=?
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                            | 格式                          |
| ------ | ------ | ---- | ------------------------------- | ----------------------------- |
| owner  | string | 是   | Query参数<br>身份标识表示 owner | domain format<br/>cat.fuxi.id |
| apiKey | string | 是   | Json参数<br>license 的 apiKey   | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 查询成功；<br>返回内容为 DataIdentifier list                 |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与 owner身份标识所在的 client 不一致<br>实际身份与验证的身份不同，没有权限查询该身份标识下的数据标识 |
| 404    | 根据 client_id 没有查询到身份标识                            |
| 500    | 查询失败                                                     |

**示例**

```powershell
http GET http://localhost:4444/identifier?owner=bt.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211224173735184](FUXI.IS CA API Guide.assets/image-20211224173735184.png)

### 3.5 查询包含相关tag的数据标识

**说明**

根据 tag 查询所有数据标识。

根据 tag，查询有相同 tag 的数据标识。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identifier?tag=?
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                          |
| ------ | ------ | ---- | ----------------------------- | ----------------------------- |
| owner  | string | 是   | Query参数<br>                 |                               |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                         |
| ------ | -------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 DataIdentifier list |
| 400    | 参数错误                                     |
| 500    | 查询失败                                     |

**示例**

```powershell
http GET http://localhost:4444/identifier?tag=News Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

### 3.6 查询包含相关metadata的数据标识

**说明**

根据 metadata查询所有数据标识。

根据 metadata，查询有相同 metadata的数据标识。需要使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/identifier?metadata=?
```

**请求参数**

| 参数名 | 类型   | 必填 | 说明                          | 格式                          |
| ------ | ------ | ---- | ----------------------------- | ----------------------------- |
| owner  | string | 是   | Query参数<br>                 | Json格式<br>"site:zhihu"      |
| apiKey | string | 是   | Json参数<br>license 的 apiKey | Authorization:"bearer apiKey" |

**响应结果**

| 响应码 | 说明                                         |
| ------ | -------------------------------------------- |
| 200    | 查询成功；<br>返回内容为 DataIdentifier list |
| 400    | 参数错误                                     |
| 500    | 查询失败                                     |

**示例**

```
http GET http://localhost:4444/identifier?metadata="site:zhihu" Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

## 4、订阅 API

### 4.1 订阅数据

**说明**

通过数据标识订阅一个数据。

申请订阅一个数据标识。每个身份标识下可以为自己的数据创建多个数据标识，用户可对其进行管理操作，也可以对其他用户的数据进行订阅。

申请订阅数据标识需要使用 apiKey 进行身份验证，如果订阅成功会返回一个订阅 id。

如果订阅其他用户的数据，则需要其他用户进行审批，如果订阅自己的数据则不需要审批。

**请求路径**

`POST`

```
/subscriptions
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名     | 类型    | 必填 | 说明                                  | 格式                               |
| ---------- | ------- | ---- | ------------------------------------- | ---------------------------------- |
| apiKey     | string  | 是   | Json参数<br>与 identity 对应的 apiKey | Authorization:"bearer apiKey"      |
| requestor  | srtring | 是   | Json参数<br/>申请者                   | domain format<br/>bt2.fuxi.id      |
| identifier | string  | 是   | Json参数<br>申请订阅的数据标识        | domain format<br/>news.cat.fuxi.id |
| recipient  | string  | 是   | Json参数<br/>                         | domain format<br/>                 |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 201    | 创建成功，返回的 subscription 字段为订阅 id                  |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限申请订阅 |
| 404    | 没有找到数据标识的拥有者                                     |
| 500    | 创建数据标识失败                                             |

**示例**

requestor != owner 订阅其他用户的数据

```powershell
http POST http://localhost:4444/subscriptions Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA" requestor=bt2.fuxi.id identifier=news.bt.fuxi.id recipient=sy.fuxi.id
```

![image-20211225140534011](FUXI.IS CA API Guide.assets/image-20211225140534011.png)

requestor == owner 订阅自己的数据

```powershell
http POST http://localhost:4444/subscriptions Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA" requestor=bt.fuxi.id identifier=news.bt.fuxi.id recipient=sy.fuxi.id
```

<img src="FUXI.IS CA API Guide.assets/image-20211225174429752.png" alt="image-20211225174429752" style="zoom: 50%;" />

### 4.2 查询某个订阅

**说明**

根据 id 查询某个订阅。是

根据 id 进行查询，id 是成功申请订阅数据标识后返回的 subscription 字段的内容。需要使用 apiKey 进行身份验证。

注：只能查询自己的订阅。

**请求路径**

`GET`

```
/subscriptions/{id}?identity=?
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名   | 类型   | 必填 | 说明                                  | 格式                                             |
| -------- | ------ | ---- | ------------------------------------- | ------------------------------------------------ |
| apiKey   | string | 是   | Json参数<br>与 identity 对应的 apiKey | Authorization:"bearer apiKey"                    |
| id       | string | 是   | Path参数<br>成功订阅后返回的 id 值    | bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id |
| identity | string | 是   | Query参数<br>申请订阅者的身份标识     | domain format<br>bt2.fuxi.id                     |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 查询成功，返回相应信息                                       |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限查询 |
| 404    | 没有找到该身份标识                                           |
| 500    | 查询失败                                                     |

**示例**

As data requestor

```powershell
http GET http://localhost:4444/subscriptions/bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id?identity=bt2.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

As data owner

```powershell
http GET http://localhost:4444/subscriptions/bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id?identity=bt.fuxi.id   Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA" 
```

![image-20211225143022741](FUXI.IS CA API Guide.assets/image-20211225143022741.png)

### 4.3 删除订阅

**说明**

删除订阅。

根据订阅的 id 、订阅者的身份标识对其进行删除，需要使用 apiKey 进行身份认证。

**请求路径**

`DELETE`

```
/subscriptions/{id}?identity=?
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名   | 类型   | 必填 | 说明                                  | 格式                                             |
| -------- | ------ | ---- | ------------------------------------- | ------------------------------------------------ |
| apiKey   | string | 是   | Json参数<br>与 identity 对应的 apiKey | Authorization:"bearer apiKey"                    |
| id       | string | 是   | Path参数<br>成功订阅后返回的 id 值    | bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id |
| identity | string | 是   | Query参数<br>申请订阅者的身份标识     | domain format<br/>bt2.fuxi.id                    |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 204    | 删除成功                                                     |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限删除 |
| 404    | 没有找到该身份标识                                           |
| 500    | 删除失败                                                     |

**示例**

```powershell
http DELETE http://localhost:4444/subscriptions/bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id?identity=bt2.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

<img src="FUXI.IS CA API Guide.assets/image-20211225145048877.png" alt="image-20211225145048877" style="zoom:50%;" />

### 4.4 查询订阅

**说明**

List  查询订阅。

输入身份标识、role 和 apiKey 来查询订阅。role 的取值为 requesor 或 owner，表明自己作为订阅的请求者或者数据的拥有者来进行查询。使用 apiKey 进行身份验证。

**请求路径**

`GET`

```
/subscriptions
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名   | 类型   | 必填 | 说明                                  | 格式                          |
| -------- | ------ | ---- | ------------------------------------- | ----------------------------- |
| apiKey   | string | 是   | Json参数<br>与 identity 对应的 apiKey | Authorization:"bearer apiKey" |
| role     | string | 是   | Json参数<br>取值为requesor或者owner   | requesor or owner             |
| identity | string | 是   | Json参数<br>                          | domain format<br/>bt2.fuxi.id |
| status   | string | 否   | Json参数<br>已审批的为 Granted        |                               |
| type     | string | 否   | Json参数                              |                               |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 查询成功，返回相应订阅信息                                   |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限查询 |
| 404    | 没有找到该身份标识                                           |
| 500    | 查询失败                                                     |

**示例**

As data requestor

```powershell
http GET http://localhost:4444/subscriptions role==requestor identity==bt2.fuxi.id Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211225160012911](FUXI.IS CA API Guide.assets/image-20211225160012911.png)



As data owner

```powershell
http GET http://localhost:4444/subscriptions role==owner identity==bt.fuxi.id status==Granted Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

![image-20211225175200714](FUXI.IS CA API Guide.assets/image-20211225175200714.png)

### 4.5 审批订阅

**说明**

审批某个订阅。

输入订阅 id、数据拥有者的身份标识、apiKey 和 status 进行审批。

如果审批通过，订阅的状态将变为Granted，订阅的Content字段将包括生成的Access Token；否则，订阅的状态为Refused，Content字段或者为拒绝原因，或者为其他任何内容。

**请求路径**

`PATCH`

```
/subscriptions/{id}?identity=?
```

**请求参数**

Json参数，以 Json 的格式放在请求体Body中。

| 参数名  | 类型   | 必填 | 说明                               | 格式                                             |
| ------- | ------ | ---- | ---------------------------------- | ------------------------------------------------ |
| apiKey  | string | 是   | Json参数<br>license 的 apiKey      | Authorization:"bearer apiKey"                    |
| id      | string | 是   | Path参数<br>成功订阅后返回的 id 值 | bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id |
| dentity | string | 是   | Query参数<br>数据拥有者的身份标识  | domain format<br/>bt2.fuxi.id                    |
| status  | string | 是   | Json参数<br>Granted 为同意审批     | Granted                                          |

**响应结果**

| 响应码 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 200    | 创建成功；<br>返回内容为相应的注册信息 CreateDataIdentifier，如下所示<br> |
| 400    | 参数错误                                                     |
| 401    | license 验证失败（apiKey验证失败）                           |
| 403    | license 所在的 client 与身份标识所在的 client 不一致<br/>实际身份与验证的身份不同，没有权限审批 |
| 404    | 没有找到该身份标识                                           |
| 500    | 审批失败                                                     |

```powershell
http PATCH localhost:4444/subscriptions/bc14a1ff404221dd13d9cd09cb8c73ae.news.bt.fuxi.id?identity=bt.fuxi.id status=Granted Authorization:"bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJkaWQ6ZnV4aTo1MzQwOWQ3YmFjN2NiMjA2YWVmNTNiNDU2NWNhYjgyNTAyODRkOThjNWU2MDhiODJhODFmZjc5ZjRjMjFiNDg2IiwiaXNzIjoiZnV4aSJ9.oWMAIhyTDLTQPtkxQdmfuSD4X92Oy1EzXH6BReEpxYA"
```

<img src="FUXI.IS CA API Guide.assets/image-20211225175534096.png" alt="image-20211225175534096" style="zoom:50%;" />






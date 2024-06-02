```
nest new [項目名稱]
```

controller 控制器
service 服務
module 模組

## 8.nestJs cli 指令

```
nest g co [控制器名稱]
nest g s [服務名稱]
nest g mo [模組名稱]
nest g resource [資源名稱]// 會將相關的 controller、service、module 建立完，並且建立 RESTful API
```

## 7.nestJs Restful 風格設計

### 1.介紹 restful

RESTful 是一種架構風格，它是一種網路架構的概念，它是一種用來架構網路應用的分類法，它是一種設計網路 API 的方式。

- 查詢：GET /users
- 新增：POST /users
- 修改：PUT /users/:id
- 修改：PATCH /users/:id
- 刪除：DELETE /users/:id

### 2.Restful 版本控制

一共有三種版本控制方式：

1. URI Versioning：使用 URI 的版本號，例如：http://api.example.com/v1/users
2. Header Versioning：使用 HTTP Header 的版本號，例如：HTTP/1.1 200 OK
3. Media Type Versioning: 使用媒體類型的版本號，請求的 Header 包含 Accept 屬性，例如：Accept: application/json; version=1.0

使用第一種的配置方式，在 app.module.ts 裡面加上以下設定：

```ts
import { NestFactory } from '@nestjs/core'
import { VersioningType } from '@nestjs/common'
import { AppModule } from './app.module'
async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  app.enableVersioning({
    type: VersioningType.URI,
  })
  await app.listen(3000)
}
bootstrap()
```

在 controller 裡面， 針對整個 controller

```ts
@Controller({
  path: 'users',
  version: '1',
})
export class UserController {
  constructor(private readonly userService: UserService) {}
}
```

針對特定路由加上@Version('1')，表示版本號為 1。

```ts
  @Version('1')
  @Get()
  findAll() {
    return this.userService.findAll();
  }
```

### 3.Status Code

200 OK
304 Not Modified // 資料沒有更新
400 Bad Request // 請求參數錯誤
401 Unauthorized // 未登入
403 Forbidden // 禁止存取，通常是沒有權限，要加上 origin referrer
404 Not Found // 找不到資料
405 Method Not Allowed // 請求方法不被允許
500 Internal Server Error // 伺服器端錯誤
502 Bad Gateway //上游伺服器回應錯誤或伺服器問題

---

## 8.nestJs 控制器詳解

nestJs 提供了方法參數裝飾器，可以用來快速取得參數

- @Request() :req
- @Response() :res
- @Next() :next
- @Session() :req.session
- @Param(key?: string) :req.params[key]
- @Body(key?: string) :req.body[key]
- @Query(key?: string) :req.query[key]
- @Headers(name: string) :req.headers[name]
- @HttpCode

---

## 9.nestJs Session

session 是伺服器為每個用戶的瀏覽器創建一個唯一的 session id，用來記錄用戶的狀態，可以用來儲存用戶的登入資訊、購物車、瀏覽紀錄等。

使用 nestJs 的預設框架 express，他也支援 express 的。

```console
npm i express-session
npm i @types/express-session -D
```

通過在 main.ts 引入，並通過 app.use() 來使用。

```ts
import * as session from 'express-session'

app.use(session())
```

### 參數配置

- secret: 加密用的密鑰，必須設定
- rolling: 是否重新產生 session id，預設為 false
- cookie: 設定 cookie 的參數，包含 maxAge、domain、path、httpOnly、secure、sameSite
- name: session 的名稱，預設為 connect.sid

---

## 10.nestJs providers

providers 是 nestJs 的核心概念，他是用來注入的依賴項目，可以是其他服務、模組、類別等。

1. 創建 server class，並使用 @Injectable() 來標記為可注入的類別。
2. 在 對應 module 中引入該類別，並使用 providers 屬性來注入。
3. 在 對應 controller 中使用 來注入剛剛的類別，並且可以使用其方法。

### 1.自定義名稱

```ts
// in app.service.ts
@Module({
  imports: [DemoModule, UserModule],
  controllers: [AppController, DemoController],
  providers: [
    //AppService,下面是完整寫法
    {
      provide: 'ABC', // 自定義注入名稱，可在控制器使用，但就必須用@Inject('ABC')修飾來標注名稱
      useClass: AppService, //要注入的服務類別
    },
  ],
})

```

```ts
// in app.controller.ts
@Controller()
export class AppController {
  constructor(@Inject('ABC') private readonly appService: AppService) {}

  @Get('hello')
  getHello(): string {
    return this.appService.getWilliam()
  }
}
```

### 2.自定義值

```ts
// in app.service.ts
@Module({
  imports: [DemoModule, UserModule],
  controllers: [AppController, DemoController],
  providers: [
    //AppService,下面是完整寫法
    {
      provide: 'ABC', // 自定義注入名稱，可在控制器使用，但就必須用@Inject('ABC')修飾來標注名稱
      useClass: AppService, //要注入的服務類別
    },
    // 自定義值
    {
      provide: 'test',
      useValue: ['William', 'Sandy', 'Emily'],
    },
  ],
})
export class AppModule {}
```

```ts
// in app.controller.ts
@Controller()
export class AppController {
  constructor(
    @Inject('ABC') private readonly appService: AppService,
    @Inject('test') private readonly englishLearn: string[]
  ) {}

  @Get('hello')
  getHello(): string[] {
    //return this.appService.getWilliam();
    return this.englishLearn
  }
}
```

### 3.自定義工廠

```ts
    AppService2,//先注入
    // 自定義工廠
    {
      provide: 'CCC',
      inject: [AppService2],//在這個模塊注入
      useFactory(appService2: AppService2) {//這邊就拿得到了
        console.log(appService2.getHello());
        return 123;
      },
    },
```

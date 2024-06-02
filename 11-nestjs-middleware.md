## nestjs-middleware

middleware 是路由處理程序之前調用的函數
middleware 可以執行以下任務

- 執行任何程式
- 對請求和回應物件進行修改
- 結束請求-回應流程
- 調用 stack 中下一個 middleware
- 如果當前 middleware 沒有結束請求-回應流程，必須調用 next() 來繼續下一個 middleware，否則請求將被掛起

### 1.針對特定模組

1.定義 middleware

```ts
import { Injectable, NestMiddleware } from '@nestjs/common'
import { Request, Response, NextFunction } from 'express'

@Injectable()
export class Logger implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('我來了 嘿嘿嘿')
    res.send('我被攔截了 嘿嘿嘿')
    //next();
  }
}
```

2.在 module 中引入 middleware

```ts
export class UserModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // consumer.apply(Logger).forRoutes('user');//針對路由
    // consumer.apply(Logger).forRoutes({ path: 'user', method: RequestMethod.GET });//針對特定路由
    consumer.apply(Logger).forRoutes(UserController) //針對控制器
  }
}
```

### 2.針對所有路由

1. 定義 middleware 同上
2. 在 main.ts 中引入 middleware

```ts
function MiddlewareAll(req: Request, res: Response, next: NextFunction) {
  console.log(req.originalUrl)
  next()
}

app.use(MiddlewareAll)
```

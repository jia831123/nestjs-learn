### 攔截器

NestJS 提供了一個 `Interceptor` 介面，用來攔截請求並對其進行處理，這些功能受(AOP)啟發，可以用來處理如：

- 函數執行之前/之後 綁定額外邏輯
- 轉換從函數返回結果
- 轉換從函數拋出的異常
- 擴展基本函數行為
- 根據所選條件完全重寫函數(如緩存)

### 範例：給凡回結果統一成一種格式

1. 建立一個 `ResultInterceptor` 類別，繼承 `NestInterceptor` 介面。

```ts
interface Data<T> {
  data: T
}
@Injectable()
export class Response<T> implements NestInterceptor {
  intercept(context, next: CallHandler): Observable<Data<T>> {
    return next.handle().pipe(
      map((data) => ({
        data,
        status: 200,
        message: 'ok',
        success: true,
      }))
    )
  }
}
```

2. 在 main 中 `app.useGlobalInterceptors()` 加入攔截器。

```ts
app.useGlobalInterceptors(new ResponseInterceptor())
```

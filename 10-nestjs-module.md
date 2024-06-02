## 10 NestJs Module

### 1.使用 nest g res [resource name] 時，nest 會幫我們自動在 appModule 引入

### 2.共享模塊

想要讓別人也用，用 exports 來分享模塊

```ts
// in UserModule
@Module({
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],
})

```

### 3.全域模塊

1.在該模塊上使用@Global()，記得也是要在 exports 裡面

```ts
@Global()
@Module({
  providers: [
    {
      provide: 'Config',
      useValue: { baseUrl: '/api' },
    },
  ],
  exports: [
    {
      provide: 'Config',
      useValue: { baseUrl: '/api' },
    },
  ],
})
export class ConfigModule {}
```

2.在 AppModule 上引入

```ts
@Module({
  imports: [DemoModule, UserModule, ListModule, ConfigModule],})
```

3.在其他模塊上使用

```ts
@Controller('list')
export class ListController {
  constructor(
    private readonly listService: ListService,
    @Inject('Config') private readonly base: any
  ) {}
  @Get()
  findAll() {
    return this.base
  }
}
```

### 4.動態模塊，可以傳參數

1.在模塊上使用中定義一個 static 方法，可以傳參數

```ts
@Global()
@Module({})
export class ConfigModule {
  static forRoot(options: Options): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'Config',
          useValue: { baseUrl: '/api' + options.path },
        },
      ],
      exports: [
        {
          provide: 'Config',
          useValue: { baseUrl: '/api' + options.path },
        },
      ],
    }
  }
}
```

2.在 AppModule 上，使用

```ts
@Module({
  imports: [
    DemoModule,
    UserModule,
    ListModule,
    ConfigModule.forRoot({
      path: '/william',
    }),
  ],})
```

3.結果，就會不同，

```ts
const before = { baseUrl: '/api/william' }
const after = { baseUrl: '/api' }
```

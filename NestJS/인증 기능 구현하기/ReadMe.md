### 인증 기능 구현을 위한 준비

참조: [https://www.inflearn.com/course/따라하는-네스트-제이에스/unit/87241?tab=curriculum](https://www.inflearn.com/course/%EB%94%B0%EB%9D%BC%ED%95%98%EB%8A%94-%EB%84%A4%EC%8A%A4%ED%8A%B8-%EC%A0%9C%EC%9D%B4%EC%97%90%EC%8A%A4/unit/87241?tab=curriculum)

- 모듈 구조
    
    AppModule
    
     - BoardModule
    
     - AuthModule
    
      - AuthController
    
      - UserEntity
    
      - AuthService
    
      - UserRepository
    
      - JWT, Passport
    

**CLI를 이용하여 모듈, 컨트롤러, 서비스 생성**

```jsx
nest g module auth // auth 모듈 생성
nest g controller auth -no-spec // auth 컨트롤러 생성
nest g service auth --no-spec // auth 서비스 생성
```

**User를 위한 Entity 생성**

유저 데이터를 위한 유저 Entity 생성

**user.entity.ts**

```jsx
@Entity()
export class User extends BaseEntity{
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    username: string;

    @Column()
    password: string;
}
```

**Repository 생성**

User Entitry를 생성, 수정, 삭제 등의 로직을 처리

**user.repository.ts**

```jsx
@EntityRepository(User)
export class UserRepository extends Repository<User>{
    
}
```

**생선된 User Repository를 다른 곳에서 사용 하도록 auth Module에서 imports 안에 UserRepositotry 추가**

**auth.module.ts**

```jsx
@Module({
  imports: [
    TypeOrmModule.forFeature([UserRepository])
  ],
  controllers: [AuthController],
  providers: [AuthService]
})
export class AuthModule {}
```

**Repsitory Injection**

User Repository를 auth Service 안에서 사용하도록 User Repository 넣어주기

**auth.service.ts**

```jsx
@Injectable()
export class AuthService {
    constructor(
        @InjectRepository(UserRepository)
        private userRepository: UserRepository
    ) {}
}
```

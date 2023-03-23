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
---
### 회원 가입 기능 구현

**user.repository.ts**

```jsx
@EntityRepository(User)
export class UserRepository extends Repository<User>{
    async createUser(authCredentialsDto: AuthCredentialsDto): Promise<void>{
        const {username, password} = authCredentialsDto;
        const user = this.create({ username, password });
        await this.save(user);
    }
}
```

**auth/dto/auth-credential.dto.ts**

```jsx
export class AuthCredentialsDto {
    username: string;
    password: string;
}
```

**auth.service.ts**

```jsx
@Injectable()
export class AuthService {
    constructor(
        @InjectRepository(UserRepository)
        private userRepository: UserRepository,
    ) {}

    async signUp(authCredentialsDto: AuthCredentialsDto): Promise<void> {
        return this.userRepository.createUser(authCredentialsDto);
    }
}
```

**auth.controller.ts**

```jsx
@Controller('auth')
export class AuthController {
    constructor( private authService:AuthService){}

    @Post('/signup')
    signUp(@Body() authCredentialsDto: AuthCredentialsDto): Promise<void> {
        return this.authService.signUp(authCredentialsDto);
    }
}
```
---
### 유저 데이터 유효성 체크

원하는 이름의 길이, 비밀번호 길이 같은 유효성 체크를 할 수 있게 각 Columnn에 맞는 조건을 넣어주기

**Class-validator 모듈 사용**

**auth-credentials.ts**

```jsx
export class AuthCredentialsDto {
    @IsString()
    @MinLength(4)
    @MaxLength(20)
    username: string;

    @IsString()
    @MinLength(4)
    @MaxLength(20)
    //영어, 숫자만 가능한 유효성 체크
    @Matches(/^[a-zA-Z0-9]*$/, {
        message: 'password only accepts english and number'
    })
    password: string;
}
```

**ValidationPipe**

요청이 컨트롤러에 있는 핸들러로 들어왔을 때 Dto에 있는 유효성 조건에 맞게 체크를 해주려면 ValidationPepe를 넣어주기

**auth.controller.ts**

```jsx
@Post('/signup')
signUp(@Body(ValidationPipe) authCredentialsDto: AuthCredentialsDto): Promise<void> {
    return this.authService.signUp(authCredentialsDto);
}
```
---
### 유저 이름에 유니크한 값 주기

1) repository 에서 findOne 메소드를 이용

2) database 레벨에서 같은 이름을 가진 유저가 있다면 에러 처리

**user.entity.ts**

```jsx
@Unique(['username'])
```

원하는 에러 처리 

**user.repository.ts**

```jsx
try {
    await this.save(user);
} catch (error) {
    if(error.code === '23505') {
        throw new ConflictException('Existing username')
    } else {
        throw new InternalServerErrorException();
    }
}
```

---

### 비밀번호 암호화 하기

**bcryptjs 모듈 사용 (암호화 모듈)**

```bash
$ npm install bcryptjs --save
```

1) 원본 비밀번호를 저장

2) 비밀번호 암호화 키 사용 (알고리즘 + 암호화 키)

3) SHA256 > Hash 저장 // 단방향, 복호화 불가

 / Hash 한 값을 다음에도 동일하게 hash 것과 동일한지 비교

5) 솔트(salt) + 비밀번호(Plain Password) 를 Hash 암호화 해서 처리

 예) 1234 ==⇒ salt_1234

        letmein ==⇒ salt_letmein

---

**user.repository.ts**

```jsx
const salt = await bcrypt.genSalt();
const hashedPassword = await bcrypt.hash(password, salt);

const user = this.create({ username, password:hashedPassword });
```

---

### 로그인 기능 구현하기

**user.service.ts**

```jsx
async signIn(authCredentialsDto: AuthCredentialsDto): Promise<string> {
	const { username, password } = authCredentialsDto;
	const user = await this.userRepository.findOne({username});

	if(user && (await bcrypt.compare(password, user.password))) {
		return 'login success';
	} else {
		throw new UnauthorizedException('login Failed')
	}
}
```

**user.controller.ts**

```jsx
@Post('/signin')
signIn(@Body(ValidationPipe) authCredentialsDto: AuthCredentialsDto) {
    return this.authService.signIn(authCredentialsDto);
}
```

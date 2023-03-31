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
---

### JWT에 대하여

JWT (JSON Web Token) : 정보를 안전하게 전할 때 혹은 유저의 권한 같은 것을 체크할 때 사용되는 유용한 모델

**JWT 구조**

1) Header

 토큰에 대한 메타 데이터 포함 (타입, 해싱 알고리즘 SHA256, RSA …)

2) Payload

 유저 정보나 만료 기간, 주제 등

3) Verify Signature

 토큰이 보낸 사람에 의해 서명되었으며 변경되지 않았는지 확인하는데 사용되는 서명.

**JWT 사용 흐름**

유저 로그인 → 토큰 생성 → 토큰 보관
토큰 생성 : <유저 정보 + (Hashing 알고리즘) + Secret Text>

**예제)**

Admin만 볼 수 있는 글을 보고자 할때 → 요청을 보낼때 보관하고 있던 Token을 Header에 넣어서 같이 보낸다 → 서버에서는 JWT를 이용해서 Token을 다시 생성한 후 두개를 비교. → 통과가되면 Admin 유저가 원하는 글을 볼 수 있도록 처리.

-> Client 에서 넘어온 Header + Client에서 넘어온 Payload  + Server에서 가지고 있는 Secret Text로 새로 생성해서 Header에 있던 Token의 Verify Signature와 비교. 일치하면 성공
---

### JWT를 이용해서 토큰 생성하기

**필요한 설치 모듈**

@nestjs/jwt : nestjs에서 jwt를 사용하기 위해 필요한 모듈

@nestjs/passport: nestjs에서 passport를 사용하기 위해 필요한 모듈

// passport > passport모듈

// passport-jwt > jwt 모듈

```bash
$ npm install @nestjs/jwt @nestjs/passport passport passport-jwt --save
```

1. **auth 모듈 imports에 넣어주기**

```jsx
@Module({
	imports: [
		PassportModule.register({ defaultStrategy: 'jwt'}),
		JwtModule.register({
			secret:'Secret1234',
			signOptions: {
				expiresIn: 60 * 60,
			}
		}),
	]
})
```
**로그인 성공 시 JWT를 이용해서 토큰 생성해주기 (auth.service.ts)**

```jsx
async signIn(authCredentialsDto: AuthCredentialsDto): Promise<{accessToken: string}> {
        const { username, password } = authCredentialsDto;
        const user = await this.userRepository.findOne({username});

        if(user && (await bcrypt.compare(password, user.password))){
            // 유저 토큰 생성 ( Secret + Payload )
            const payload = { username };
            const accessToken = await this.jwtService.sign(payload); // 알아서 payload + secret 처리로 토큰 생성

            return { accessToken };
        } else {
            throw new UnauthorizedException('login Failed')
        }
    }
```
---


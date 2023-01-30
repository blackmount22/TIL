NestJS
---

### 1. 기본 구조 설명

— eslintrc.js : 개발자들이 특정한 규칙을 가지고 규칙을 짤 수 있게 도와주는 라이브러리. 타입스크립트 가이드 라인 제시, 문법 에러 알림

— prettierrc : 코드의 포맷터 역할

— nest-cli.json : Nest JS 자체 필요한 형식

— src 폴더 : 대부분 Nest JS 로직, 소스

---

### 2. 흐름도

Request → Controller → Service → Controller return → Response 

- Nest JS 모듈이란
    
    @Module () 데코레이터로 주석이 달린 클래스, Nest 시작점
    
    // 객체 개념처럼 하나의 밀접한 관련된 내용을 모듈에 저장
    
    // 기본적으로 싱글 톤이므로 모듈간에 쉽게 공급자의 동일한 인스턴스를 공유 할 수 있다.
    
    // Module 내부에 Controller, Entity, Service, Repository 등 존재
    

ex) BoardModule

- BoardController + BoardEntitiy + BoardService + BoardRepository + ValidationPipe

### 모듈 생성하기

Board 모듈 생성 명령어

```bash
nest g module boards
// nest: using nestcli
// g: generate
// module : schematic that i want to create
// boards : name of the schematic
```

---

### NestJS Controllers 이란?

컨트롤러는 들어오는 요청을 처리하고 클라이언트에 응답을 반환하는 것.

컨트롤러는 @Controller 데코레이터로 클래스를 데코레이션하여 정의

Handler란?

핸드럴는 @Get, @Post, @Delete 등과 같은 데코레이터로 장식된 메소드

```bash
nest g controller boards --no-spec
// nest: using nestcli
// g: generate
// --no-spec: 테스트를 위한 소스 코드 생성 X
```

---

### NestJS Provider란

프로바이더의 주요 아이디어는 종속성으로 **주입**

(Service, Repository 등)

→ Provider 등록하기

: Module 파일에 Providers 항목안에 해당 모듈에서 사용하고자 하는 Provider 등록 추가

---

### NestJS Service 만들기

데이터베이스에서 데이터베이스 관련 로직 처리

```jsx
nest g service boards --no-spec
```

@Injectable : 다른 Controller나 다른 NestJS 에서 사용 가능

@Module > provider 에 자동 추가

- Dependency Injection
    
    Board Service를 Board Controller에서 이용할 수 있게 해주기 위해
    
    ```tsx
    @Controller('boards')
    export class BoardsController {
    	boardsService: BoardsService;
    
    	// boardsService 파라미터를 BoardsService 타입으로 지정
    	// this 에 private 접근 제한 boardsService를 사용할 수 있도록 지정
    	constructor(boardsService: BoardsService) {
    		this.boardsService = boardsService;
    	}
    }
    
    // 위 소스 간편화 처리
    // 접근 제한자를 생성자 파라미터에 선언하면 접근 제한자가 사용된 생성자 파라미터는 
    // 암묵적으로 클래스 프로퍼티로 선언된다.
    @Controller('boards')
    export class BoardsController {
    	constructor(private boardsService: BoardService) {}
    
    	getAllTask(){
    		this.boardsService
    	}
    }
    ```
    

---

### Service란?

Service 를 Controller 에서 이용할 수 있는 방법 (Dependency Injection) 종속성을 부여해야하는 작업 필요

→ 클래스의 Constructor 에서 이루어지는 처리

```jsx
@Controller('boards')
export class BoardsController{
	baordsService : BoardsService; 

	constructor(boardsService: BoardService) {
		this.boardsService = boardsService
	}
}

==>
@Controller('boards')
export class BoardsController{
	constructor(**private** **boardsService**: BoardsService) {}
}
// 생성자 파라미터는 암묵적으로 클래스 프로퍼티로 선언
// private이 붙으면 boardsService 가 해당 class 내에 바로 사용할 수 있는 프로퍼티로

// Java 기준으로 풀어서 코드 작성했을 경우
class A(int b){
 private int b;

	생성자 A(int b){
		this.b=b;
	}
}
```

---

### Model (boards.model.ts)

- Class를 이용하거나 Interface이용
- Interface: 변수의 타입만을 체크
- Class: 변수의 타입도 체크하고 인스턴스 또한 생성할 수 있음.

→ Board 공개글 / 비공개글 존재로 enum 사용

Model로 타입을 설정 후 return Type, 선언에 Model 타입을 명시 가능

→ 게시물에 관한 로직을 처리하는 곳은 Service

→ Service에서 처리 후 Controller 에서 호출

- 게시물 생성(Service)

```jsx
// 게시글 생성
  createBoard(title: string, description: string) {
      const board: Board = {
          id,
					// uuid로 임의의 유니크 아이디 생성
          title,
          // title: title -> 과 동일한 문법
          description,
          //description: description 
          status: BoardStatus.PUBLIC
      }
  }
```

uuid 모듈 사용

```jsx
npm install uuid
```

- 게시물 생성(Controller)

→ NestJS에서는 @Body body를 이용해서 데이터 넘겨받기

request 보내온 값을 가져오며, 하나씩 가져오려면 @Body(’title’) title 혹은 @Body(’description’) description 이런식으로 조회 처리

```jsx
@Post()
createBoad(@Body() body){
	console.log('body', body);
}
```

---

### DTO (Data Transfer Object) 란?

계층간 데이터 교환을 위한 객체이다.

데이터베이스에서 데이터를 얻어 Service, Controller로 전달할때 사용하는 객체

DTO를 쓰는 이유는

→ 데이터 유효성을 체크하는 효율적

→ 안정적인 코드로 사용된다.

Ex) Board의 경우 title, description property만 이용하지만, 실무에서 여러곳 + 많은 프로퍼티를 수정해야할때는 모두 다 수정이 필요하여, DTO를 사용하면 편리하다.

DTO는 Class로 작성, 클래스는 인터페이스와 다르게 런타임에서 작동하기 때문에 파이프 같은 기능을 이용할 때 더 유용

### DTO 적용하기

실제 Controller 와 Service에 적용

```jsx
const title = createBoardDto.title;
const {title} = createBoardDto;
// 동일한 문법으로 한 번에

const {title, description} = createBoardDto; // 로 처리 가능
```

---

### 게시물 CRUD

**1) ID 로 게시물 조회하기**

**Controller**

```jsx
@Get('/:id')
// 두개 이상 파라미터 인 경우 (@Param() params: string[])
getBoardByID(@Param('id') id: string) : Board {
    return this.boardsService.getBoardByID(id);
}
```

**Service**

```jsx
// 게시물 찾기
  getBoardByID(id: string) : Board{
      return this.boards.find((board) => board.id === id);
  }
```

**2) ID로 특정 게시물 삭제하기**

**Service**

```jsx
@Get('/:id')
// 두개 이상 파라미터 인 경우 (@Param() params: string[])
getBoardByID(@Param('id') id: string) : Board {
    return this.boardsService.getBoardByID(id);
}
```

**Controller**

```jsx
@Delete('/:id')
deleteBoard(@Param('id') id:string): void {
    this.boardsService.deleteBoard(id);
}
```
**3) 특정 게시물 업데이트하기** 

**Service**

```jsx
updateBoardStatus(id: string, status: BoardStatus): Board {
	const board = getBoadByID(id);
	board.status = status;
	return board;
}
```

**Controller**

```jsx
@Patch('/:id/status')
updateBoardStatus(
	@Param('id') id:string,
	@Body('status') status:BoardStatus,
) {
	return this.boardService.updateBoardStatus(id, status);
}
```

---

### NestJS Pipes

Pipe란 @injectable() 데코레이터로 주석이 달린 클래스

→ 파이프는 data transformation 과 data validation을 위해서 사용

→ 컨트롤러 경로 처리기에 의해 처리되는 인수에 대해 작동

clinet → Request → **Pipe (중간 가공 처리) → 성공 시,** Handler로 (Controller)

- Data Transformation
    
    ex) string ‘7’ ⇒ int 7로 변경
    
- Data validation ?
    
    ex) 만약 길이가 10자 이하여야 하는데 10자 이상되면 에러 처리로
    

**Pipe 사용 방법**

1) Handler-level Pipes

→ 모든 파라미터에 적용되는 pipe

2) Parameter-level Pipes

→ 각 파라미터에 적용 되는 pipie

3) Global-level Pipes

→ Client의 모든 요청에 적용되는 pipe, main.ts 에 넣어주면 된다.
---

### 유효성 체크 Pipe

1) class-validator, class-transformer 모듈을 추가 설치

```jsx
npm install class-validator class-transformer --save
```

```jsx
//DTO 에서 @IsNotEmpty() 로 빈 값 여부 체크
export class CreateBoardDto {
    @IsNotEmpty()
    title: string;

    @IsNotEmpty()
    description: string;
}

// Controller에서 등록이 필요
@Post()
@UsePipes(ValidationPipe)
createBoard(
    @Body() createBoardDto: CreateBoardDto
): Board {
    return this.boardsService.createBoard(createBoardDto);
}
```

### 특정 게시물을 찾을 때 없는 경우 결과 값 처리

```tsx
getBoardByID(id: string) : Board{
     const found = this.boards.find((board) => board.id === id);

     if(!found) {
        throw new NotFoundException(`Can't find Board with id ${id}`);
     }

     return found;
}
```

---

### 커스텀 파이프를 이용한 유효성 체크

- **커스텀 파이프 구현 방법**
    
    → PipeTransform 인터페이스를 구현해줘야한다.
    
    → transform() 메소드 // 첫번째 파라미터는 처리가 된 인자의 value, 두번째 파라미터는 인자에 대한 메타 데이터를 포함한 객체

---

### TypeORM이란?

TypeORM은 node.js에서 실행되고 TypeScript로 작성된 객체 관계형 매퍼 라이브러리이다.

### ORM(Object Relational Mapping) 이란?

- 객체와 관계형 데이터베이스의 데이터를 자동으로 변형 및 연결하는 작업이다.
- ORM을 이용한 개발은 객체와 데이터베이스의 변형에 유연하게 사용할 수 있다.

→ Object 객체와 <-> 관계형 DataBase 를 매핑

- **TypeORM vs Pure JavaScript**
    
    ```jsx
    // TypeORM
    const boards = Board.find({title:'Hello', status:'PUBLIC'});
    
    // Pure Javascript
    db.query(SELECT * FROM boards WHERE title="Hello" AND status="PUBLIC", (err, result) => 
    {
    	if(err) {
    		throw new Error('Error!')
    	}
    	boards = result.rows;
    })
    ```
    

### TypeORM 특징과 이점

 - 모델을 기반으로 데이터베이스 테이블 체계를 자동 생성

 - 데이터베이스에서 개체를 쉽게 삽입, 업데이트 및 삭제 할 수 있다.

 - 테이블 간 매핑(일대일, 일대 다 및 다 대다)을 만든다.

 - 간단한 CLI 명령을 제공한다.

 - TypeORM은 간단한 코딩으로 ORM 프레임 워크를 사용하기 쉽다.

 - TypeORM은 다른 모듈과 쉽게 통합된다.

---

### TypeORM 사용을 위한 모듈

1) @nestjs/typeorm // NestJS에서 TypeORM을 사용하기 위해 연동해주는 모듈

2) typeorm // TypeORM 모듈

3) pg // Postgres 모듈

// https://docs.nestjs.com/techniques/database

```jsx
npm install pg typeorm @nestjs/typeorm --save
```

- **TypeORM 애플리케이션에 연결하기**
    
    1) TypeORM 설정파일 생성 
    
     → src/configs/typeorm.configs.ts
    
    2) typeORM 설정파일 작성
    
    3) Root Module에서 Import 하기 (app.module.ts에 import)
    
    ```jsx
    @Module({
      imports: [
        TypeOrmModule.forRoot(typeORMConfig),
        BoardsModule],
    }) 
    ```
    

---

### Entity 데코레이터 (Board 클래스 기준)

 - @Entity() Board 클래스가 엔티티임을 나타내는데 사용 // CREATE TABLE Board 부분

 - @PrimaryGeneratedColumn() id 열이 기본키임을 나타내는 데 사용

 - @Column()

---

### Repository

 - 엔티티 개체와 함께 작동하며, 엔티티 찾기, 삽입, 업데이트 삭제 등을 처리

USER → Request → Controller → SERVICE → Repository(DB 관련 일)

---

### Service에 Repository 넣어주기 (Repository Injection)

```jsx
// Controller에서 Service 호출하듯
// Service 에서 Repository 똑같이 Injection 처리
constructor(
    @InjectRepository(BoardRepository)
    private boardRepository:BoardRepository
){}
```

---

### 게시물 생성하기

**board.service.ts**

```jsx
async createBoard(createBoardDTO: CreateBoardDto) : Promise<Board> {
	const {title, description} = createBoardDto;

	const board = this.boardRepository.create({
		title,
		description,
		status: BoardStatus.PUBLIC
	})

	await this.boardRepository.save(board);
	return board;
}
```

**board.controller.ts**

```jsx
@Post()
@UsePipes(ValidationPipe)
createBoard(@Body() createBoardDto: CreateBoardDto): Promise<Board> {
	return this.boardsService.createBoard(createBoardDto);
}
```

**board.repository.ts**

```jsx
@EntityRepository(Board)
export class BoardRepository extends Repository<Board> {
    async createBoard(createBoardDto: CreateBoardDto) : Promise<Board>{
        const {title, description} = createBoardDto;

        const board = this.create({
            title,
            description,
            status: BoardStatus.PUBLIC
        })

        await this.save(board);
        return board;
    }
}
```

repository.ts 에서 DB create를 처리하므로, service.ts는 아래와 같이 재 수정 필요

```jsx
createBoard(createBoardDto: CreateBoardDto) : Promise<Board>{
    return this.boardRepository.createBoard(createBoardDto);
}
```

---

### 게시물 삭제하기

 - remove() : 무조건 존재하는 아이템을 remove 메소드를 이용해서 지워야한다, 그러지 않으면 에러가 발생.

 - delete() : 만약 아이템이 존재하면 지우고 존재하지 않으면 아무런 영향이 없다.

**board.service.ts**

```jsx
async deleteBoard(id:number): Promise<void>{
	const result = await 
}
```

---
### 게시물 업데이트

**board.service.ts**

```jsx
async updateBoardStatus(id: number, status:BoardStatus): Promise<Board> {
	const board = await this.getBoardByID(id);
	
	board.status = status;
	await this.boardRepository.save(board);

	return board;
}
```

**board.controller.ts**

```jsx
@Patch('/:id/status')
updateBoardStatus{
	@Param('id', ParseIntPipe) id:number,
	@Body('status', BoardStatusValidationPipe) status: BoardStatus,
} {
	return this.boardService.updateBoardStatus(id, status);
}
```

---

### 모든 게시물 가져오기

**board.service.ts**

```jsx
async getAllBoards() : Promise<Board[]> {
	return this.boardRepository.
}
```

**board.controller.ts**

```jsx
@Get('/')
getAllBoards(): Promise<Board[]>{
	this.boardService.getAllBoards();
}
```

---

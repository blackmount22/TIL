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

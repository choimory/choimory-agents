# 백엔드 - TypeScript, NestJS, TypeORM 개발 규칙

## 적용대상

- TypeScript
- NestJS
- TypeORM

---

## 데이터 객체

- Entity와 DTO는 1:1로 매칭한다.
- DTO, 요청 객체, 응답 객체는 `readonly`를 사용하여 불변 객체 형태로 작성한다.
- 데이터 객체 간의 변환은 정적 팩토리 메소드를 선언하여 사용한다.
- 정적 팩토리 메소드는 변환 결과가 되는 데이터 객체의 함수로 작성하여, 해당 데이터 객체의 책임으로 한다.
- TypeScript에서는 Java의 Builder 패턴 대신 생성자 또는 객체 리터럴을 이용하여 객체를 생성한다.
- 단, Entity가 포함된 변환은 Entity에 DTO 변환 책임을 두지 않고 DTO, 요청 객체, 응답 객체 등 다른 데이터 객체에 변환 책임을 위임한다.
- 순환 참조 방지를 위해 하위 Entity 또는 DTO는 상위 Entity 또는 DTO를 포함하여 변환하지 않고, 본인 이하의 하위 Entity 또는 DTO만 포함하여 변환한다.
- Entity는 데이터 수정 작업을 제외하고는 조회 후 즉시 DTO로 변환하고 이후 비즈니스 로직에서는 사용하지 않는다.
- 비즈니스 로직 내에서는 Entity 대신 DTO를 도메인 데이터 객체로 활용한다.
- Entity 데이터 수정 작업은 Entity 내부 메소드로 작성한다.
- TypeORM은 JPA의 더티 체킹과 동일한 방식으로 변경사항을 자동 반영하지 않으므로 Entity 수정 후 Repository의 `save()`를 명시적으로 호출한다.
- 일부 필드만 조회하는 경우에도 Entity와 1:1로 매칭되는 DTO를 사용한다.
- 일부 필드 조회 시 조회하지 않은 DTO 필드는 `null`로 둘 수 있다.
- API 응답 형태가 다른 경우 DTO를 분리하지 않고 API별 Response 객체를 별도로 작성한다.
- API 응답에 여러 DTO, 집계값, 계산값 등이 필요한 경우 Service에서 이를 조합하여 Response 객체로 변환한다.

```typescript
// ============================================================
// Entity
// - Entity와 DTO는 1:1로 매칭한다.
// - Entity에는 DTO 변환 메소드를 작성하지 않는다.
// - Entity 수정은 Entity 내부 메소드로 처리한다.
// - 수정 후 Repository.save()를 호출한다.
// ============================================================

import {
  Column,
  Entity,
  PrimaryGeneratedColumn,
} from 'typeorm';

@Entity('post')
export class Post {

  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column()
  authorName: string;

  private constructor(
    title: string,
    content: string,
    authorName: string,
  ) {
    this.title = title;
    this.content = content;
    this.authorName = authorName;
  }

  static create(
    title: string,
    content: string,
    authorName: string,
  ): Post {
    return new Post(
      title,
      content,
      authorName,
    );
  }

  update(
    title: string,
    content: string,
  ): void {
    this.title = title;
    this.content = content;
  }
}
```

```typescript
// ============================================================
// DTO
// - Entity와 1:1 매칭한다.
// - Entity 조회 후 즉시 DTO로 변환하여 사용한다.
// - readonly를 사용하여 불변 객체 형태로 작성한다.
// - 일부 필드 조회 시 조회하지 않은 필드는 null로 둘 수 있다.
// ============================================================

export class PostDto {

  readonly id: number | null;
  readonly title: string | null;
  readonly content: string | null;
  readonly authorName: string | null;

  private constructor(params: {
    id: number | null;
    title: string | null;
    content: string | null;
    authorName: string | null;
  }) {
    this.id = params.id;
    this.title = params.title;
    this.content = params.content;
    this.authorName = params.authorName;
  }

  static from(post: Post): PostDto {
    return new PostDto({
      id: post.id,
      title: post.title,
      content: post.content,
      authorName: post.authorName,
    });
  }

  static listOf(
    id: number,
    title: string,
    authorName: string,
  ): PostDto {
    return new PostDto({
      id,
      title,
      content: null,
      authorName,
    });
  }
}
```

```typescript
// ============================================================
// Request
// - API 요청 객체는 readonly를 사용한다.
// - class-validator를 이용하여 요청 값을 검증한다.
// - Entity 생성이 필요한 경우 Request에서 Entity를 생성한다.
// ============================================================

import {
  IsInt,
  IsNotEmpty,
  IsString,
  MaxLength,
} from 'class-validator';

export class PostCreateRequest {

  @IsInt()
  readonly memberId: number;

  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  readonly title: string;

  @IsString()
  @IsNotEmpty()
  readonly content: string;

  @IsString()
  @IsNotEmpty()
  readonly authorName: string;

  constructor(params: {
    memberId: number;
    title: string;
    content: string;
    authorName: string;
  }) {
    this.memberId = params.memberId;
    this.title = params.title;
    this.content = params.content;
    this.authorName = params.authorName;
  }

  toEntity(): Post {
    return Post.create(
      this.title,
      this.content,
      this.authorName,
    );
  }
}
```

```typescript
// ============================================================
// Response
// - API 응답 객체는 readonly를 사용한다.
// - API 응답 형태별로 별도 Response를 작성한다.
// - DTO → Response 변환 책임은 Response가 가진다.
// ============================================================

export class PostCreateResponse {

  readonly id: number;
  readonly title: string;

  private constructor(params: {
    id: number;
    title: string;
  }) {
    this.id = params.id;
    this.title = params.title;
  }

  static from(postDto: PostDto): PostCreateResponse {
    return new PostCreateResponse({
      id: postDto.id!,
      title: postDto.title!,
    });
  }
}


export class PostDetailResponse {

  readonly id: number;
  readonly title: string;
  readonly content: string;
  readonly authorName: string;

  private constructor(params: {
    id: number;
    title: string;
    content: string;
    authorName: string;
  }) {
    this.id = params.id;
    this.title = params.title;
    this.content = params.content;
    this.authorName = params.authorName;
  }

  static from(postDto: PostDto): PostDetailResponse {
    return new PostDetailResponse({
      id: postDto.id!,
      title: postDto.title!,
      content: postDto.content!,
      authorName: postDto.authorName!,
    });
  }
}


export class PostListResponse {

  readonly id: number;
  readonly title: string;
  readonly authorName: string;

  private constructor(params: {
    id: number;
    title: string;
    authorName: string;
  }) {
    this.id = params.id;
    this.title = params.title;
    this.authorName = params.authorName;
  }

  static from(postDto: PostDto): PostListResponse {
    return new PostListResponse({
      id: postDto.id!,
      title: postDto.title!,
      authorName: postDto.authorName!,
    });
  }
}
```

---

# 레이어 호출

- Controller 함수는 하나의 Service 함수만 호출한다.
- Controller는 API 요청 객체를 받아 바로 Service로 전달하며, Service가 반환한 Response 객체를 반환한다.
- Controller는 API 요청 값에 대한 형식 및 기본 유효성 검증을 담당한다.
- NestJS의 `ValidationPipe`와 `class-validator`를 이용해 Request 객체의 기본 요청 값 검증을 수행한다.
- 데이터 존재 여부, 중복 여부, 권한, 상태 등 비즈니스 규칙에 대한 검증은 Service에서 Handler를 호출하여 수행한다.
- Service 함수는 하나의 API에 대한 전체 비즈니스 로직 흐름을 다루는 오케스트레이션을 담당한다.
- Service는 Request 객체를 전달받고 API Response 객체를 반환한다.
- Service는 Handler에서 반환받은 DTO 및 비즈니스 처리 결과를 조합하여 Response 객체로 변환한다.
- Service 함수는 로직 단위로 여러 Handler 함수를 호출할 수 있다.
- Service는 다른 도메인의 Handler도 호출할 수 있다.
- 하나의 API 흐름에 트랜잭션이 필요한 경우 Service가 트랜잭션 경계를 담당한다.
- TypeORM 기본 기능을 사용하는 경우 `DataSource.transaction()`을 이용한다.
- 여러 Handler가 동일한 트랜잭션에 참여해야 하는 경우 Service에서 생성된 `EntityManager`를 Handler에 전달한다.
- Handler 함수는 Service에서만 호출할 수 있다.
- Handler 함수 하나는 하나의 세부 로직만 담당한다.
- Handler는 필요 시 자기 도메인의 Repository를 통해 Entity를 조회, 저장, 수정, 삭제하고 DTO로 변환한다.
- Handler는 자기 도메인의 Repository만 접근한다.
- 다른 도메인의 데이터 처리가 필요한 경우 해당 도메인의 Handler를 호출하는 책임은 Service가 가진다.
- Repository는 데이터 접근 책임만 가진다.
- 일반 Repository 조회는 Entity를 반환한다.
- Custom Repository는 조회 목적에 따라 Entity 또는 DTO를 반환할 수 있다.

---

```typescript
// ============================================================
// Controller
// - Controller 함수는 하나의 Service 함수만 호출한다.
// - API 요청 값에 대한 기본 유효성 검증을 수행한다.
// - Service가 반환한 Response 객체를 그대로 반환한다.
// ============================================================

import {
  Body,
  Controller,
  Post,
} from '@nestjs/common';

@Controller('api/posts')
export class PostController {

  constructor(
    private readonly postService: PostService,
  ) {}

  @Post()
  createPost(
    @Body() request: PostCreateRequest,
  ): Promise<PostCreateResponse> {
    return this.postService.createPost(request);
  }
}
```

애플리케이션 전역에는 다음과 같이 `ValidationPipe`를 적용한다.

```typescript
// main.ts

import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  }),
);
```

---

```typescript
// ============================================================
// Service
// - Service 함수는 전체 비즈니스 로직 흐름을 담당한다.
// - 트랜잭션 경계를 Service에서 관리한다.
// - 여러 Handler를 호출한다.
// - Handler가 반환한 DTO 및 비즈니스 처리 결과를 조합하여
//   Response 객체를 반환한다.
// ============================================================

import {
  Injectable,
} from '@nestjs/common';

import {
  DataSource,
} from 'typeorm';

@Injectable()
export class PostService {

  constructor(
    private readonly dataSource: DataSource,

    private readonly memberValidateHandler:
      MemberValidateHandler,

    private readonly postValidateHandler:
      PostValidateHandler,

    private readonly postCreateHandler:
      PostCreateHandler,
  ) {}

  async createPost(
    request: PostCreateRequest,
  ): Promise<PostCreateResponse> {

    return this.dataSource.transaction(
      async (entityManager) => {

        // 1. 회원 검증
        await this.memberValidateHandler
          .validateWritableMember(
            request.memberId,
            entityManager,
          );

        // 2. 게시글 생성 검증
        await this.postValidateHandler
          .validateCreate(
            request,
            entityManager,
          );

        // 3. 게시글 생성
        const postDto =
          await this.postCreateHandler.create(
            request,
            entityManager,
          );

        // 4. 응답 객체 변환
        return PostCreateResponse.from(postDto);
      },
    );
  }
}
```

---

```typescript
// ============================================================
// Handler
// - Handler 함수는 Service에서만 호출한다.
// - Handler 함수 하나는 하나의 세부 로직만 담당한다.
// - Handler는 자기 도메인의 Repository만 접근한다.
// - 트랜잭션이 필요한 경우 Service로부터 EntityManager를 받는다.
// ============================================================

import {
  BadRequestException,
  Injectable,
  NotFoundException,
} from '@nestjs/common';

import {
  EntityManager,
} from 'typeorm';


@Injectable()
export class MemberValidateHandler {

  async validateWritableMember(
    memberId: number,
    entityManager: EntityManager,
  ): Promise<void> {

    const memberRepository =
      entityManager.getRepository(Member);

    const member =
      await memberRepository.findOne({
        where: {
          id: memberId,
        },
      });

    if (!member) {
      throw new NotFoundException(
        '회원을 찾을 수 없습니다.',
      );
    }

    if (member.isBlocked()) {
      throw new BadRequestException(
        '차단된 회원입니다.',
      );
    }
  }
}


@Injectable()
export class PostValidateHandler {

  async validateCreate(
    request: PostCreateRequest,
    entityManager: EntityManager,
  ): Promise<void> {

    const postRepository =
      entityManager.getRepository(Post);

    const existsTitle =
      await postRepository.exists({
        where: {
          title: request.title,
        },
      });

    if (existsTitle) {
      throw new BadRequestException(
        '이미 존재하는 제목입니다.',
      );
    }
  }
}


@Injectable()
export class PostCreateHandler {

  async create(
    request: PostCreateRequest,
    entityManager: EntityManager,
  ): Promise<PostDto> {

    const postRepository =
      entityManager.getRepository(Post);

    const post =
      request.toEntity();

    const savedPost =
      await postRepository.save(post);

    return PostDto.from(savedPost);
  }
}
```

---

# Repository

TypeORM에서는 Spring Data JPA의 `JpaRepository`처럼 Entity별 Repository 인터페이스를 반드시 선언하지 않는다.

기본 CRUD는 `Repository<Entity>`를 사용한다.

```typescript
// Module에서 Entity 등록

@Module({
  imports: [
    TypeOrmModule.forFeature([
      Post,
      Member,
    ]),
  ],

  controllers: [
    PostController,
  ],

  providers: [
    PostService,

    PostCreateHandler,
    PostValidateHandler,
    MemberValidateHandler,

    PostQueryRepository,
  ],
})
export class PostModule {}
```

트랜잭션이 필요하지 않은 Handler에서는 일반 Repository를 직접 주입할 수도 있다.

```typescript
@Injectable()
export class PostFindHandler {

  constructor(
    @InjectRepository(Post)
    private readonly postRepository:
      Repository<Post>,
  ) {}

  async findById(
    id: number,
  ): Promise<PostDto> {

    const post =
      await this.postRepository.findOne({
        where: {
          id,
        },
      });

    if (!post) {
      throw new NotFoundException(
        '게시글을 찾을 수 없습니다.',
      );
    }

    return PostDto.from(post);
  }
}
```

단, Service에서 시작한 하나의 트랜잭션에 여러 Handler를 참여시켜야 하는 경우에는 일반적으로 주입받은 Repository 대신 해당 트랜잭션의 `EntityManager`를 사용한다.

```typescript
entityManager.getRepository(Post);
```

---

# Entity 수정

TypeORM은 JPA의 영속성 컨텍스트 기반 더티 체킹과 동일하게 Entity 변경사항을 자동 반영하지 않는다.

따라서 Entity 내부 메소드로 값을 변경하고 Repository의 `save()`를 명시적으로 호출한다.

```typescript
@Injectable()
export class PostUpdateHandler {

  async update(
    postId: number,
    title: string,
    content: string,
    entityManager: EntityManager,
  ): Promise<PostDto> {

    const postRepository =
      entityManager.getRepository(Post);

    const post =
      await postRepository.findOne({
        where: {
          id: postId,
        },
      });

    if (!post) {
      throw new NotFoundException(
        '게시글을 찾을 수 없습니다.',
      );
    }

    // Entity 내부 메소드에서 상태 변경
    post.update(
      title,
      content,
    );

    // TypeORM에서는 명시적으로 저장
    const savedPost =
      await postRepository.save(post);

    return PostDto.from(savedPost);
  }
}
```

---

# Custom Repository

- 복잡한 조회는 Custom Repository 클래스로 분리한다.
- 단순 CRUD는 TypeORM `Repository<Entity>`를 사용한다.
- Custom Repository는 조회 목적에 따라 Entity 또는 DTO를 반환할 수 있다.
- 단순 목록이나 통계 조회처럼 Entity 전체를 생성할 필요가 없는 경우 DTO를 직접 반환할 수 있다.

```typescript
// ============================================================
// Custom Repository
// - 복잡한 조회를 담당한다.
// - 조회 목적에 따라 Entity 또는 DTO를 반환할 수 있다.
// ============================================================

import {
    Injectable,
} from '@nestjs/common';

import {
    DataSource,
} from 'typeorm';

@Injectable()
export class PostQueryRepository {

    constructor(
        private readonly dataSource: DataSource,
    ) {}

    async findPostList(): Promise<PostDto[]> {

        const rows =
            await this.dataSource
                .getRepository(Post)
                .createQueryBuilder('post')
                .select([
                    'post.id AS id',
                    'post.title AS title',
                    'post.authorName AS authorName',
                ])
                .getRawMany<{
                    id: number;
                    title: string;
                    authorName: string;
                }>();

        return rows.map(
            row =>
                PostDto.listOf(
                    row.id,
                    row.title,
                    row.authorName,
                ),
        );
    }
}
```

---

# 전체 호출 구조

```text
Controller
   │
   │ Request
   ▼
Service
   │
   ├── MemberValidateHandler
   │      └── Member Repository
   │
   ├── PostValidateHandler
   │      └── Post Repository
   │
   └── PostCreateHandler
          └── Post Repository
               │
               ▼
             Entity
               │
               ▼
              DTO
               │
               ▼
Service
   │
   │ DTO → Response
   ▼
Controller
   │
   ▼
Response
```

## 역할 정리

| Layer      | 역할                             |
| ---------- | ------------------------------ |
| Controller | HTTP 요청/응답 및 기본 입력 검증          |
| Service    | 하나의 API에 대한 전체 비즈니스 흐름 오케스트레이션 |
| Handler    | 하나의 세부 비즈니스 로직                 |
| Repository | 데이터 접근                         |
| Entity     | DB 매핑 및 Entity 자체 상태 변경        |
| DTO        | 내부 비즈니스 데이터 전달                 |
| Request    | API 입력 데이터                     |
| Response   | API 출력 데이터                     |

## 핵심 원칙

```text
Controller
    ↓
Service
    ↓
Handler
    ↓
Repository
    ↓
Entity
```

조회 결과는 가능한 빠르게 DTO로 변환한다.

```text
Repository
    ↓
Entity
    ↓
DTO
```

비즈니스 로직에서는 가능한 Entity를 직접 전달하지 않는다.

```text
Service ↔ Handler

DTO 중심
```

Entity 변경이 필요한 경우에만 Handler 내부에서 Entity를 사용한다.

```text
Handler
    ↓
Entity 조회
    ↓
Entity.update()
    ↓
Repository.save()
    ↓
DTO 변환
```

Service는 여러 Handler를 조합하여 하나의 API 전체 흐름을 구성한다.

```text
Controller
    ↓
Service
    ├─ Handler A
    ├─ Handler B
    ├─ Handler C
    └─ Response 생성
```

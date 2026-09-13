## 백엔드 - Java, Springboot, JPA 개발 규칙

### 적용대상

- Java
- Springboot
- JPA

### 데이터 객체

- Entity와 DTO는 1:1로 매칭한다.
- DTO, 요청 객체, 응답 객체는 불변으로 작성한다.
- 데이터 객체 간의 변환은 정적 팩토리 메소드를 선언하여 사용한다.
- 정적 팩토리 메소드는 변환 결과가 되는 데이터 객체의 함수로 작성하여, 해당 데이터 객체의 책임으로 한다.
- DTO, 요청 객체, 응답 객체의 정적 팩토리 메소드는 Builder 패턴을 이용하여 작성한다.
- 단, Entity가 포함된 변환은 Entity에 정적 팩토리 메소드를 작성하지 않고, DTO, 요청 객체, 응답 객체 등 다른 데이터 객체에 변환 책임을 위임한다.
- 순환 참조 방지를 위해 하위 Entity 또는 DTO는 상위 Entity 또는 DTO를 포함하여 변환하지 않고, 본인 이하의 하위 Entity 또는 DTO만 포함하여 변환한다.
- Entity는 데이터 수정 작업을 제외하고는 조회 후 즉시 DTO로 변환하고 폐기한다.
- 비즈니스 로직 내에서는 Entity 대신 DTO를 도메인 데이터 객체로 활용한다.
- Entity 데이터 수정 작업은 Entity 내부 함수로 작성하고, JPA 더티 체킹을 통해 반영한다.
- 일부 필드만 조회하는 경우에도 Entity와 1:1로 매칭되는 DTO를 사용한다.
- 일부 필드 조회 시 조회하지 않은 DTO 필드는 null로 둘 수 있다.
- API 응답 형태가 다른 경우 DTO를 분리하지 않고, API별 응답 객체를 별도로 작성한다.
- API 응답에 여러 DTO, 집계값, 계산값 등이 필요한 경우 Service에서 이를 조합하여 응답 객체로 변환한다.

```java
// ============================================================
// Entity
// - Entity와 DTO는 1:1로 매칭한다.
// - Entity에는 DTO 변환 메소드를 작성하지 않는다.
// - Entity 수정은 Entity 내부 함수로 처리한다.
// ============================================================

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // 게시글 ID

    private String title; // 제목
    private String content; // 내용
    private String authorName; // 작성자명

    private Post(
            String title,
            String content,
            String authorName
    ) {
        this.title = title;
        this.content = content;
        this.authorName = authorName;
    }

    public static Post create(
            String title,
            String content,
            String authorName
    ) {
        return new Post(
                title,
                content,
                authorName
        );
    }

    public void update(
            String title,
            String content
    ) {
        this.title = title;
        this.content = content;
    }
}

// ============================================================
// DTO
// - Entity와 1:1 매칭한다.
// - Entity 조회 후 즉시 DTO로 변환하여 사용한다.
// - 정적 팩토리 메소드는 Builder 패턴을 이용하여 작성한다.
// - 일부 필드 조회 시 조회하지 않은 필드는 null로 둘 수 있다.
// ============================================================

@Getter
@Builder(toBuilder = true)
public final class PostDto {

    private final Long id;
    private final String title;
    private final String content;
    private final String authorName;

    public static PostDto from(Post post) {
        return PostDto.builder()
                .id(post.getId())
                .title(post.getTitle())
                .content(post.getContent())
                .authorName(post.getAuthorName())
                .build();
    }

    public static PostDto listOf(
            Long id,
            String title,
            String authorName
    ) {
        return PostDto.builder()
                .id(id)
                .title(title)
                .content(null)
                .authorName(authorName)
                .build();
    }
}

// ============================================================
// Request
// - API 요청 객체는 불변으로 작성한다.
// - 정적 팩토리 메소드는 Builder 패턴을 이용하여 작성한다.
// - Entity 생성이 필요한 경우 Request에서 Entity를 생성한다.
// ============================================================

@Getter
@Builder(toBuilder = true)
public final class PostCreateRequest {

    @NotNull
    private final Long memberId;

    @NotBlank
    @Size(max = 100)
    private final String title;

    @NotBlank
    private final String content;

    @NotBlank
    private final String authorName;

    @JsonCreator
    public PostCreateRequest(
            @JsonProperty("memberId") Long memberId,
            @JsonProperty("title") String title,
            @JsonProperty("content") String content,
            @JsonProperty("authorName") String authorName
    ) {
        this.memberId = memberId;
        this.title = title;
        this.content = content;
        this.authorName = authorName;
    }

    public Post toEntity() {
        return Post.create(
                title,
                content,
                authorName
        );
    }
}

// ============================================================
// Response
// - API 응답 객체는 불변으로 작성한다.
// - API 응답 형태별로 별도 Response를 작성한다.
// - DTO → Response 변환 책임은 Response가 가진다.
// - 정적 팩토리 메소드는 Builder 패턴을 이용하여 작성한다.
// ============================================================

@Getter
@Builder(toBuilder = true)
public final class PostCreateResponse {

    private final Long id;
    private final String title;

    public static PostCreateResponse from(PostDto postDto) {
        return PostCreateResponse.builder()
                .id(postDto.getId())
                .title(postDto.getTitle())
                .build();
    }
}

@Getter
@Builder(toBuilder = true)
public final class PostDetailResponse {

    private final Long id;
    private final String title;
    private final String content;
    private final String authorName;

    public static PostDetailResponse from(PostDto postDto) {
        return PostDetailResponse.builder()
                .id(postDto.getId())
                .title(postDto.getTitle())
                .content(postDto.getContent())
                .authorName(postDto.getAuthorName())
                .build();
    }
}

@Getter
@Builder(toBuilder = true)
public final class PostListResponse {

    private final Long id;
    private final String title;
    private final String authorName;

    public static PostListResponse from(PostDto postDto) {
        return PostListResponse.builder()
                .id(postDto.getId())
                .title(postDto.getTitle())
                .authorName(postDto.getAuthorName())
                .build();
    }
}
```

### 레이어 호출

- Controller 함수는 하나의 Service 함수만 호출한다.
- Controller는 API 요청 객체를 받아 바로 Service로 건네주며, Service가 반환한 응답 객체를 반환한다.
- Controller는 API 요청 값에 대한 형식 및 기본 유효성 검증을 담당한다.
- Controller에서 Spring Validation의 `@Valid` 또는 `@Validated`를 이용해, 요청 객체의 기본 요청 값 검증을 수행한다.
- 데이터 존재 여부, 중복 여부, 권한, 상태 등 비즈니스 규칙에 대한 검증은 Service에서 호출하고, Handler에서 수행한다.
- Service 함수는 하나의 API에 대한 전체 비즈니스 로직 흐름을 다루는 오케스트레이션을 담당한다.
- Service는 요청 객체를 건네받고, API 응답 객체를 반환한다.
- Service는 Handler에서 반환받은 DTO 및 비즈니스 처리 결과를 조합하여 응답 객체로 변환한다.
- Service 함수는 로직 단위로 여러 Handler 함수를 호출할 수 있다.
- Service는 다른 도메인의 Handler도 호출할 수 있다.
- Service 함수가 `@Transactional`을 담당한다.
- Handler 함수는 Service에서만 호출할 수 있다.
- Handler 함수 하나는 하나의 세부 로직만 담당한다.
- Handler는 필요시 도메인의 Repository를 호출하여 Entity를 조회, 저장, 수정, 삭제하고 DTO로 변환한다.
- Handler는 자기 도메인의 Repository만 호출할 수 있다.
- Repository는 데이터 접근 책임만 가진다.
- Repository는 Entity를 반환한다.
- 커스텀 Repository는 조회 목적에 따라 Entity 또는 DTO를 반환할 수 있다.

```java
// ============================================================
// Controller
// - Controller 함수는 하나의 Service 함수만 호출한다.
// - API 요청 값에 대한 기본 유효성 검증을 수행한다.
// - Service가 반환한 응답 객체를 그대로 반환한다.
// ============================================================

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/posts")
@Validated
public class PostController {

    private final PostService postService;

    @PostMapping
    public ResponseEntity<PostCreateResponse> createPost(
            @Valid @RequestBody PostCreateRequest request
    ) {
        return ResponseEntity.ok(
                postService.createPost(request)
        );
    }
}

// ============================================================
// Service
// - Service 함수는 전체 비즈니스 로직 흐름을 담당한다.
// - Service 함수가 @Transactional을 담당한다.
// - 비즈니스 검증이 필요한 Handler를 호출한다.
// - 여러 Handler를 호출하고, DTO 및 비즈니스 처리 결과를
//   조합하여 응답 객체를 반환한다.
// ============================================================

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class PostService {

    private final MemberValidateHandler memberValidateHandler; // 다른 도메인 Handler
    private final PostValidateHandler postValidateHandler;
    private final PostCreateHandler postCreateHandler;

    @Transactional
    public PostCreateResponse createPost(PostCreateRequest request) {
        // 1. 회원 검증
        memberValidateHandler.validateWritableMember(
                request.getMemberId()
        );

        // 2. 게시글 생성 검증
        postValidateHandler.validateCreate(request);

        // 3. 게시글 생성
        PostDto postDto = postCreateHandler.create(request);

        // 4. 응답 객체 변환
        return PostCreateResponse.from(postDto);
    }
}

// ============================================================
// Handler
// - Handler 함수는 Service에서만 호출한다.
// - Handler 함수 하나는 하나의 세부 로직만 담당한다.
// - Handler는 자기 도메인의 Repository만 호출한다.
// ============================================================

@Component
@RequiredArgsConstructor
public class MemberValidateHandler {

    private final MemberRepository memberRepository;

    public void validateWritableMember(Long memberId) {
        Member member = memberRepository.findById(memberId)
                .orElseThrow(
                        () -> new IllegalArgumentException(
                                "회원을 찾을 수 없습니다."
                        )
                );

        if (member.isBlocked()) {
            throw new IllegalArgumentException(
                    "차단된 회원입니다."
            );
        }
    }
}

@Component
@RequiredArgsConstructor
public class PostValidateHandler {

    private final PostRepository postRepository;

    public void validateCreate(PostCreateRequest request) {
        boolean existsTitle =
                postRepository.existsByTitle(request.getTitle());

        if (existsTitle) {
            throw new IllegalArgumentException(
                    "이미 존재하는 제목입니다."
            );
        }
    }
}

@Component
@RequiredArgsConstructor
public class PostCreateHandler {

    private final PostRepository postRepository;

    public PostDto create(PostCreateRequest request) {
        Post post = request.toEntity();

        Post savedPost = postRepository.save(post);

        return PostDto.from(savedPost);
    }
}

// ============================================================
// Repository
// - Repository는 데이터 접근 책임만 가진다.
// - Repository는 Entity를 반환한다.
// ============================================================

public interface PostRepository
        extends JpaRepository<Post, Long> {

    boolean existsByTitle(String title);
}

public interface MemberRepository
        extends JpaRepository<Member, Long> {
}

// ============================================================
// Custom Repository
// - 조회 목적에 따라 Entity 또는 DTO를 반환할 수 있다.
// ============================================================

@Repository
@RequiredArgsConstructor
public class PostQueryRepository {

    private final JPAQueryFactory queryFactory;

    public List<PostDto> findPostList() {
        return queryFactory
                .select(Projections.constructor(
                        PostListProjection.class,
                        post.id,
                        post.title,
                        post.authorName
                ))
                .from(post)
                .fetch()
                .stream()
                .map(PostListProjection::toDto)
                .toList();
    }
}
```
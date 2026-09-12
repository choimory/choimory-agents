## 주석 규칙

### 적용대상

- Javascript, TypeScript
- Java, Kotlin

### Document Comment

- 모든 객체에는 클래스의 용도를 설명하는 Doc Comment 주석을 작성한다.
- 모든 객체의 필드에는 필드 측면에 필드명을 설명하는 간략한 주석을 작성한다.
- 모든 함수에는 함수의 용도, 파라미터, 리턴 값, 발생 예외 등을 설명하는 Doc Comment 주석을 작성한다.
- 함수 내 로직이나 다른 함수 호출 등의 로직을 흐름 단위로 분리하여, 흐름 별로 간단한 한글 주석을 작성한다.
- 하위 주석 양식은 코드 컨벤션과는 관련이 없음. 주석 방식만 참고.

```java
/**
 * 사용자 정보 관련 객체
 */
public class User {
    private Long id;          // 사용자 고유 ID
    private String name;      // 사용자 이름
    private String email;     // 사용자 이메일 주소
    private boolean active;   // 사용자 활성화 여부
}

/**
 * 사용자 회원가입, 조회, 수정, 삭제 기능을 제공하는 서비스 클래스입니다.
 * 사용자 관련 비즈니스 로직을 처리하고 Repository와 연동합니다.
 */
public class UserService {
    private final UserRepository userRepository; // 사용자 데이터 접근 객체
    private final PasswordEncoder passwordEncoder; // 비밀번호 암호화 객체

    /**
     * 사용자 ID를 기준으로 사용자 정보를 조회합니다.
     *
     * @param userId 조회할 사용자의 고유 ID
     * @return 조회된 사용자 정보
     * @throws UserNotFoundException 사용자를 찾을 수 없는 경우 발생
     */
    public UserResponse findUserById(Long userId) {
        // 사용자 ID를 기준으로 사용자 엔티티를 조회한다.
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException("사용자를 찾을 수 없습니다."));

        // 조회된 사용자 엔티티를 응답 객체로 변환한다.
        return UserResponse.from(user);
    }

    /**
     * 신규 사용자를 생성합니다.
     *
     * @param request 사용자 생성 요청 정보
     * @return 생성된 사용자 정보
     * @throws DuplicateEmailException 이미 사용 중인 이메일인 경우 발생
     */
    public UserResponse createUser(UserCreateRequest request) {
        // 이메일 중복 여부를 확인한다.
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateEmailException("이미 사용 중인 이메일입니다.");
        }

        // 비밀번호를 암호화한다.
        String encodedPassword = passwordEncoder.encode(request.getPassword());

        // 요청 정보와 암호화된 비밀번호를 기반으로 사용자 엔티티를 생성한다.
        User user = new User(
                request.getName(),
                request.getEmail(),
                encodedPassword
        );

        // 생성된 사용자 엔티티를 저장한다.
        User savedUser = userRepository.save(user);

        // 저장된 사용자 엔티티를 응답 객체로 변환한다.
        return UserResponse.from(savedUser);
    }
}
```
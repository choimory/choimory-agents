## 주석 규칙

### 적용대상

- JavaScript
- TypeScript
- React
- Next.js

### Document Comment

- 모든 컴포넌트에는 컴포넌트의 용도를 설명하는 Doc Comment 주석을 작성한다.
- 모든 타입(Type), 인터페이스(Interface)에는 해당 객체의 용도를 설명하는 Doc Comment 주석을 작성한다.
- 타입 또는 인터페이스의 모든 필드에는 필드 측면에 필드의 용도를 설명하는 간략한 주석을 작성한다.
- 모든 함수 및 Custom Hook에는 함수의 용도, 파라미터, 리턴 값, 발생 가능한 예외 등을 설명하는 Doc Comment 주석을 작성한다.
- 컴포넌트 내부의 주요 상태(State), Ref 등은 용도를 명확하게 알기 어려운 경우 간략한 주석을 작성한다.
- 함수 또는 컴포넌트 내부의 로직이나 다른 함수 호출 등의 로직을 흐름 단위로 분리하여, 흐름별로 간단한 한글 주석을 작성한다.
- 단순한 값 할당, 명확한 JSX 렌더링 등 코드 자체로 의미를 충분히 파악할 수 있는 부분에는 불필요한 주석을 작성하지 않는다.
- 하위 주석 양식은 코드 컨벤션과는 관련이 없음. 주석 방식만 참고한다.

```typescript
/**
 * 사용자 정보를 나타내는 타입
 */
export interface User {
    id: number;         // 사용자 고유 ID
    name: string;       // 사용자 이름
    email: string;      // 사용자 이메일 주소
    active: boolean;    // 사용자 활성화 여부
}

/**
 * 사용자 상세 컴포넌트의 Props
 */
interface UserDetailProps {
    userId: number;     // 조회할 사용자 고유 ID
}

/**
 * 사용자 정보를 조회하여 화면에 표시하는 컴포넌트입니다.
 * 사용자 ID를 기준으로 사용자 정보를 조회하고 로딩 및 오류 상태를 처리합니다.
 *
 * @param props 컴포넌트 Props
 * @returns 사용자 상세 화면
 */
export function UserDetail({ userId }: UserDetailProps) {
    const [user, setUser] = useState<User | null>(null); // 조회된 사용자 정보
    const [loading, setLoading] = useState(false);        // 사용자 정보 조회 상태
    const [error, setError] = useState<string | null>(null); // 조회 오류 메시지

    useEffect(() => {
        /**
         * 사용자 정보를 조회합니다.
         *
         * @returns 사용자 정보 조회 완료 Promise
         */
        const fetchUser = async (): Promise<void> => {
            // 사용자 정보 조회를 시작한다.
            setLoading(true);
            setError(null);

            try {
                // 사용자 ID를 기준으로 사용자 정보를 조회한다.
                const response = await getUser(userId);

                // 조회된 사용자 정보를 상태에 저장한다.
                setUser(response);
            } catch (error) {
                // 사용자 정보 조회 중 발생한 오류를 처리한다.
                setError('사용자 정보를 조회할 수 없습니다.');
            } finally {
                // 사용자 정보 조회 상태를 종료한다.
                setLoading(false);
            }
        };

        fetchUser();
    }, [userId]);

    // 사용자 정보를 조회 중인 경우 로딩 화면을 표시한다.
    if (loading) {
        return <Loading />;
    }

    // 사용자 정보 조회 중 오류가 발생한 경우 오류 화면을 표시한다.
    if (error) {
        return <ErrorMessage message={error} />;
    }

    // 사용자 정보가 존재하지 않는 경우 빈 화면을 표시한다.
    if (!user) {
        return null;
    }

    // 조회된 사용자 정보를 화면에 표시한다.
    return (
        <div>
            <h1>{user.name}</h1>
            <span>{user.email}</span>
        </div>
    );
}

/**
 * 사용자 ID를 기준으로 사용자 정보를 조회합니다.
 *
 * @param userId 조회할 사용자의 고유 ID
 * @returns 조회된 사용자 정보
 * @throws Error 사용자 정보 조회 요청에 실패한 경우 발생
 */
export async function getUser(userId: number): Promise<User> {
    // 사용자 조회 API를 호출한다.
    const response = await fetch(`/api/users/${userId}`);

    // API 요청 실패 여부를 확인한다.
    if (!response.ok) {
        throw new Error('사용자 정보 조회에 실패했습니다.');
    }

    // 응답 데이터를 사용자 객체로 변환하여 반환한다.
    return response.json();
}

/**
 * 사용자 목록 조회 기능을 제공하는 Custom Hook입니다.
 * 사용자 목록과 조회 상태, 오류 정보를 관리합니다.
 *
 * @returns 사용자 목록 조회 관련 상태 및 기능
 */
export function useUsers() {
    const [users, setUsers] = useState<User[]>([]);       // 사용자 목록
    const [loading, setLoading] = useState(false);        // 사용자 목록 조회 상태
    const [error, setError] = useState<string | null>(null); // 조회 오류 메시지

    /**
     * 사용자 목록을 조회합니다.
     *
     * @returns 사용자 목록 조회 완료 Promise
     */
    const fetchUsers = async (): Promise<void> => {
        // 사용자 목록 조회를 시작한다.
        setLoading(true);
        setError(null);

        try {
            // 사용자 목록 조회 API를 호출한다.
            const response = await getUsers();

            // 조회된 사용자 목록을 상태에 저장한다.
            setUsers(response);
        } catch (error) {
            // 사용자 목록 조회 중 발생한 오류를 처리한다.
            setError('사용자 목록을 조회할 수 없습니다.');
        } finally {
            // 사용자 목록 조회 상태를 종료한다.
            setLoading(false);
        }
    };

    return {
        users,
        loading,
        error,
        fetchUsers,
    };
}
```
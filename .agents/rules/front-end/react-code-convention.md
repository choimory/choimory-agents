## 프론트엔드 - Next.js, React 개발 규칙

### 적용대상

- Next.js
- React

### 기본 구조

- 화면은 `page`, `container`, `component` 단위로 분리한다.
- `page`는 라우팅 진입점 역할만 담당한다.
- `container`는 API 호출, 상태 관리, 이벤트 흐름 등 화면의 로직을 담당한다.
- `component`는 UI 렌더링만 담당한다.
- 공통 UI는 `components/common` 또는 `shared/ui`에 작성한다.
- 도메인별 화면과 컴포넌트는 도메인 폴더 내부에 작성한다.
- 하나의 컴포넌트는 하나의 주요 책임만 가진다.
- 컴포넌트가 커지면 UI 단위 또는 로직 단위로 분리한다.

```
src
├── app
│   └── posts
│       └── page.tsx
│
├── features
│   └── post
│       ├── api
│       │   └── postApi.ts
│       ├── model
│       │   └── postTypes.ts
│       ├── container
│       │   └── PostCreateContainer.tsx
│       ├── components
│       │   └── PostCreateForm.tsx
│       └── hooks
│           └── usePostCreate.ts
│
└── shared
    ├── api
    ├── ui
    └── utils
```

---

### 데이터 객체

- API 요청 타입과 응답 타입은 명확하게 분리한다.
- API 응답 타입을 화면 컴포넌트에서 직접 변경하지 않는다.
- 화면에서 사용하는 데이터 형태가 API 응답과 다르면 ViewModel 타입을 별도로 작성한다.
- API 응답 객체를 ViewModel로 변환하는 함수는 별도 mapper 함수로 작성한다.
- 컴포넌트 props 타입은 해당 컴포넌트 파일 내부 또는 가까운 위치에 작성한다.
- 도메인 공통 타입은 `model` 또는 `types` 파일에 작성한다.
- `any` 사용은 금지하고, 불가피하면 `unknown` 사용 후 타입 가드를 작성한다.

```tsx
// postTypes.ts

export type PostCreateRequest = {
  title: string;
  content: string;
};

export type PostCreateResponse = {
  id: number;
  title: string;
};

export type PostDetailResponse = {
  id: number;
  title: string;
  content: string;
  authorName: string;
};

export type PostViewModel = {
  id: number;
  title: string;
  content: string;
  author: string;
};

export const toPostViewModel = (
  response: PostDetailResponse
): PostViewModel => ({
  id: response.id,
  title: response.title,
  content: response.content,
  author: response.authorName,
});
```

---

### API 호출

- API 호출 함수는 컴포넌트 내부에 직접 작성하지 않는다.
- API 호출은 도메인별 `api` 파일에 작성한다.
- API 함수는 요청 객체를 받고 응답 객체를 반환한다.
- HTTP client 설정은 공통 모듈에서 관리한다.
- API 에러는 공통 에러 처리 규칙을 따른다.
- API URL 문자열은 컴포넌트에 직접 작성하지 않는다.
- React Query를 사용하는 경우 query key는 도메인별 상수로 관리한다.

```tsx
// postApi.ts

import { apiClient } from '@/shared/api/apiClient';
import type {
  PostCreateRequest,
  PostCreateResponse,
  PostDetailResponse,
} from '../model/postTypes';

export const postApi = {
  createPost: async (
    request: PostCreateRequest
  ): Promise<PostCreateResponse> => {
    const { data } = await apiClient.post<PostCreateResponse>(
      '/api/posts',
      request
    );

    return data;
  },

  getPost: async (postId: number): Promise<PostDetailResponse> => {
    const { data } = await apiClient.get<PostDetailResponse>(
      `/api/posts/${postId}`
    );

    return data;
  },
};
```

---

### 컴포넌트

- 컴포넌트는 가능한 한 props를 통해 데이터를 전달받는다.
- UI 컴포넌트는 API 호출을 직접 수행하지 않는다.
- UI 컴포넌트는 전역 상태에 직접 의존하지 않는다.
- 이벤트 함수는 `handle` 접두사를 사용한다.
- boolean props는 `is`, `has`, `can`, `should` 접두사를 사용한다.
- 컴포넌트 파일명은 PascalCase로 작성한다.
- hook 파일명은 `use` 접두사를 사용한다.
- 하나의 파일에는 하나의 주요 컴포넌트만 작성한다.

```tsx
// PostCreateForm.tsx

type PostCreateFormProps = {
  title: string;
  content: string;
  isSubmitting: boolean;
  onChangeTitle: (title: string) => void;
  onChangeContent: (content: string) => void;
  onSubmit: () => void;
};

export function PostCreateForm({
  title,
  content,
  isSubmitting,
  onChangeTitle,
  onChangeContent,
  onSubmit,
}: PostCreateFormProps) {
  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        onSubmit();
      }}
    >
      <input
        value={title}
        onChange={(event) => onChangeTitle(event.target.value)}
      />

      <textarea
        value={content}
        onChange={(event) => onChangeContent(event.target.value)}
      />

      <button type="submit" disabled={isSubmitting}>
        등록
      </button>
    </form>
  );
}
```

---

### Container / Hook

- Container는 화면의 데이터 흐름과 이벤트 흐름을 담당한다.
- 복잡한 상태와 이벤트 로직은 custom hook으로 분리한다.
- custom hook은 상태, 이벤트 함수, 비동기 처리 결과를 반환한다.
- custom hook은 UI를 반환하지 않는다.
- Container는 hook 결과를 UI 컴포넌트에 props로 전달한다.

```tsx
// usePostCreate.ts

import { useState } from 'react';
import { postApi } from '../api/postApi';

export function usePostCreate() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async () => {
    setIsSubmitting(true);

    try {
      await postApi.createPost({
        title,
        content,
      });
    } finally {
      setIsSubmitting(false);
    }
  };

  return {
    title,
    content,
    isSubmitting,
    setTitle,
    setContent,
    handleSubmit,
  };
}
```

```tsx
// PostCreateContainer.tsx

import { PostCreateForm } from '../components/PostCreateForm';
import { usePostCreate } from '../hooks/usePostCreate';

export function PostCreateContainer() {
  const {
    title,
    content,
    isSubmitting,
    setTitle,
    setContent,
    handleSubmit,
  } = usePostCreate();

  return (
    <PostCreateForm
      title={title}
      content={content}
      isSubmitting={isSubmitting}
      onChangeTitle={setTitle}
      onChangeContent={setContent}
      onSubmit={handleSubmit}
    />
  );
}
```

---

### Page

- `page.tsx`는 가능한 얇게 유지한다.
- `page.tsx`에서는 화면 Container를 호출한다.
- page에서 직접 API 호출, 복잡한 상태 관리, 이벤트 로직을 작성하지 않는다.
- Next.js 서버 컴포넌트에서 필요한 초기 데이터 조회는 page 또는 server action에서 수행할 수 있다.
- 클라이언트 상호작용이 필요한 영역은 Client Component로 분리한다.

```tsx
// app/posts/create/page.tsx

import { PostCreateContainer } from '@/features/post/container/PostCreateContainer';

export default function PostCreatePage() {
  return <PostCreateContainer />;
}
```
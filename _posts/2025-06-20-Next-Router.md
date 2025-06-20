---
title: "Next.js Routing System"
date: 2025-06-20 # 오늘 날짜
categories: [Frontend, Next.js]
tags: [nextjs, app-router, page-router, react, ssr, csr, ssg]
layout: post
---

이번 포스팅에서는 Next.js의 두 가지 라우팅 방식인 Page Router와 App Router를 비교해보려 합니다.

## 1. Page Router (pages 디렉토리)
> Page Router는 next.js 초기부터 존재했던 라우팅 방식으로,  
> pages 디렉토리 안에 파일을 생성하면 해당 파일의 경로가 웹페이지의 경로로 자동 매핑된다.
> next 최신버전에서도 여전히 지원하지만 공식적으로는 App Router 방식을 추천한다.

**📁 파일 기반 라우팅 예시**
* `pages/index.js`는 웹 애플리케이션의 루트 경로 (`/`)에 매핑됩니다.
* `pages/about.js`는 `/about` 경로에 매핑됩니다.
* `pages/dashboard/settings.js`는 `/dashboard/settings` 경로에 매핑됩니다.

**🔄 동적 라우팅**

Page Router는 동적 라우팅으로 파일명에 대괄호 `[]`를 사용합니다.

* `pages/posts/[id].js`와 같이 대괄호를 사용하여 동적 경로를 정의합니다.
    * `/posts/1` 또는 `/posts/hello-world`와 같은 URL이 이 파일에 매핑됩니다.
    * 페이지 컴포넌트 내에서는 `useRouter` 훅을 사용하여 `id` 값을 추출할 수 있습니다.
  

### **✨ 주요 특징 및 동작 방식**

* **페이지 컴포넌트**: `pages` 디렉토리 내의 각 `.js`, `.jsx`, `.ts`, `.tsx` 파일은 기본적으로 **React 컴포넌트**를 내보내야 하며, 이 컴포넌트가 해당 경로의 UI를 담당합니다.
* **데이터 페칭 함수**: Page Router는 페이지 단위의 데이터 페칭을 위해 특별한 함수들을 제공합니다.
    * `getServerSideProps (SSR)`: 페이지 요청이 들어올 때마다 서버에서 데이터를 미리 가져와 페이지를 렌더링합니다. 항상 최신 데이터를 보여줘야 하는 경우에 적합합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // pages/products/[id].tsx

        import { GetServerSideProps, NextPage } from 'next';

        interface Product {
          id: string;
          name: string;
        }

        interface ProductPageProps {
          product: Product;
        }

        const ProductPage: NextPage<ProductPageProps> = ({ product }) => {
          // `product`는 이미 getServerSideProps에서 가져와서 여기에 전달된 상태입니다.
          return (
            <div>
              <h1>상품 이름: {product.name}</h1>
              <p>상품 ID: {product.id}</p>
            </div>
          );
        };

        export const getServerSideProps: GetServerSideProps = async (context) => {
          const { id } = context.query;

          // 실제 API 호출 대신 간단한 Mock 데이터 사용
          const product: Product = { id: id as string, name: `상품 ${id}` };

          return { props: { product } };
        };
        //getServerSideProps 함수는 next가 페이지를 렌더링 하기 전 서버에서 미리 데이터를 가져옵니다.
        //이 함수가 반환하는 props가 ProductPage로 전달됩니다.

        export default ProductPage;
        ```
        </div>
        </details>
     
    * `getStaticProps (SSG)`: 빌드 시점에 데이터를 미리 가져와 HTML 파일을 생성합니다. 데이터가 자주 변경되지 않는 블로그 게시물과 같은 콘텐츠에 적합합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // pages/blog/index.tsx
        import { GetStaticProps, NextPage } from 'next';

        interface Post {
          id: number;
          title: string;
        }

        interface BlogPageProps {
          posts: Post[];
        }

        const BlogPage: NextPage<BlogPageProps> = ({ posts }) => {
          return (
            <div>
              <h1>최신 블로그 게시물</h1>
              <ul>
                {posts.map((post) => (
                  <li key={post.id}>{post.title}</li>
                ))}
              </ul>
            </div>
          );
        };

        export const getStaticProps: GetStaticProps = async () => {
          // 실제 API 호출 대신 간단한 Mock 데이터 사용
          const posts: Post[] = [
            { id: 1, title: '첫 번째 게시물' },
            { id: 2, title: '두 번째 게시물' },
          ];
          return { props: { posts } };
        };
        //이 함수는 빌드 시점에 서버에서 실행됩니다.
        //페이지를 미리 HTML 파일로 생성하는데 필요한 데이터를 가져오는 역할을 합니다.

        export default BlogPage;
        ```
        </div>
        </details>
    * `getStaticPaths` (`getStaticProps`와 함께 사용): 동적 경로 (`pages/posts/[id].tsx`와 같은)에 대해 `getStaticProps`를 사용하여 미리 빌드할 경로(즉, `id` 값)의 목록을 정의합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // pages/posts/[id].tsx
        import { GetStaticProps, GetStaticPaths, NextPage } from 'next';

        interface Post {
          id: string;
          title: string;
        }

        interface PostPageProps {
          post: Post;
        }

        const PostPage: NextPage<PostPageProps> = ({ post }) => {
          return (
            <div>
              <h1>{post.title}</h1>
              <p>게시물 ID: {post.id}</p>
            </div>
          );
        };

        export const getStaticPaths: GetStaticPaths = async () => {
          // 미리 빌드할 게시물 ID 목록
          const paths = [{ params: { id: '1' } }, { params: { id: '2' } }];
          return { paths, fallback: false }; // fallback: false는 정의되지 않은 경로는 404
        };
        //이 함수 또한 빌드 시점에 실행되어 동적 경로(`[id]`)에 대해 
        //어떤 특정 `id` 값들을 가진 페이지를 미리 정적으로 생성할지 결정합니다.

        export const getStaticProps: GetStaticProps = async (context) => {
          const { id } = context.params!;
          // 실제 API 호출 대신 간단한 Mock 데이터 사용
          const post: Post = { id: id as string, title: `게시물 ${id}` };
          return { props: { post } };
        };

        export default PostPage;
        ```
        </div>
        </details>
* **\_app.tsx 및 \_document.tsx**:
    * `pages/_app.tsx`: 모든 페이지에 공통으로 적용되는 레이아웃이나 전역 CSS, 상태 관리 프로바이더 등을 설정할 수 있습니다.  
    즉 모든 페이지를 감싸는 최상위 컴포넌트 역할을 합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // pages/_app.tsx
        import type { AppProps } from 'next/app';
        import '../styles/globals.css'; // 전역 스타일 임포트

        function MyApp({ Component, pageProps }: AppProps) {
          return (
            <div>
              <header>글로벌 헤더</header>
              <Component {...pageProps} /> {/* 실제 페이지 컴포넌트 */}
              <footer>글로벌 푸터</footer>
            </div>
          );
        }

        export default MyApp;
        ```
        </details>
        </div>
    * `pages/_document.tsx`: HTML 문서의 `<html>`과 `<body>` 태그를 커스터마이징할 때 사용합니다. 주로 서버 사이드 렌더링 시 웹폰트나 외부 스크립트 삽입 등에 활용됩니다.
      * 이 파일은 서버에서만 렌더링되며, 클라이언트 사이드에서는 리렌더링 되지 않습니다.
      * 주로 SEO나 성능 최적화와 관련된 목적으로 사용됩니다.

* **API Routes**: `pages/api` 디렉토리 내에 파일을 생성하여 서버리스 함수로 동작하는 API 엔드포인트를 쉽게 만들 수 있습니다. 클라이언트 측 코드에서 직접 데이터베이스에 접근하지 않고 안전하게 서버측 로직을 처리할 수 있습니다.
  * <details>
    <summary>예시 코드</summary>
    <div markdown = "1">
    ```typescript
    // pages/api/hello.ts
    // 이 파일의 경로가 곧 API 엔드포인트의 URL이 됩니다. 
    // 예를 들어, 이 파일은 `/api/hello` 경로로 접근할 수 있습니다.
    import type { NextApiRequest, NextApiResponse } from 'next';

    type Data = {
      name: string;
    };

    export default function handler(
      req: NextApiRequest,
      res: NextApiResponse<Data>
    ) {
      res.status(200).json({ name: 'John Doe' });
    }
    ```
    </details>
    </div>

### **✅ Page Router의 장점:**

* **직관적인 파일 기반 라우팅**: 파일 경로가 URL 경로와 1:1로 매핑되어 매우 직관적이고 이해하기 쉽습니다.
* **빠른 시작**: 간단한 웹사이트, 블로그, 랜딩 페이지 등을 빠르게 구축할 수 있습니다. 복잡한 설정 없이 바로 시작 가능합니다.
* **풍부한 자료 및 커뮤니티 지원**: 오랫동안 사용되어 온 방식이므로 관련 튜토리얼, 문서, 커뮤니티 지원이 매우 활발합니다.
* **명확한 렌더링 방식 선택**: `getServerSideProps`와 `getStaticProps`를 통해 페이지별로 SSR 또는 SSG를 명확하게 선택하고 제어할 수 있습니다.

### **⚠️ Page Router의 단점:**

* **데이터 페칭의 복잡성 및 비효율성**: 각 페이지 컴포넌트에서 데이터 페칭 로직을 직접 관리해야 합니다. 컴포넌트 트리가 깊어질 수록 `props drilling` 문제가 발생할 수 있습니다.
* **레이아웃 관리의 한계**: `_app.tsx`를 통해 전역 레이아웃을 설정할 수는 있지만, 특정 페이지 그룹에만 적용되는 복잡한 중첩 레이아웃을 구현하기는 번거롭습니다.
* **부분 렌더링의 제약**: 페이지 단위로 렌더링되는 특성상, 데이터가 변경되었을 때 UI의 특정 부분만 효율적으로 업데이트하는 데 제약이 있을 수 있습니다.
* **React Server Components (RSC) 미지원**: Page Router는 RSC를 직접 지원하지 않아, 서버/클라이언트 컴포넌트 분리 및 최적화 이점을 활용할 수 없습니다

---

## 2. App Router (app 디렉토리)

> App Router는 Next.js 13부터 도입된 최신 라우팅 방식이며, React의 새로운 기능인 **React Server Components (RSC)**를 기반으로 합니다. 이 방식은 `app` 디렉토리를 사용하며, 기존 Page Router의 한계를 극복하고 더 유연하고 성능 최적화된 웹 애플리케이션 개발을 목표로 합니다.

### ✨ **주요 특징 및 동작 방식:**

* **폴더 기반 라우팅**: `app` 디렉토리 내의 폴더 구조가 URL 경로의 세그먼트를 정의합니다. Page Router와 달리, 경로 세그먼트를 나타내는 폴더 안에 특정 파일명(예: `page.tsx`, `layout.tsx`)을 사용하여 UI를 정의합니다.

    **핵심 파일:**
    * `page.tsx`: 특정 URL 경로에 대한 고유한 UI를 렌더링하는 역할을 합니다. 각 경로 세그먼트에는 반드시 하나의 `page` 파일이 있어야 합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```ts
        // 이 파일은 Next.js의 'app' 디렉토리 안에 있으며, `dashboard` 폴더 안의 `page.tsx`입니다.
        // 따라서 이 파일은 `/dashboard` 경로에 접근했을 때 최종적으로 렌더링될 UI를 정의합니다.
        export default function DashboardPage() {
          return <h1>App Router - 대시보드 페이지</h1>;
        }
        ```
        </details>
        </div>
    * `layout.tsx`: 여러 페이지에 걸쳐 공유되는 UI를 정의합니다. `children` prop을 통해 하위 콘텐츠를 렌더링 합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // app/dashboard/layout.tsx
        // `layout.tsx`는 해당 경로 세그먼트(`dashboard`)와 그 하위 경로의 모든 페이지들이 공유할 UI를 정의합니다.

        export default function DashboardLayout({ children }: { children: React.ReactNode }) {
          return (
            // 이 컴포넌트가 모든 대시보드 관련 페이지에 공통적으로 적용될 레이아웃 UI를 반환합니다.
            <html>
              <body>
                <header style={{ borderBottom: '1px solid gray', padding: '10px' }}>
                  <h1>App Router - 대시보드 레이아웃</h1>
                </header>
                <main style={{ padding: '20px' }}>{children}</main> 
                // 여기에 page.tsx 또는 하위 레이아웃이 렌더링됨
                <footer style={{ borderTop: '1px solid gray', padding: '10px' }}>
                  <p>&copy; 2025 MyApp</p>
                </footer>
              </body>
            </html>
          );
        }
        ```
        </details>
        </div>
    * `loading.tsx`: React Suspense와 통합되어 작동하며, 해당 경로 세그먼트의 데이터 페칭 중 보여줄 로딩 UI를 정의합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // app/dashboard/loading.tsx
        export default function DashboardLoading() {
          return <p>⏳ 대시보드 데이터를 로딩 중입니다...</p>;
        }
        ```
        </details>
        </div>
    * `error.tsx`: 해당 경로 세그먼트에서 에러가 발생했을 때 보여줄 에러 UI를 정의합니다. React Error Boundaries와 유사하게 작동하며, 에러 복구 로직을 구현할 수 있습니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // app/products/[id]/error.tsx
        'use client'; // Error Boundaries는 클라이언트 컴포넌트여야 합니다.
        import { useEffect } from 'react';

        export default function Error({
          error,
          reset,
        }: {
          error: Error & { digest?: string };
          reset: () => void;
          // `error`: 발생한 에러 객체입니다. Next.js는 에러의 요약을 나타내는 `digest` 속성을 추가할 수 있습니다.
          // `reset`: 에러 상태를 재설정하여 에러가 발생한 컴포넌트를 다시 렌더링하려고 시도하는 함수입니다.
        })
        {
          useEffect(() => {
            // 에러를 로깅.
            console.error(error);
          }, [error]);

          return (
            <div>
              <h2>문제가 발생했습니다!</h2>
              <button onClick={() => reset()}>다시 시도</button>
            </div>
          );
        }
        ```
        </details>
        </div>
    * `template.tsx`: `layout.tsx`와 유사하게 하위 경로에 UI를 적용하지만, `layout`과 달리 라우트 변경 시 새로운 인스턴스가 마운트됩니다. 이는 CSS 애니메이션과 같이 마운트/언마운트 시점에 특정 동작이 필요한 경우에 유용합니다.
    * `not-found.tsx`: 해당 경로에서 페이지를 찾을 수 없을 때 (404 에러) 보여줄 UI를 정의합니다.
  

* **React Server Components (RSC) 기반**: App Router의 가장 큰 특징은 React Server Components를 기본적으로 사용한다는 점입니다.
    * **서버 컴포넌트 (기본)**: `app` 디렉토리 내의 모든 컴포넌트는 기본적으로 서버 컴포넌트로 간주됩니다. 서버에서 렌더링되어 클라이언트 측으로 전송되므로, 초기 로딩 속도를 향상시킵니다. 서버 리소스(파일 시스템, 데이터베이스 등)에 직접 접근할 수 있습니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        import ClientButton from './components/ClientButton';

        export default function HomePage() {
          return (
            <div>
              <h1>환영합니다!</h1>
              <ClientButton /> {/* 서버 컴포넌트 내에서 클라이언트 컴포넌트 사용 */}
            </div>
          );
        }
        ```
        </details>
        </div>
    * **클라이언트 컴포넌트 (`"use client"`)**: 파일 상단에 `"use client"` 지시어를 명시하면 해당 컴포넌트는 클라이언트 컴포넌트로 지정됩니다. `useState`, `useEffect`와 같은 React Hooks를 사용해야 하거나, 브라우저 API(DOM, 로컬 스토리지 등)에 접근해야 할 때 사용합니다. 
      * 클라이언트 컴포넌트는 서버 컴포넌트의 자식으로만 사용 가능합니다.
      * <details>
        <summary>예시 코드</summary>
        <div markdown = "1">
        ```typescript
        // app/components/ClientButton.tsx
        'use client';
        import { useState } from 'react';

        export default function ClientButton() {
          const [count, setCount] = useState(0);
          return (
            <button onClick={() => setCount(count + 1)}>
              클릭 횟수: {count}
            </button>
          );
        }
        ```
        </details>
        </div>
* **새로운 데이터 페칭 방식**: App Router는 서버 컴포넌트 내에서 일반적인 `fetch` API를 직접 사용하여 데이터를 가져올 수 있습니다. 페이지 단위가 아닌 컴포넌트 단위로 데이터 페칭이 가능해, 데이터 흐름이 더 유연하고 `props drilling`이 줄어듭니다.
  * <details>
    <summary>예시 코드</summary>
    <div markdown = "1">
    ```typescript
    // app/dashboard/page.tsx (서버 컴포넌트)
    interface User {
      id: number;
      name: string;
      email: string;
    }

    async function getUsers(): Promise<User[]> {
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1초 지연
      return [
        { id: 1, name: 'Alice', email: 'alice@example.com' },
        { id: 2, name: 'Bob', email: 'bob@example.com' },
      ];
    }

    export default async function DashboardPage() {
      const users = await getUsers(); // 서버에서 직접 데이터 페칭

      return (
        <div>
          <h1>사용자 목록</h1>
          <ul>
            {users.map(user => (
              <li key={user.id}>
                {user.name} ({user.email})
              </li>
            ))}
          </ul>
        </div>
      );
    }
    ```
    </details>
    </div>
* **스트리밍 및 Suspense**: 
  * React의 Suspense 기능과 Next.js의 스트리밍 렌더링을 활용해, 데이터가 준비되는 대로 UI를 점진적으로 보여줄 수 있습니다.
  * loading.tsx 파일을 통해 로딩 상태에서 대체 UI(스켈레톤 등)를 제공하며, 사용자는 더 빠른 피드백을 받을 수 있습니다.
* **강화된 SEO 기능**: `generateMetadata` 함수를 사용하여 각 페이지별로 동적인 메타데이터(`title`, `description` 등)를 설정할 수 있어 SEO 최적화에 용이합니다.
  * <details>
    <summary>예시 코드</summary>
    <div markdown = "1">
    ```typescript
    // app/products/[slug]/page.tsx
    import type { Metadata } from 'next';
    //next 제공 Meatdata 활용

    interface Product {
      id: string;
      name: string;
      description: string;
    }

    // Mock 데이터 페칭 함수
    async function getProduct(slug: string): Promise<Product> {
      // 실제 API 호출 대신 간단한 Mock 데이터 사용
      return {
        id: slug,
        name: `Mock Product for ${slug}`,
        description: `이것은 ${slug} 상품에 대한 상세 설명입니다.`,
      };
    }

    type Props = {
      params: { slug: string };
    };

    export async function generateMetadata({ params }: Props): Promise<Metadata> {
      const product = await getProduct(params.slug);
      return {
        title: product.name,
        description: product.description,
      };
    }

    export default async function ProductDetailPage({ params }: Props) {
      const product = await getProduct(params.slug);
      return (
        <div>
          <h1>{product.name}</h1>
          <p>{product.description}</p>
        </div>
      );
    }
    ```
    </details>
    </div>


* **Server Actions**: 
  * 서버에서 직접 실행되는 비동기 함수를 정의하여 클라이언트-서버 간의 통신을 간소화하고, 폼 제출 및 데이터 변경과 같은 작업을 더 쉽게 처리할 수 있도록 합니다.
  * 별도의 API 라우트 없이도 서버 로직을 안전하게 실행할 수 있어 개발 생산성이 향상됩니다.
  * <details>
    <summary>예시 코드</summary>
    <div markdown = "1">
    ```ts
    // app/actions.ts (서버 액션 파일)
    'use server'; // 이 파일의 모든 함수가 서버에서만 실행됨을 명시

    export async function createTodo(formData: FormData) {
      const todoText = formData.get('todo') as string;
      // 실제 데이터베이스에 저장하는 로직 (예: db.todos.create({ data: { text: todoText } }))
      console.log('새로운 할 일 생성:', todoText);
    }
    ```
    ```ts
    // app/add-todo/page.tsx (클라이언트 컴포넌트에서 사용)
    'use client';

    import { createTodo } from '../actions'; // 서버 액션 임포트

    export default function AddTodoPage() {
      return (
        <form action={createTodo}> {/* form의 action prop으로 서버 액션 지정 */}
          <input type="text" name="todo" placeholder="새 할 일" required />
          <button type="submit">할 일 추가</button>
        </form>
      );
    }
    ```
    </details>
    </div>

### **✅ App Router의 장점:**

* **향상된 성능 및 효율성**: 
  * 서버 컴포넌트(RSC)로 인한 초기 로딩 속도 개선과 번들 크기 감소됩니다.
  * 스트리밍 및 Suspense를 통한 점진적 렌더링으로 사용자 경험 향상됩니다.
* **유연한 데이터 페칭**: 컴포넌트 레벨에서 `fetch`를 직접 사용하여 데이터를 가져올 수 있어, 데이터 흐름이 더 명확해지고 `props drilling`을 줄일 수 있습니다.
* **중첩 레이아웃의 용이성**: `layout.tsx` 파일을 통해 여러 레이아웃을 효율적으로 중첩하고 관리할 수 있어 복잡한 UI 구조를 쉽게 구현할 수 있습니다.
* **SEO 최적화**: `generateMetadata` 함수를 통해 동적으로 메타데이터를 설정하여 SEO 최적화를 강화할 수 있습니다. 
* **대규모 애플리케이션에 적합**: 서버와 클라이언트 컴포넌트의 명확한 분리로 대규모 애플리케이션의 유지보수성과 확장성을 높입니다.

### **⚠️ App Router의 단점:**

* **높은 학습 곡선**: RSC, Suspense 등 새로운 개념이 많아 초기 학습 및 적응 시간이 필요합니다.
* **기존 프로젝트 마이그레이션의 어려움**: Page Router 기반의 기존 프로젝트를 App Router로 전환하는 것이 코드 재구성이 필요해 복잡할 수 있습니다.
* **생태계 초기 단계**: Page Router에 비해 아직 생태계가 작으며, 일부 서드파티 라이브러리나 도구와의 호환성 문제가 발생할 수 있습니다.
* **디버깅 복잡성 증가**: 서버와 클라이언트 간의 경계가 모호해질 수 있어 디버깅 과정이 더 복잡하게 느껴질 수 있습니다.

---

## 3. Page Router와 App Router 비교

두 라우팅 방식의 주요 차이점을 다음 표로 정리했습니다.

| 특징            | Page Router (`pages/`)                                      | App Router (`app/`)                           |
| :-------------- | :---------------------------------------------------------- | :-------------------------------------------- |
| **기반 기술**   | React Component                                             | React Server Components (RSC)                 |
| **파일 구조**   | 파일 경로 = URL 경로                                        | `app/폴더/` 기반 세그먼트 정의&특수 파일 사용 |
| **레이아웃**    | `_app.tsx` 및 `_document.tsx`를<br> 통한 전역 레이아웃      | `layout.tsx` 파일을 통한 중첩 레이아웃 지원.  |
| **데이터 페칭** | `getServerSideProps`, <br>`getStaticProps` 등 (페이지 단위) | 컴포넌트 단위 `fetch`                         |
| **렌더링 방식** | SSR, SSG, CSR 혼합                                          | 스트리밍 + 점진적 렌더링                      |
| **SEO**         | `next/head` 수동 설정                                       | `generateMetadata` 자동화                     |
| **State/Hooks** | 모든 컴포넌트에서 사용 가능                                 | 클라이언트 컴포넌트에서만 사용 가능           |
| **학습 곡선**   | 낮음 (직관적)                                               | 높음 (새로운 개념 많음)                       |
| **주요 사용처** | 간단한 웹사이트, 블로그, 정적 페이지 등                     | 대시보드, 대규모 서비스 등                    |

---

## 4. 언제 어떤 라우터를 선택해야 할까?

Next.js는 현재 두 라우팅 방식을 모두 지원하며, 프로젝트의 특정 요구사항에 따라 적절한 방식을 선택해야 합니다.

* **Page Router가 적합한 경우:**
    * **간단한 웹사이트 또는 블로그**: 페이지 구조가 단순하고, 복잡한 중첩 레이아웃이나 고도의 인터랙티브가 필요 없는 경우.
    * **빠른 프로토타이핑**: 빠르게 아이디어를 구현하고 배포해야 할 때.
    * **SEO가 중요하지만 데이터 페칭 복잡성이 낮은 경우**: SSG를 활용하여 정적 페이지를 생성하는 것이 유리할 수 있습니다.

* **App Router가 적합한 경우:**
    * **복잡한 애플리케이션**: 대시보드, SaaS 플랫폼, 소셜 미디어 애플리케이션 등 복잡한 UI와 동적인 데이터 처리가 필요한 경우.
    * **성능 최적화**: 초기 로딩 속도, 번들 크기, 데이터 로딩 중 사용자 경험(스트리밍, Suspense)을 극대화하고 싶을 때.
    * **컴포넌트 단위의 데이터 페칭 및 관리의 필요성**: `props drilling`을 줄이고 컴포넌트 레벨에서 데이터 관리를 유연하게 하고 싶을 때.

* **하이브리드 접근 방식**: Next.js는 `pages`와 `app` 디렉토리를 동시에 사용하는 것을 허용하지만 권장되지 않습니다.
 예로 /app(신규 기능) + /pages(기존 API) 혼용 시 네비게이션 불일치가 발생할 수 있습니다.

## ✨ 결론

> Next.js의 라우팅 시스템은 Page Router에서 App Router로 발전했습니다.
> 
> "Page Router는 단순성과 안정성을, App Router는 성능과 미래 지향성을 제공합니다.
> 
> 프로젝트 규모와 팀 역량을 고려해 선택하되, 신규 프로젝트에는 App Router를 우선 검토하는 것이 바람직합니다."

---

## 📚 참고

* [Next.js 공식 문서 - Pages Router](https://nextjs.org/docs/pages)
* [Next.js 공식 문서 - App Router](https://nextjs.org/docs/app)
* [Difference Between Page Router Vs App Router In Next.Js](https://appicsoftwares.com/blog/difference-between-page-router-vs-app-router-in-next-js/)

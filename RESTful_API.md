# RESTful API

RESTful API는 웹(HTTP)의 장점을 최대한 활용하여 **"자원(Resource)을 이름(URI)으로 구분하고, 상태(State)를 주고받는"** 통신 방식이다.

쉽게 말하면 **"전 세계 공통으로 통하는 체계적인 메뉴판과 주문 규칙"** 이라고 생각하면 된다.

## RESTful API의 핵심 3요소
RESTful API에서 가장 중요한 것은 "무엇(자원)"을 "어떻게(행위)" 할 것인지 명확히 하는 것이다.
1. 자원 (Resource) : "무엇" (URI)
   - 모든 데이터에 고유한 **주소(URI)** 를 부여한다.
   - **규칙** : 명사(Noun)를 사용하며, 소문자와 복수형을 주로 쓴다. (/users, ~~/getUsres~~)

2. 행위 (Verb) : "어떻게" (HTTP Method)
   - 자원에 대해 어떤 행동을 할 것인지는 **HTTP 메서드**로 결정한다. 이것이 곧 **CRUD(Create, Read, Update, Delete)** 기능과 연결된다.

      |HTTP 메서드|역할|CRUD|예시 (회원 관리)|
      |------|---|---|---|
      |**GET**|데이터 조회|Read|`GET /users` (목록 조회)|
      |**POST**|데이터 생성|Create|`POST /usres` (신규 가입)|
      |**PUT**|전체 수정|Update|`PUT /users/1` (1번 회원 정보 통째로 변경)|
      |**PATCH**|일부 수정|Update|`PATCH /users/1`(1번 회원의 닉네임만 변경)|
      |**DELETE**|데이터 삭제|Delete|`DELETE /users/1` (1번 회원 탈퇴)|
   
3. 표현 (Representation) : "어떤 모양으로"
     - 클라이언트와 서버가 데이터를 주고받을 때 어떤 형식으로 주고받을지 지정한다.
     - 보통 99% **JSON** 형식을 사용한다.

## RESTful API 설계 예시 (쇼핑몰)
만약 쇼핑몰의 **'상품(Products)'** 기능을 만든다면, RESTful한 주소는 다음과 같다.
- 상품 목록 보기 : `GET /products`
- 특정 상품(ID : 10) 상세 보기 : `GET /products/10`
- 새 상품 등록하기 : `POST /products` (데이터는 Body에 담아 보냄)

  - 데이터 준비 : 보내고자 하는 데이터를 json 객체 형식으로 만든다.
      ```json
      // data.json
      {
         "id" : 10,
         "name" : "Tissue",
         "price" : 5000
      }
      ```
   - 웹 브라우저 환경에서 가장 표준적으로 서버에 데이터를 담아 보내는 방법이다.
      ```javascript
      // script.js
      {
         fetch('https://api.example.com/submit', {
            method: 'POST',
            headers: {
               'Content-Type': 'application/json',
            },
            body: JSON.stringify({ // JavaScript 객체를 JSON 문자열로 변환
               id : 10,
               name : "Tissue",
               price : 5000
            })
         })
         .then(response => response.json())
         .then(data => console.log(data))
         .catch(error => console.error('Error:', error));
      }
      ```

## 꼭 지켜야 할 특징 (Stateless)
   RESTful API의 가장 중요한 기술적 특징은 **무상태성(Stateless)** 이다.
   - 의미 : 서버는 클라이언트의 이전 상태(문맥)를 기억하지 않습니다.

   - 예시 : "나 아까 로그인했어"라고 서버가 알아주지 않는다.
  
   - 방법 : 클라이언트는 요청을 보낼 떄마다 **"내가 누구인지 증명하는 토큰(Token)"** 을 매번 함께 보내야함

## 응답 코드 (State Code)의 중요성
RESTful API는 성공/실패 여부를 **HTTP 상태 코드**로 명확하게 알려주어야 한다.

- `200 OK` : 성공

- `201 Created` : 생성 성공 (POST 요청 시)
- `400 Bad Request` : 요청 형식이 잘못됨
- `401 Unauthorized` : 로그인 필요 (인증 실패)
- `404 Not Found` : 없는 자원 (주소 틀림)
- `500 Internal Server Error` : 서버 터짐 (개발자 실수)

# TypeScript 환경에서 RESTful API 요청 예제

TypeScript 환경에서 RESTful API를 요청할 때 가장 대중적으로 사용하는 라이브러리는 **Axios**이다.

1. 사전 준비 : 데이터 타입 정의 (Interface)
   먼저 주고받을 데이터의 형태를 정의한다.
   ```ts
   // types.ts

   // 상품 데이터 타입
   export interface Product {
      id : number;
      name : string;
      price : number;
      description?: string; //선택적 속성, 있을수도 있고 없을수도 있고 (사용 주의)
   }

   // 새 상품 등록 시 사용할 타입 (ID는 서버가 만드므로 제외)
   export type NewProduct = Omit<Product, "id">;
   ```
2. API 요청 함수 모음 (API Service)

   각 메서드 별로 TypeScript 문법 포인트와 함께 예제 코드 작성
   ```ts
      import axios from 'axios';
      import { Product, NewProduct } from './types';

      const BASE_URL = 'http://api.myshop.com';

      // 1. GET : 항목 조회 (Read) 
      // axios.get<응답타입>(주소)
      export const getProducts = async () => {
         // 서버에서 Product 배열([] product)이 온다고 명시
         const response = await axios.get<Product[]>(`${BASE_URL}/products`);
         return response.data;
      }

      // 2. POST : 데이터 생성 (Create)
      // axios.post<응답타입>(주소, 보낼데이터)
      export const creatProduct = async (productData: NewProduct) => {
         await axios.post<Product>(`${BASE_URL}/products`, productData);
         return response.data;
      }

      // 3. PUT : 전체 수정 (Update)
      // 모든 필드를 다 보내야 함
      export const updateProduct = async (id : number, productData : NewPorduct) => {
         const response = await axios.put<Product>(`${BASE_URL}/products/${id}`, productData);
         return response.data;
      }

      // 4. PATCH : 일부 수정 (Update)
      // 변경할 필드만 보내면 됨 -> Partial 유틸리티 타입응 사용한다.
      // Partial<Product>는 모든 속성을 선택적(Optional)으로 만듬
      export const updateProductPrice = async (id : number, patchData : Partial<Product>) => {
         const response = await axios.patch<Product>(`${BASE_URL}/products/${id}`, patchData);
         return response.data;
      }

      // 5. DELETE : 삭제 (Delete)
      // 보통 응답 데이터가 없거나 성공 메세지만 온다.
      export const deleteProduct = ansyn (id : number) => {
         const response = await axios.delete< {success : boolean} >(`${BASE_URL}/products/${id}`);
         return response.data;
      }
   ```

# React 컴포넌트에서 실제 사용 예시
 위 TypeScript로 만든 API 요청 함수들을 컴포넌트에서 사용해보자.

   ```tsx
   import React, { useEffect, useState } from 'react';
   import { getProducts, createProduct, updateProductPrice } from './apiService';
   import { Product } from '/types';

   export default function ProductPage() {
      const [products, setProducts] = useState<Product[]>([]);

      // 1. GET 예시 (화면 렌더링 시)
      useEffect(() => {
         const loadData = async () => {
            try{
               const data = await getProducts();
               setProducts(data); 
               // data는 자동으로 Product[] 타입으로 인식됨
               // 이유는? : apiService.ts 에서 정의한 getProducts 함수에서 제네릭<>을 통해 리턴 타입을 명시함.
               // TypeScript의 가장 큰 장점 - 한 번 정의하고, 어디서든 안전하게 사용이 가능함.
            } catch (error) {
               console.error("불러오기 실패", error);
            }
         };
         loadData();
      }, []);
      
      // 2. POST 예시 (버튼 클릭 시)
      const handleAdd = async () => {
         const newOne = { name : "게이밍 마우스", price : 50000 };
         const created = await creatProduct(newOne);
         setProducts([...products, created]); // 화면 즉시 갱신
      };

      // 4. PATCH 예시 (가격만 변경)
      const handleDiscount = async (id : number) => {
         // 가격만 1000원으로 변경
         await updateProductPrice(id, { price : 1000 });
         // 이후 다시 목록을 불러오거나 State 업데이트
      };

      return (
         <div>
            <h1>상품 목록</h1>
            <button onClick={handleAdd}>상품 추가</button>
            <ul>
               {products.map(p => (
                  <li key={p.id}>
                     {p.name} - {p.price}원
                     <button onClick={() => handleDiscount(p.id)}>할인 적용</button>
                  </li>
               ))}
            </ul>
         </div>
      );
   }
   ```

## TypeScript를 사용하여 API 요청 시 핵심 포인트는?

1. 제네릭 사용 (`<Type>`) : `axios.get<Product[]>` 처럼 제네릭을 명시하면, `response.data`를 사용할 때 자동 완성이 되고 타입 에러를 잡아준다.
2. 유틸리티 타입 활용 : 
   
   - `Omit` : ID처럼 서버가 만드는 필드를 뺄 때 사용 (POST)
   - `Partial` : 일부 필드만 수정할 때 타입을 유연하게 만들 때 사용 (PATCH)

# jQuery 에서 API 요청 보내기 AJAX(Asynchoronous JavaScript and XML) 메서드 사용

jQuery의 가장 큰 장점은 복잡한 브라우저 호환성 문제를 해결하고, 매우 직관적이고 짧은 코드로 서버와 통신할 수 있다는 점이다.

|메서드|역할|특징|
|------|---|---|
|`$.ajax()`|만등|GET, POST, PUT, DELETE 등 모든 설정이 가능하다.|
|`$.get()`|조회 전용|GET 요청을 빠르게 보낼 때 사용하는 단축 문법이다.|
|`$.post()`|생성 전용ㅇ|POST 요청을 빠르게 보낼 때 사용하는 단축 문법이다.|

1. GET 데이터 조회 (Read)
```js
$.ajax({
   url : 'http://api.myshop.com/products' // 요청할 주소
   type : "GET"   // 요청 방식 (GET)
   dataType: "json"  // 서버에서 받을 데이터 타입
   success : function(response){
      // 성공 시 실행
      console.log("데이터 조회 성공", response);
   },
   error : function(xhr, status, error){
      // 실패 시 실행
      console.error("에러 발생", error);
   }
});
```
**단축 문법** : `$.get("http://api.myshop.com/products", function(data){ ... });`

2. POST : 데이터 생성 (Create)
```js
$.ajax({
   url : "http://api.myshop.com/products",
   type : "POST",
   data : {
      name: "게이밍 마우스",
      price: 50000
   }, // jQuery가 자동으로 폼 데이터 형식으로 변환해서 보냄
   success : function(response){
      console.log("생성 완료", response);
   }
});
```

3. PUT : 데이터 수정 (Update)
- 주의 : **REST API**에서 JSON 형식으로 데이터를 받길 원한다면 `contentType` 과 `JSON.stringify` 설정이 필요할 수 있다.
```js
$.ajax({
   url : "http://api.myshop.com/products/10",
   type : "PUT",
   contentType : "application/json", // 서버에게 JSON 보낸다 하고 알려주는 역할
   data : {
      name: "무선 마우스",
      price: 60000
   }, // 객체를 JSON 문자열로 변환
   success : function(response){
      console.log("수정 완료", response);
   }
});
```

4. DELETE : 데이터 삭제 (Delete)
```js
$.ajax({
   url : "http://api.myshop.com/products/10",
   type : "DELETE",
   success : function(response){
      console.log("삭제 완료");
   }
})
```

## 최신 스타일 : Promise 패턴 (.done, .fail)
콜백 지옥(callback hell)을 피하기 위해 `success`, `error` 대신 **체이닝(Chainig)** 방식을 권장하고있다.
```js
$.ajax({
   url : "http://api.myshop.com/products",
   type : "GET"
})
.done(function(data){
   // 성공 (HTTP 200)
   console.log("성공!", data);
})
.fail(function(xhr, status, error){
   // 실패 (HTTP 4xx, 5xx)
   console.log("오류 발생", error)
})
.always(function(){
   // 성공하든 실패하든 무조건 실행 (예 : 로딩 아이콘 끄기)
   console.log("요청 종료");
})
```

# Promise 패턴
React 에서 **Promise 패턴**은 주로 비동기 작업(API 호출 등)울 처리하고, 그 결과에 따라 화면(UI)을 업데이트하는 표준화된 방식을 말한다.

React는 기본적으로 동기적(순서대로 즉시 실행)으로 작동하지만, 서버에서 데이터를 가져오는 것은 시간이 걸리는 비동기 작업이다. 이때 **Promise**가 다리 역할을 한다.

## Promise의 3가지 상태와 React State 매핑
Promise에는 2가지 상태가 있고, 이를 React 컴포넌트의 상태로 치환해서 관리하는 것이 정석이다.

|Promise 상태|의미|React State 매핑 (예시)| 화면 UI|
|-----|---|---|---|
|Pending|대기 중 (진행 중)|`isLoading : ture`|로딩 스피너 (Spinner)|
|Fulfilled|이행 됨 (성공)|`data : {...}`|실제 데이터/목록|
|Rejected|거부됨 (실패)|`error : "에러 메세지"`|에러 안내 문구|

## 최신 트렌드 실전 코드 : React Query (TanStack Query)
요즘 실무에서는 Promise 패턴을 라이브러리를 사용해서 훨씬 쉽게 사용한다고 한다.
```js
import {useQuery} from '@tanstack/react-query';

export default function UseList() {
   // useQuery가 loading, error, data 상태를 알아서 관리해줌
   const { data, isLoading, error } = useQuery({
      queryKey : ['users'],
      queryFn : () => axios.get('/users').then(res => res.data),
   })

   if (isLoading) return <div> 로딩 중 ...</div>;
   if (error) return <div>에러 발생!</div>

   return <div>{data.name}</div>
}
```
## TypeScript를 활용한 커스텀 훅 (useAxios)
매번 컴포넌트마다 `loading`, `error`, `try-catch`를 쓰면 코드가 지저분해진다. 이걸 하나로 묶어서 재사용하는 `hooks/useAxios.ts`이다.
```ts
// hooks/useAxios.ts
import { useState, useEffect } from 'react';
import axios from 'axios';

// 제네릭 T를 사용해서 어떤 데이터든 받을 수 있게 만듦
export function useAxios<T>(url : srting) {
   const [data, setData] = useState<T | null>(null);
   const [loading, setLoading] = useState(false);
   const [error, setError] = useState<string | null>(null);

   useEffect(() => {
      const fetchData = async () => {
         setLoading(true);
         try {
            const res = await axios.get<T>(url);
            setData(res.data);
         } catch (err) {
            if(axios.isAxiosError(err)) {
               setError(err.message);
            }
         } finally {
            setLoading(False);
         }
      };
      fetchData();
   },[url]);

   //Hook의 리턴 값
   return { data, loading, error };
}
```
# Virtual DOM이란?
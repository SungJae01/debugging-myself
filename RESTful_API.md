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
      |**GET**|데이터 조회|Read|```GET /users``` (목록 조회)|
      |**POST**|데이터 생성|Create|```POST /usres``` (신규 가입)|
      |**PUT**|전체 수정|Update|```PUT /users/1``` (1번 회원 정보 통째로 변경)|
      |**PATCH**|일부 수정|Update|```PATCH /users/1```(1번 회원의 닉네임만 변경)|
      |**DELETE**|데이터 삭제|Delete|```DELETE /users/1``` (1번 회원 탈퇴)|
   
   - 표현 (Representation) : "어떤 모양으로"
     - 클라이언트와 서버가 데이터를 주고받을 때 어떤 형식으로 주고받을지 지정한다.
     - 보통 99% **JSON** 형식을 사용한다.

## RESTful API 설계 예시 (쇼핑몰)
만약 쇼핑몰의 **'상품(Products)'** 기능을 만든다면, RESTful한 주소는 다음과 같다.
- 상품 목록 보기 : ```GET /products```
- 특정 상품(ID : 10) 상세 보기 : ```GET /products/10```
- 새 상품 등록하기 : ```POST /products``` (데이터는 Body에 담아 보냄)

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

- ```200 OK``` : 성공

- ```201 Created``` : 생성 성공 (POST 요청 시)
- ```400 Bad Request``` : 요청 형식이 잘못됨
- ```401 Unauthorized``` : 로그인 필요 (인증 실패)
- ```404 Not Found``` : 없는 자원 (주소 틀림)
- ```500 Internal Server Error``` : 서버 터짐 (개발자 실수)

# TypeScript 환경에서 RESTful API 요청 예제

TypeScript 환경에서 RESTful API를 요청할 때 가장 대중적으로 사용하는 라이브러리는 **Axios**이다.

1. 사전 준비 : 데이터 타입 정의 (Interface)
   먼저 주고받을 데이터의 형태를 정의한다.
   ```typescript
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
   ```typescript
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
      //Partial<Product>는 모든 속성을 선택적(Optional)으로 만듬
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


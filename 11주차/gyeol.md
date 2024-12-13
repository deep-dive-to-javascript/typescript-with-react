# 13장 - 타입스크립트와 객체 지향

## 13.1 타입스크립트의 객체 지향

- 타입스크립트는 객체 지향을 구현할 수 있도록 도와주는 자바스크립트의 슈퍼셋
- 타입스크립트는 점진적 타이핑, 구조적 타이핑, 덕 타이핑이 결합된 언어
    - 점진적 타이핑
        - 프로그램 전체가 아닌 개발자가 명시한 일부분만 정적 타입 검사를 거치게 하고 나머지 부분은 그대로 동적 타입 검사가 이루어지게 해서 점진적 개선을 할 수 있도록 해줌
    - 덕 타이핑
        - 객체의 변수와 메서드 집합이 객체의 타입을 결정하게 해줌
    - 구조적 타이핑(구조적 타입 시스템)
        - 노미널 타이핑(Nominal Typing)과 달리 객체의 속성에 해당하는 특정 타입의 속성을 갖는지를 검사해서 타입 호환성을 결정

- 노미널 타이핑(Nominal Typing, 명칭적 타이핑)
    - 명시적인 선언이나 이름에 의존하여 명확한 상속 관계를 지향
    - 특정 키워드를 통해 서로 호환 가능하다고 명시적으로 표현된 타입 간의 할당 만을 허용
    - 노미널 타이핑 언어는 인터페이스와 클래스가 일대일로 대응됨
    - 자바, C#
      
- 객체 지향의 관점에서 타입스크립트가 프론트엔드에게 주는 이점
    - 타입스크립트는 prop을 인터페이스로 정의 가능
        - 객체 지향 패러다임에서는 객체 간의 협력 관계에 초점을 둠
        → 컴포넌트 간의 협력 관계를 표현하는 것이 prop
        - 객체 자체가 아니라 프레임워크에 의해 객체의 의존성이 주입되는 DI(의존성 주입) 패턴을 따름 
        → 타입 스크립트를 사용하면 이러한 패턴의 명확한 표현이 가능
        
        ⇒ 리액트, 뷰 같은 도구를 사용해 DI를 활용하고 있으며, 타입스크립트로 DI 패턴을 더욱 명확하게 표현할 수 있음
        
    - 타입스크립트 자체가 객체 지향적으로 다양한 측면을 표현하는 데 큰 장점을 가지고 있음
        - 점진적 타이핑, 구조적 타이핑, 덕 타이핑
- 레이아웃은 변동 사항이 생길 가능성이 높으므로 미확정 영역으로 두고 공통으로 사용되는 컴포넌트와 비즈니스 영역에서 객체 지향 원칙을 적용하여 설계하면 좋은 구조를 개발할 수 있음

## 13.2 우아한형제들의 활용 방식

- 우아한 형제들의 한 팀에서는 아래와 같은 설계 방식을 사용
    - 온전히 레이아웃만 담당하는 컴포넌트 영역
    - 컴포넌트 영역 위에서 레이아웃과 비즈니스 로직을 연결해주는 커스텀 훅 영역
    - 훅 영역 위에서 객체로서 상호 협력하는 모델 영역
    - 모델 영역 위에서 API를 해석하여 모델로 전달하는 API 레이어 영역

### 1. 컴포넌트 영역

- 장바구니 관련 다이얼로그 컴포넌트 코드
    
    ```tsx
    // components/CartCloseoutDialog.tsx
    import { useCartStore } from "store/modules/cart";
    
    const CartCloseoutDialog: React.VFC = () => {
      const cartStore = useCartStore();
      return (
        <Dialog
          opened={cartStore.PresentationTracker.isDialogOpen("closeout")}
          title="마감 세일이란?"
          onRequestClose={cartStore.PresentationTracker.closeDialog}
        >
          <div
            css={css`
              margin-top: 8px;
            `}
          >
            지점별 한정 수량으로 제공되는 할인 상품입니다. 재고 소진 시 가격이
            달라질 수 있습니다. 유통기한이 다소 짧으나 좋은 품질의 상품입니다.
          </div>
        </Dialog>
      );
    };
    export default CartCloseoutDialog;
    ```
    
    - 온전히 레이아웃 영역만 담당
    - 비즈니스 로직은 useCartStore 내부에 존재

### 2. 커스텀 훅 영역

- 장바구니에 상품을 담는 비즈니스 로직을 레이아웃과 연결해주기 위한 커스텀 훅 영역
    
    ```tsx
    // store/cart.ts
    class CartStore {
      public async add(target: RecommendProduct): Promise<void> {
        const response = await addToCart(
          addToCartRequest({
            auths: this.requestInfo.AuthHeaders,
            cartProducts: this.productsTracker.PurchasableProducts,
            shopID: this.shopID,
            target,
          })
        );
        return response.fork(
          (error, _, statusCode) => {
            switch (statusCode) {
              case ResponseStatus.FAILURE:
                this.presentationTracker.pushToast(error);
                break;
              case ResponseStatus.CLIENT_ERROR:
                this.presentationTracker.pushToast(
                  "네트워크가 연결되지 않았습니다."
                );
                break;
              default:
                this.presentationTracker.pushToast(
                  "연결 상태가 일시적으로 불안정합니다."
                );
            }
          },
          (message) => this.applyAddedProduct(target, message)
        );
      }
    }
    
    const [CartStoreProvider, useCartStore] = setupContext<CartStore>("CartStore");
    export { CartStore, CartStoreProvider, useCartStore };
    ```
    
    - 해당 스토어 객체에서 최종적으로 사용되는 setupContext는 컨텍스트와 관련된 훅을 다루는 유틸리티 함수이기 때문에 훅 영역의 로직으로 봐도 무방함
    - 위 코드에서 호출하는 addToCart는 API를 호출하는 함수. 그리고 내부에서는 addToCardRequest 시리얼라이저 함수를 호출하고 있음
- addToCardRequest 시리얼라이저 함수
    
    ```tsx
    // serializers/cart/addToCartRequest.ts
    
    import { AddToCartRequest } from "models/externals/Cart/Request";
    import { IRequestHeader } from "models/externals/lib";
    import {
      RecommendProduct,
      RecommendProductItem,
    } from "models/internals/Cart/RecommendProduct";
    import { Product } from "models/internals/Stuff/Product";
    
    interface Params {
      auths: IRequestHeader;
      cartProducts: Product[];
      shopID: number;
      target: RecommendProduct;
    }
    
    function addToCartRequest({
      auths,
      cartProducts,
      shopID,
      target,
    }: Params): AddToCartRequest {
      const productAlreadyInCart = cartProducts.find(
        (product) => product.getId() === target.getId()
      );
      return {
        body: {
          items: target.getItems().map((item) => ({
            itemId: item.id,
            quantity: getItemQuantityFor(productAlreadyInCart, item),
            salePrice: item.price,
          })),
          productId: target.getId(),
          shopId: shopID,
        },
        headers: auths,
      };
    }
    
    export { addToCartRequest };
    ```
    
    - AddToCardRequest 타입의 객체를 반환
    - 매개변수로 받는 target은 RecommendProduct 타입을 가짐
    - RecommendProduct 타입에 대한 정의는 모델 영역에 존재

### 3. 모델 영역

- RecommendProduct 타입에 대한 정의
    
    ```tsx
    // models/Cart.ts
    export interface AddToCartRequest {
      body: {
        shopId: number;
        items: { itemId: number; quantity: number; salePrice: number }[];
        productId: number;
      };
      headers: IRequestHeader;
    }
    
    /**
     * 추천 상품 관련 class
     */
    
    export class RecommendProduct {
      public getId(): number {
        return this.id;
      }
    
      public getName(): string {
        return this.name;
      }
    
      public getThumbnail(): string {
        return this.thumbnailImageUrl;
      }
    
      public getPrice(): RecommendProductPrice {
        return this.price;
      }
    
      public getCalculatedPrice(): number {
        const price = this.getPrice();
        return price.sale?.price ?? price.origin;
      }
    
      public getItems(): RecommendProductItem[] {
        return this.items;
      }
    
      public getType(): string {
        return this.type;
      }
    
      public getRef(): string {
        return this.ref;
      }
    
      constructor(init: any) {
        this.id = init.id;
        this.name = init.displayName;
        this.thumbnailImageUrl = init.thumbnailImageUrl;
        this.price = {
          sale: init.displayDiscounted
            ? {
                price: Math.floor(init.salePrice),
                percent: init.discountPercent,
              }
            : null,
          origin: Math.floor(init.retailPrice),
        };
        this.type = init.saleUnit;
        this.items = init.items.map((item) => {
          return {
            id: item.id,
            minQuantity: item.minCount,
            price: Math.floor(item.salePrice),
          };
        });
        this.ref = init.productRef;
      }
    
      private id: number;
      private name: string;
      private thumbnailImageUrl: string;
      private price: RecommendProductPrice;
      private items: RecommendProductItem[];
      private type: string;
      private ref: string;
    }
    ```
    
    - AddToCartRequest는 클래스로 표현된 객체로 추천 상품을 나타냄
    - AddToCartRequest  객체는 다른 컴포넌트 및 모델 객체와 함께 협력하게 됨

### 4. API 레이어 영역

- 훅에서 실제로 실행되는 addToCard 함수
    
    ```tsx
    // apis/Cart.ts
    
    // APIResponse는 데이터 로드에 성공한 상태와 
    // 실패한 상태의 반환 값을 제네릭하게 표현해주는 API 응답 객체
    // (APIResponse<OK, Error>)
    interface APIResponse<OK, Error> {
      // API 응답에 성공한 경우의 데이터 형식
      ok: OK;
      // API 응답에 실패한 경우의 에러 형식
      error: Error;
    }
    export const addToCart = async (
      param: AddToCartRequest
    ): Promise<APIResponse<string, string>> => {
      return (await GatewayAPI.post<IAddCartResponse>("/v3/cart", param)).map(
        (data) => data.message
      );
    };
    ```
    

### 정리

- 이처럼 각 객체에 컴포넌트, 훅, 모델, API 레이어 영역과 같이 적절한 역할과 책임을 할당하여 설계하는 것이 좋음
- 객체 지향 구현 자체 ≠ 클래스
    
    → 클래스는 객체를 표현하는 방법의 도구일 뿐. 컴포너트를 함수형으로 선언하든 클래스형으로 선언하든 모두 객체를 나타냄 
    
- 프로젝트를 어떤 방식이 가장 적합한 지 고민하면서 설계하는 노력이 필요

## 13.3 캡슐화와 추상화

- 올바른 협력을 설계하기 위해 적절한 캡슐화와 추상화가 필요
- 캡슐화
    - 다른 객체 내부의 데이터를 꺼내와서 직접 다루지 않고 해당 객체에게 처리할 행위를 따로 요청함으로써 협력하는 것
    - 컴포넌트는 객체이므로 컴포넌트의 내부 데이터인 상태(state)는 캡슐화의 대상이 될 수 있음
    → 컴포넌트 내의 상태와 prop을 잘 다루는 것도 캡슐화의 개념에 부합
- 추상화
    - 객체들을 모델링하는 과정 자체
    - 사람이 객체에 대해 쉽게 인지할 수 있도록 적합한 설계를 하는 것
- Prop drilling
    - Prop drilling이 심할 수록 컴포넌트 간 결합도가 증가. 내부 처리 로직이 외부로 드러날 확률 증가
        
        → Prop drilling은 좋지 못한 관계를 형성하게 하고 캡슐화를 저해함
        
    - 이런 문제를 해결하기 위해 옵저버 패턴이 등장했고 나아가 컨텍스트 API 및 Redux, Recoil과 같은 상태 관리 라이브러리가 생겨남
    → 이런 도구들을 활용하여 적절하게 캡슐화되고 추상화된 컴포넌트를 만들 수 있음.
- 객체 지향 패러다임에 매몰되지 않고 유기적인 협력 관계를 만드는 것과 명확하게 도메인을 분리하는 것에 대해 집중하다보면 객체지향이 추구하는 방향과 가까운 프론트엔드 개발을 할 수 있음

## 13.4 정리

- 객체 지향 패러다임의 원론적인 부분에 집중하기 보다 실제 코드를 작성하며 객체 지향을 경험하는 것이 중요
- 객체지향을 단순한 설계 방법이 아닌 현시대의 패러다임으로 보는 관점이 필요
    
    → 함수, 클래스, 모듈을 분리하는 것도 객체 지향 프로그래밍의 일부
    
- 객체 그 자체보다 객체의 책임을 고려할 것

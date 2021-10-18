# 1. 사용자 로그인

[POST] [https://dev-msa-oauth.***.com/oauth/token](https://dev-msa-oauth.***.com/oauth/token)

```jsx
client_id:clientapp
client_secret:secret
grant_type:password
username:****@itsdcode.com
password:********
```

로그인 할 때 token 정보가 생성됨

```jsx
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsiZGNvZGUtYXBpLXJlc291cmNlLWlkIl0sInVzZXJfbmFtZSI6IjYxMzEwIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImV4cCI6MTYyNDQyNjcxMSwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6IjczNDA0ZDQxLTZmZWUtNDdjNi04YWYyLTE3MjJhOThjZDBjYyIsImNsaWVudF9pZCI6ImNsaWVudGFwcCJ9.50uipeqVQwasBq10gpCm9YWhDis1m-AprIK3hjf8bPo",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsiZGNvZGUtYXBpLXJlc291cmNlLWlkIl0sInVzZXJfbmFtZSI6IjYxMzEwIiwic2NvcGUiOlsicmVhZCIsIndyaXRlIl0sImF0aSI6IjczNDA0ZDQxLTZmZWUtNDdjNi04YWYyLTE3MjJhOThjZDBjYyIsImV4cCI6MTYyNTYzNjMxMSwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6IjhhMGZiMmQyLThkOTktNDI3Ni1hODdjLTY4ZDcyYzI0NmJmZiIsImNsaWVudF9pZCI6ImNsaWVudGFwcCJ9.y_HzW2qhwz6TEDzNoRqD1Ud9mYHCsaqNdMJTlRawiWA",
    "expires_in": 86399,
    "scope": "read write",
    "jti": "73404d41-6fee-47c6-8af2-1722a98cd0cc"
}
```

기본적으로 한명의 유저로 테스트 진행(가능하면 여러 계정 테스트도 필요하다면 진행하나, 현재는 재고 차감 테스트이므로 한 계정으로도 큰 이슈가 없다고 판단함)

# 2. 부하요청건

## 2-1.  재고 소진 API

[GET] [https://dev-api-orders.***.com/orders/payment/choice](https://dev-api-orders.***.com/orders/payment/choice)

```jsx
storeId:itsdcode
merchantId:itsdcode
amount:1500
timestamp:20210621
paymentOrderId:testorder210621112151
customData:%7B%22orderId%22%3A0%2C%22addressId%22%3A84577%2C%22iamportId%22%3Anull%2C%22payMethod%22%3A%22card%22%2C%22usePoint%22%3A0%2C%22shippingFee%22%3A0%2C%22shippingText%22%3A%22%22%2C%22paymentOrderId%22%3A%22P_1624337711110_144602_843b81%22%2C%22userId%22%3A144602%2C%22paymentAmount%22%3A0.0%2C%22eventProductPriceId%22%3Anull%2C%22eventProductPriceListId%22%3Anull%2C%22linkprice%22%3Anull%2C%22productInfoList%22%3A%5B%7B%22productId%22%3A219622%2C%22color%22%3A%22BLACK%22%2C%22size%22%3A%22L%22%2C%22colorId%22%3Anull%2C%22sizeId%22%3Anull%2C%22count%22%3A1%7D%5D%2C%22couponCodeList%22%3A%5B%5D%2C%22finalAmount%22%3A3000%2C%22productInStockList%22%3Anull%2C%22productStockList%22%3A%5B%7B%22productId%22%3A219622%2C%22boutiqueId%22%3A1%2C%22productName%22%3A%22210521%20KINGMING%20TEST%20PRODUCT%20003%22%2C%22salePrice%22%3A3000.00%2C%22productStockId%22%3A311191%2C%22colorId%22%3A73%2C%22sizeId%22%3A1060%2C%22color%22%3A%22BLACK%22%2C%22size%22%3A%22L%22%2C%22shippingFee%22%3A0.00%2C%22orderCount%22%3A1%2C%22crawlerUpdatedAt%22%3A1621507684000%2C%22supplier%22%3A1212%2C%22ordering%22%3A3%7D%5D%7D
```

get 방식으로 paymentOrderId 만 랜덤으로 계속 변경되면 됨. 

paymentOrderId = testorder210621112151
였으면, testorder210621112152 와 같이 1을 증가시켜도 되고, 202111111 같은 그냥 랜덤숫자도 상관없음.

timestamp 와 customData 는 테스트 당일 셋팅되어야함. 

(customData 는 재고를 한 50개 정도 남긴 상품으로 셋팅된 임의의 셋팅값임)

## 2-2. 가상의 결제모듈 API

[GET] [https://dev-api-orders.***.com](https://dev-api-orders.***.com/orders/payment/choice)/orders/payment/kcp/payResultTest/{paymentOrderId}

KCP 결제창을 띄워 사람들이 결제를 시도했다고 가정하고 진행. paymentOrderId 는 위에 생성된 랜덤ID 값으로 호출 함.

## 2-3. 재고 복구 API

[DELETE] [https://dev-api-orders.***.com](https://dev-api-orders.***.com/orders/payment/choice)/orders/payment/cancel/{paymentOrderId}

KCP 결제창을 띄웠다가 취소하는 경우 위 API를 통해서 미리 할당한 재고를 복구함.

# 3. 부하테스트 CASE

## 3-1. 1차 러프한 부하 테스트

상품 재고를 50개 정도 설정된 상품 준비

2-1 → 2-2 순으로 순간 100건 이상의 요청콜을 순간적으로 발생시켜, 정상적으로 처리되는지 1차 확인.

데이터 확인 후 문제가 없다면 순간 1000건 이상의 요청콜로 2차 테스트 진행.

# 3-2. 2차 부하 테스트  
## (본 테스트부터 직접 시나리오 작성 및 실행, Given-When-Then 콘셉트 적용)

- 2차 테스트는 현실적인 트래픽 양과 SELECT, INSERT, UPDATE I/O가 섞인 환경을 테스트하고자 합니다

- **Given**
    
    2021년 5~6월 결제 시도 횟수를 참고로 10분에 2,000회 정도의 트래픽 준비
    
    인프라 환경(서버 수, 사양 등)을 운영 서버와 가능한 동일하게 설정
    
    상품 A: 800개
    
    상품 B: 1600개
    
    paymentOrderId: 중복이 생기지 않도록 생성 방식 개선
    

- **When**
    
    상품(A, B)과 테스트 케이스는 다음을 순차적으로 실행
    
    케이스 1: A 2개 구매
    
    케이스 2: B 3개 구매
    
    케이스 3: 케이스 1(A 2개) 복구
    
    케이스 4: A 3개, B 1개 구매
    
    통합 케이스는 1~4 전체를 말하며, 통합 케이스를 500회 실행 (총 주문 시도 2,000번)
    

- **Then**
    
    통합 케이스가 한 번 돌 때마다 A 3개, B 4개가 빠지게 됨
    
    통합 케이스 266회 실행 후 A 재고 마감
    
    이후 통합 케이스 약 178회 동안 케이스 2만 통과
    
    마지막으로 남은 56번의 통합 케이스는 모두 실패
    

## 예전 백**님이 부하 테스트 했던 포멧 공유

인수 테스트 할때 부하테스트 했었는데 참고 바랍니다. (사내 노션 문서)

# 4. 부하 테스트 결과

### 2021-06-23

[2021-06-23 결제 부하 테스트](사내 노션 문서)

### 2021-06-24

[2021-06-24 결제 부하 테스트](사내 노션 문서)

[2021-06-24 결제 부하 테스트 (order-list 1대)](사내 노션 문서)

# 5. 테스트 결과 정리

가장 중요한 이슈는 product_stock_id의 duplicate key 익셉션과 Deadlock, Connection TImeout(트랜잭션 생성 불가)였습니다.

(위 링크 중 4. 부하 테스트 결과 > 2021-06-23 참고)

원인은 아래 이유 때문이라고 판단했습니다.

**INSERT 경쟁 상황**

- A, B, C 순서로 같은 key값을 가지는 데이터를 INSERT 하기 위해 경쟁
- A 에서 `INSERT INTO table (pk) VALUES (3);` 실행하면 A에서 exclusive lock 획득
- B와 C에서도 동일한 INSERT 구문 실행한다. (B, C는 대기상태로 빠짐)
- A를 rollback하는 순간 B, C가 경쟁 시작.
- INSERT의 경우 exclusive lock을 획득 시도해야하지만, 해당 INSERT에서 **duplicated key error**가 발생하는 경우에는 해당 인덱스 레코드에 대해 일단 **shared lock**을 먼저 획득 시도하는 특성이 있다.
- B, C가 동일 인덱스 레코드에 대해 shared lock을 먼저 획득한 후 exclusive lock을 잡으려고 하기 때문에 데드락이 발생. (B가 exclusive lock을 획득하려 해도 C가 획득한 shared lock에 의해 불가능, C -> B의 경우에도 vice versa)
- 늦게 실행된 C가 deadlock처리되어 트랜잭션이 롤백되어 종료되면, B가 exclusive lock을 획득하여 실행된다.
- A를 commit하면 B, C는 바로 duplicated key error가 발생하지만 여전히 트랜잭션은 열려있다.
- 

결국 이런 INSERT 상황에서 선행 트랜잭션이 rollback하게되면 나머지 트랜잭션들 중 하나만 성공하고 나머지는 모두 데드락으로 강제 롤백 된다는 것을 알 수 있다. 이 케이스는 얼핏 봐서는 데드락이 발생하지 않을 것 같은 상황처럼 보이기 때문에 문제 발견이 쉽지 않으므로 기억해두는 것이 좋다.

실제로 아래처럼 코드로 직접 트랜잭션을 제어하면서 DB의 락 테이블을 조사해봤습니다

```java
do {
            DefaultTransactionDefinition def = new DefaultTransactionDefinition();
            def.setName("ReserveStockTx");
            def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
            TransactionStatus status = transactionManager.getTransaction(def);

            try {
                insertProductSoldStock(result);
                isSuccess = true;
            } catch (DataIntegrityViolationException e) {
                e.printStackTrace();

                log.info("ROLLBACK! > " + def.getName());
                transactionManager.rollback(status);

                // 누군가 먼저 재고를 사용(결제)해서 duplicate exception 발생하면 재고 재조회
                result = getAvailableProductStockList(productRootRepositoryParam);
                log.debug("productUseStockList.size(): {}", productReserveStockList.size());

            } catch (Exception e) {
                transactionManager.rollback(status);
                throw new DcodeApiException(e);
            }

        } while (!isSuccess);
```

### SELECT * FROM INNODB_LOCKS;

2372789033:4604:761:458, 2372789033, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
**2372789024:4604:761:458**, **2372789024**, **X**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789032:4604:761:458, 2372789032, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789031:4604:761:458, 2372789031, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789030:4604:761:458, 2372789030, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789029:4604:761:458, 2372789029, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789028:4604:761:458, 2372789028, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789027:4604:761:458, 2372789027, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789026:4604:761:458, 2372789026, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**
2372789025:4604:761:458, 2372789025, **S**, RECORD, `dcode`.`product_sold_stock`, unique_product_stock_id, 4604, 761, 458, **428744**

- 트랜잭션 10개가 동시에 실행 중이었고, **2372789024**가 X(Exclusive Lock)을 갖고 있습니다.

나머지 9개는 S(Shared Lock)인 상태입니다. (product_stock_id = 428744)

### SELECT * FROM INNODB_LOCK_WAITS;

2372789047, 2372789047:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789050, 2372789050:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789054, 2372789054:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789052, 2372789052:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789049, 2372789049:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789051, 2372789051:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789046, 2372789046:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789053, 2372789053:4604:761:458, **2372789024**, **2372789024:4604:761:458**
2372789048, 2372789048:4604:761:458, **2372789024**, **2372789024:4604:761:458**

- LOCK_WAITS에는 대기 중인 트랜잭션 9개가 기록되어있는데, 모두 X락을 가진 2372789024 트랜잭션에 의해 막혀있었습니다.

@Transactional을 사용하면 낮은 부하 상황에서는 Deadlock 익셉션이 발생하지 않아서, 이런 경우들을 스프링 자체적으로 효과적으로 처리하고 있었던 것으로 추정합니다.

## 다른방법 고민중

1. 상품상세 및 장바구니에서 결제 페이지로 이동 할때 재고를 잡아놓는다.(queue를 이용해서)
    - 결제 페이지로 이동시 버퍼를 둔다.
    - 재고 획득 하는 과정에서 버퍼를 줄수 있는 로직을 넣는다.(홀짝)
2. 결제 페이지를 와서 결제 버튼을 누르면 잡아 놓은 재고가 있는지 확인 후 결제창을 띄운다.

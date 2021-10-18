## 요약

- 앱 푸시, 카톡 푸시 후 msa-backend에서 장애 발생, 다운은 되지 않았습니다
- 이 상황의 로그 확인 중 각각 20~30건 정도 발생한 에러에 대한 분석을 남깁니다

## 영향

- 장애를 일으킨 주요 원인은 아니었습니다

## 원인

- More than one row with the given identifier was found: 64454, for class: dcode.domain.aggregate.order.ShippingInfo; nested exception is org.hibernate.HibernateException: More than one row with the given identifier was found: 64454, for class: dcode.domain.aggregate.order.ShippingInfo
    - 주문 개편 전에 DB 스키마가 shipping_info → order 로 N:1 참조를 하고 있었는데, 실제로 엔티티는 @OneToOne 매핑이 되어있어서 Order는 ShippingInfo를 1개만 참조하도록 되어있었습니다. 최초에는 이 둘이 1:1 관계였을 것 같은데 꾸준히 주문 로직을 개편하면서 N:1이 되고, 또 shipping_info ← order_list 와 같이 스키마 변화가 되었던 것 같습니다.
    - 그러나 msa-backend의 Order 엔티티는 업데이트가 되지 않았고, Order 엔티티를 조회한 후 ShippingInfo(컬렉션이 아닌 단수 필드)를 할당하려고하니 여러 row가 select되어서 오류가 발생했습니다.
    - 운영 DB에서 아래 쿼리를 실행하면 2606건이 발견되었으며, 엔티티 수정이 필요합니다.
    
    ```sql
    select count(*)
    from shipping_info si1, shipping_info si2
    where si1.id != si2.id and si1.order_id = si2.order_id;
    ```
    
- Unable to find dcode.domain.aggregate.product.ProductStock with id 478950; nested exception is javax.persistence.EntityNotFoundException: Unable to find dcode.domain.aggregate.product.ProductStock with id 478950
    - product_stock_old 테이블에서 id = 478950 찾았습니다.

## 계기

- 시스템이나 기능의 변경, 또는 테이블 변경 이력에 대해서 일부 코드가 현행화되지 않은 듯 합니다

## 해결(또는 추후 방안)

- 두가지 모두 AtomicUserOrderService#getLastOrder에서 발생했으므로... 해당 메서드와 관련 기능들에 대한 현행화와 테스트를 진행하면 될 것 같습니다.

## 배운점

- 시스템, 로직 변경이 발생할 경우 연쇄적으로 수정해야할 코드를 일일이 찾고 바꾸기가 쉽지 않다. 이런 경우 조금 더 수월하게 처리할 수 있는 시스템이나 장치가 없을지...

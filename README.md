![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 음식배달 이벤트 스토밍 결과
![모델링_v0 4](https://user-images.githubusercontent.com/8790281/206232529-a647bd8b-16d2-4299-9cb6-468ce976b6d4.png)

# 기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다
---
![신규주문등록](https://user-images.githubusercontent.com/8790281/206195387-3a4d6792-7172-49d5-a23a-6678dca162fb.png)
---
2. 고객이 결제한다
---
![결재완료](https://user-images.githubusercontent.com/8790281/206179445-e02622ad-a0c2-4d61-b1a7-6df00f5004e8.png)
---
3. 주문이 되면 주문 내역이 입점상점주인에게 전달된다
```
    /**
     * 주문이 되면 주문정보가 상점주에게 복사된다
     */
    public static void addOrder(OrderPlaced orderPlaced){

        /** Example 1:  new item */
        FoodCooking foodCooking = new FoodCooking();
        foodCooking.setAddress(orderPlaced.getAddress());
        foodCooking.setCustomerId(orderPlaced.getCustomerId());
        foodCooking.setFoodId(orderPlaced.getFoodId());
        foodCooking.setOrderId(String.valueOf(orderPlaced.getId()));
        foodCooking.setStatus("미결재");
        foodCooking.setStoreId(orderPlaced.getStoreId());
        repository().save(foodCooking);

    }

```
4. 상점주인이 확인하여 요리해서 배달 출발한다
* 상점주 주문 접수
---
![접수됨](https://user-images.githubusercontent.com/8790281/206190456-82c830cc-8e18-4c58-8225-83d1c2e114b7.png)
---
* 요리완료
---
![요리완료](https://user-images.githubusercontent.com/8790281/206190616-ffba411d-263d-44f2-85ba-969a204416d6.png)
---
* 배송시작
---
![배송중](https://user-images.githubusercontent.com/8790281/206190867-380617c4-6b1c-43a1-b92b-87a980cd11c0.png)
---
5. 고객이 주문을 취소할 수 있다
---
![주문취소](https://user-images.githubusercontent.com/8790281/206200706-c88e9490-d879-4390-a4b0-ec00734507cd.png)
---
6. 주문이 취소되면 배달이 취소된다
---
![배송취소](https://user-images.githubusercontent.com/8790281/206200726-199cdda7-3397-44d7-a94e-e2a8df36a816.png)
---
7. 고객이 주문상태를 중간중간 조회한다
* 결재됨(Mypage CQRS)
---
![CQRS_결재됨](https://user-images.githubusercontent.com/8790281/206201172-3fad01c8-5939-4183-ba53-539fec71cc13.png)
---
* 배송시작(Mypage CQRS)
---
![CQRS_배송시작](https://user-images.githubusercontent.com/8790281/206201356-cb2c2d19-fe3f-4a23-a520-a72e0b1884bb.png)
---
8. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
---
![알림](https://user-images.githubusercontent.com/8790281/206202282-80492590-06f6-477f-b390-95a9d44e8f50.png)
---

# 추가 구현사항
1. 고객이 리뷰를 작성할 경우 고객에게 쿠폰을 지급한다
* 리뷰 등록
---
![리뷰등록_리뷰_복수개](https://user-images.githubusercontent.com/8790281/206204448-3c4c6e96-aa9f-4b83-adaf-617c8a90e2c0.png)
---
* 고객 쿠폰 개수 증가
---
![리뷰등록_고객쿠폰증가_복수개](https://user-images.githubusercontent.com/8790281/206204485-a7561f00-d817-41ab-9dd0-a9a480cc07ff.png)
---

2. 고객이 리뷰를 삭제할 경우 고객의 쿠폰을 차감한다
* 리뷰 삭제(카프카 consumer 로그)
---
![리뷰삭제_카프카consumer](https://user-images.githubusercontent.com/8790281/206204307-2996eda3-f149-4991-bede-52f0f0af99e7.png)
---
* 고객 쿠폰 개수 감소
---
![리뷰삭제_쿠폰삭제](https://user-images.githubusercontent.com/8790281/206204362-2928c2c9-835a-482a-8c6f-a7b629ec0f12.png)
---
 
 

# 체크포인트
1. Saga (Pub / Sub)
- kafka를 통한 Pub/Sub 비동기 통신
* Publish 예제 코드
```
@PostPersist
public void onPostPersist(){
    // 주문생성 이벤트 pub
    OrderPlaced orderPlaced = new OrderPlaced(this);
    orderPlaced.publishAfterCommit();

    // 주문취소 이벤트 pub
    OrderCanceled orderCanceled = new OrderCanceled(this);
    orderCanceled.publishAfterCommit();
}

```
* Subscribe 예제 코드
```
@StreamListener(value=KafkaProcessor.INPUT, condition="headers['type']=='OrderPlaced'")
public void wheneverOrderPlaced_Pay(@Payload OrderPlaced orderPlaced){

    OrderPlaced event = orderPlaced;
    System.out.println("\n\n##### listener Pay : " + orderPlaced + "\n\n");

    // Sample Logic //
    Payment.pay(event);
}

@StreamListener(value=KafkaProcessor.INPUT, condition="headers['type']=='CookFinished'")
public void wheneverCookFinished_UpdateStatus(@Payload CookFinished cookFinished){

    CookFinished event = cookFinished;
    System.out.println("\n\n##### listener UpdateStatus : " + cookFinished + "\n\n");

    // Sample Logic //
    Order.updateStatus(event);
}
```
* 요리 완료 후 완료 정보 Publish 후 라이더 서비스는 요리 완료 정보 Subscribe 후 배송 가능 상태로 변경
---
![요리완료](https://user-images.githubusercontent.com/8790281/206218095-7ad0238f-e609-42e8-93b7-645b39514cf0.png)
---
---
![배송가능](https://user-images.githubusercontent.com/8790281/206218214-ce9f0c82-2ed7-4ca0-be8a-8d7ca341d978.png)
---

2. CQRS
- Mypage를 통한 오더상태 업데이트 정보 조회
* 결재됨
---
![CQRS_결재됨](https://user-images.githubusercontent.com/8790281/206218505-f48277b3-0fbd-4307-9b83-42ba20f1cbe3.png)
---
* 상점주 접수
---
![CQRS_접수됨](https://user-images.githubusercontent.com/8790281/206218960-96e24363-87e9-42d2-aac1-2749653f24a4.png)
---
* 라이더 배송시작
---
![CQRS_배송시작](https://user-images.githubusercontent.com/8790281/206219027-b92cfab8-8b6a-4583-9444-70c22151b36f.png)
---
* 고객 배송확정
---
![CQRS_배송확정](https://user-images.githubusercontent.com/8790281/206219103-0527eccb-5568-487e-9355-5f938cc15c16.png)
---
 
3. Compensation / Correlation
* 고객이 작성한 리뷰 삭제 시 부여했던 쿠폰 개수 차감
```
public static void removeCoupon(ReviewDeleted reviewDeleted){
    /** 고객은 리뷰 삭제 시 기존 쿠폰에서 1개 차감한다. */
    repository().findByCustomerId(reviewDeleted.getCustomerId()).ifPresent(customer->{
        customer.setCouponCnt(customer.getCouponCnt() - 1); // remove coupon
        repository().save(customer);
     });
}
```
---
![리뷰삭제_카프카consumer](https://user-images.githubusercontent.com/8790281/206219333-7ffcb446-de40-4de9-a85f-ac5d4a3ba12e.png)
---
---
![리뷰삭제_쿠폰삭제](https://user-images.githubusercontent.com/8790281/206219482-33091545-db87-49c1-a36e-7a677cb1a1d0.png)
---
* 요리시작 또는 요리종료 상태인 경우 주문취소 불가, 그 외의 상태는 주문취소 가능 
```
@PreRemove
public void onPreRemove(){
    if (status.equals("요리시작") || status.equals("요리완료")) {
        throw new RuntimeException("요리가 시작되기 전에만 주문취소가 가능합니다.");
    }

    // 주문취소 이벤트 pub
    OrderCanceled orderCanceled = new OrderCanceled(this);
    orderCanceled.publishAfterCommit();
}
```
---
![요리시작취소불가](https://user-images.githubusercontent.com/8790281/206214892-7b6cad84-567f-416e-aecf-710bf6f3e68f.png)
---

4. Request / Response
- 리뷰 저장 전(onPrePersist메소드)에 해당 오더번호의 고객ID를 동기식 REST방식으로 호출하며 리뷰 저장 후 해당 고객에게 쿠폰 증정
```
@PrePersist
public void onPrePersist() {
    // req/res 형식으로 오더정보조회 후 해당 오더의 고객ID 설정
    food.external.Order order =
       FrontApplication.applicationContext.getBean(food.external.OrderService.class)
       .getOrder(Long.valueOf(getOrderId()));

    setCustomerId(order.getCustomerId());

    ReviewWritten reviewWritten = new ReviewWritten(this);
    reviewWritten.publishAfterCommit();
}
```

5. Circuit Breaker
- 리뷰 작성 시 고객의 ID를 알기 위해 해당 오더정보 조회를 req/res 방식으로 호출하며, 조회 시 특정 시간 이상 경과할 경우 서킷 브레이크 발생
---
![req_res조회](https://user-images.githubusercontent.com/8790281/206234523-563308ad-40b4-4463-b90d-d400a544b397.png)
---

- front서비스의 application.yml의 hystrix enable은 true로 timeout은 500ms로 설정
```
feign:
  hystrix:
    enabled: true
hystrix:
  command:
    default:
      execution.isolation.thread.timeoutInMilliseconds: 500
```
- 오더 객체 로드 시 강제 delay 발생(랜덤으로 400ms에서 600ms 미만의 초)
```
@PostLoad
public void makeDelay(){
    try {
        Thread.currentThread().sleep((long) (400 + Math.random() * 200));

    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
* seige를 이용하여 5초간 리뷰작성 호출 로그 : 0.5초 이상 지연된 경우 500에러 발생
---
![서킷브레이크1](https://user-images.githubusercontent.com/8790281/206231169-e86f5587-34da-4498-a151-22c1af2efba5.png)
---
---
![서킷브레이크2](https://user-images.githubusercontent.com/8790281/206231190-3a8d5c8e-6b7d-46b9-83be-4d8e3b152f88.png)
---

6. Gateway / Ingress
- gateway 서비스 8088 포트를 통해 브라우저에서 path만 변경 후 각 서비스로 정상 라우팅 확인
* 8081 포트 (front 서비스)
---
![게이트웨이_8081](https://user-images.githubusercontent.com/8790281/206215850-a3f6dd4d-f96f-4426-9229-7fc33e316c0a.png)
---

* 8082 포트 (store 서비스)
---
![게이트웨이_8082](https://user-images.githubusercontent.com/8790281/206215932-5e818de4-bdff-4f3d-b0e1-7945800ac591.png)
---

* 8083 포트 (rider 서비스)
---
![게이트웨이_8083](https://user-images.githubusercontent.com/8790281/206216003-afec408f-189c-432c-b68e-7e3f67c68e85.png)
---

* 8084 포트 (notification 서비스)
---
![게이트웨이_8084](https://user-images.githubusercontent.com/8790281/206216064-5dcc2df9-24bf-4c01-a2bb-7da4cf79caf8.png)
---

* 8085 포트 (customer 서비스)
---
![게이트웨이_8085](https://user-images.githubusercontent.com/8790281/206216118-aa119c62-98f2-43e1-aab0-71e4882d38a3.png)
---

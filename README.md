![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 예제 - 음식배달

본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다.
이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# 서비스 시나리오

배달의 민족 커버하기 - https://1sung.tistory.com/106

# 이벤트 스토밍 결과
![모델링_v0 3](https://user-images.githubusercontent.com/8790281/206177015-b10e15b2-cc95-4210-98d7-1e1a23a89574.png)

# 기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다
---
![오더주문](https://user-images.githubusercontent.com/8790281/206177958-d0157419-24fe-4475-80e1-aa3abf5bbd5a.png)
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

6. 고객이 주문을 취소할 수 있다
7. 주문이 취소되면 배달이 취소된다
8. 고객이 주문상태를 중간중간 조회한다
9. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다

# 추가 구현사항
1. 고객이 리뷰를 작성할 경우 고객에게 쿠폰을 지급한다

2. 고객이 리뷰를 삭제할 경우 고객의 쿠폰을 차감한다

 
 

# 체크포인트
1. Saga (Pub / Sub)
- kafka를 통한 Pub/Sub 비동기 통신

2. CQRS
- Mypage를 통한 오더상태 업데이트 정보 조회
 
3. Compensation / Correlation
- 요리중, 요리완료 상태이면 취소 불가, 그 외 상태 취소 가능

4. Request / Response
- 결제 처리 동기 호출

5. Circuit Breaker
- 주문에서 결제 호출을 req/res 방식으로 호출하며, 결재 시 많은 특정 시간 이상 경과 시 서킷 브레이크 발생

6. Gateway / Ingress
- gateway 서비스 > src > main> resources > application.yml

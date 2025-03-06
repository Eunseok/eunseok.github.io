문제를 해결한 원인에 대해 설명드리겠습니다. `Slots = new List<ItemSlotData>(slotCount);`에서 `Slots = new List<ItemSlotData>();`로 변경하였을 때 정상적으로 동작했다면, 이는 **리스트 생성자(`List<T>(int capacity)`)와 관련된 동작 차이**에서 기인한 가능성이 큽니다.

------

### **차이점 분석**

`new List<ItemSlotData>(slotCount);`와 `new List<ItemSlotData>();`의 동작 차이는 다음과 같습니다:

1. **`List<T>(int capacity)` 생성자**
   - 이 생성자는 특정 용량(`capacity`)을 미리 지정한 상태로 리스트를 생성합니다.
   - 하지만 용량을 설정한다고 해서 실제 데이터가 채워지는 것은 아닙니다.
   - 리스트는 초기화되었지만 **내부 데이터는 여전히 비어 있고**, 항목은 존재하지 않습니다.
   - 결과적으로, 리스트의 `Count`는 0이지만 내부적으로 `Capacity`만 설정된 상태가 됩니다.
2. **`List<T>()` 기본 생성자**
   - 기본 생성자는 용량을 설정하지 않는 빈 리스트를 생성합니다.
   - 내부적으로 시스템이 필요할 때 용량을 동적으로 증가시킵니다.
   - 따라서, 데이터 항목 추가와 관련된 동작은 동일하게 처리됩니다.

------

### **여기서 발생한 문제 가능성**

#### 1. 리스트 용량 초기화와 Unity의 직렬화 상호작용 문제

Unity는 직렬화/역직렬화 과정에서 `List<T>(int capacity)`로 설정된 용량과 내부 데이터를 처리할 때 특정 데이터를 잘못 초기화하거나, 내부 참조를 덮어쓰기하는 문제가 있을 가능성이 있습니다.

Unity는 `Capacity`만 설정된 리스트를 적절히 처리하지 못하고, 결과적으로 **`OnItemChanged` 같은 참조 타입의 초기화 상태를 기본값(null)로 덮어쓰고 있을 가능성**이 있습니다.
 이는 Unity 객체의 직렬화/역직렬화 방식과 관련된 문제로 보입니다.

#### 2. 잘못된 초기화로 슬롯이 누락되었을 가능성

`List<ItemSlotData>(slotCount)`를 사용했을 때, 내부적으로 해당 슬롯이 비어 있는 상태로 저장된 상황이 발생했을 가능성도 있습니다.

- 리스트의 용량(`Capacity`)이 14로 초기화되었더라도, **슬롯 데이터를 추가하지 않으면 빈 상태**입니다.
- 빈 슬롯(`null` 상태)을 가짐으로 인해 `itemSlotData.OnItemChanged` 필드에 접근할 때 null로 출력될 가능성이 생깁니다.

#### **결론:** Unity와의 상호작용 문제

Unity는 내부적으로 값을 직렬화하거나, Inspector를 통해 데이터를 복구할 때 **`Capacity`만 지정된 리스트**를 잘못 처리할 가능성이 매우 큽니다. 반면, 빈 리스트(`new List<T>()`)는 문제가 발생하지 않으므로 이 코드가 정상적으로 실행된 것으로 판단됩니다.

------

### **정확한 원인 탐구를 위한 실험**

다음 두 가지 실험을 통해 문제의 원인을 더 명확히 확인할 수 있습니다:

#### 1. `Capacity` 설정 후, 슬롯 데이터를 명시적으로 추가

`List<ItemSlotData>(slotCount)`로 다시 작성한 뒤, 실제 데이터를 명시적으로 추가했을 때 문제가 해결되는지 확인합니다:

```
Slots = new List<ItemSlotData>(slotCount); // Capacity 14 설정
for (int i = 0; i < slotCount; i++)
{
    var itemSlotData = new ItemSlotData();
    itemSlotData.OnItemChanged += OnInventoryChanged;

    Debug.Log($"슬롯 {i}: {(itemSlotData.OnItemChanged == null ? "null" : "구독자 있음")}");
    Slots.Add(itemSlotData);
}
```

이 실험에서 문제가 발생하지 않는다면, 원인은 **Unity와 직렬화된 초기 상태와 관련된 누락/갱신 문제**로 좁혀집니다.

------

#### 2. 리스트의 `Capacity`와 `Count` 확인

Unity 직렬화/역직렬화 과정 또는 초기화 로직에서 `Capacity`만 지정된 리스트가 잘못 처리되지 않았는지 확인합니다:

```
Debug.Log($"Slots 용량(Capacity): {Slots.Capacity}, 항목 수(Count): {Slots.Count}");
```

- `Capacity`는 설정값으로 14이지만, `Count`는 실제로 추가된 항목에 따라 달라집니다.
- 만약 `Capacity`가 설정되었더라도 `Count`가 0으로 남아 있다면, 이는 슬롯 데이터를 추가하지 않았기 때문입니다.

------

### **해결 방법**

#### 추천 코드

이 문제를 회피하려면 굳이 `Capacity`를 설정하지 말고, 다음과 같이 작성하는 것이 적절합니다:

```
Slots = new List<ItemSlotData>(); // 빈 리스트 초기화
for (int i = 0; i < slotCount; i++)
{
    var itemSlotData = new ItemSlotData();
    itemSlotData.OnItemChanged += OnInventoryChanged;

    Debug.Log($"슬롯 {i}: {(itemSlotData.OnItemChanged == null ? "null" : "구독자 있음")}");
    Slots.Add(itemSlotData);
}
```

#### 추가 설명: Unity 특성을 고려한 일반적인 지침

1. Unity와 연동된 코드에서는 가급적 

   `List<T>` 초기화를 간단하게 작성

   하는 것이 좋습니다.

   - `List<T>(int capacity)`는 Unity의 직렬화 시스템에서 데이터 손상을 발생시킬 가능성이 있기 때문입니다.

2. **실제 데이터를 추가**하지 않는 리스트는 Unity가 잘못된 직렬화를 수행할 수 있으므로 직접 추가하는 것이 안전합니다.

------

### 요약

- `new List<T>(int capacity)`는 단순히 용량만 지정하고 **데이터는 추가하지 않은 상태를 유지**합니다.
- Unity의 직렬화 과정에서 리스트 데이터를 잘못 처리했기 때문에 슬롯 데이터(`itemSlotData`)의 필드 초기화도 덮어쓰기된 것으로 보입니다.
- `new List<T>()`를 통해 빈 리스트로 초기화하면 이러한 문제를 회피할 수 있습니다.

추가로 더 확인하고 싶은 내용이 있다면 질문해주세요! 😊
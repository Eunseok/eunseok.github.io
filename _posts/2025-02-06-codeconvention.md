---
layout: "single"
title: "코드 컨벤션"
---

# 🫡 C# 코드 컨벤션 및 스타일 가이드

---

> 해당 내용은 [유니티 코드 스타일 가이드 및 전자책](https://unity.com/kr/blog/engine-platform/clean-up-your-code-how-to-create-your-own-c-code-style) 내용을 참고 하였습니다.

###### 코드 컨벤션이 뭔데? 👀

[[참고] C# 문법 종합반 1주차  3).3 변수명 ](https://teamsparta.notion.site/3-148becd7f58542d49a54713e25e723b9)

**1️⃣ 클린 코드의 중요성**

​	•	클린 코드는 **가독성, 유지보수성, 확장성**을 높이는 코드이다.

​	•	팀원 간 일관된 코딩 스타일을 유지하면 협업과 유지보수가 쉬워진다.



**2️⃣ 코딩 원칙**

​	•	**KISS (Keep It Simple, Stupid)**: 코드의 복잡성을 최소화하고 단순한 해결책을 우선한다.

​	•	**YAGNI (You Aren’t Gonna Need It)**: 필요하지 않은 기능을 미리 구현하지 않는다.

​	•	**일관성 유지**: 코드 스타일, 네이밍, 파일 구조를 통일한다.

​	•	**점진적인 개선**: 매일 조금씩 코드를 개선하는 습관을 가진다.



**3️⃣ 네이밍 규칙**

​	•	**클래스 & 인터페이스**: PascalCase, 인터페이스는 I 접두어 사용 (IExample)

​	•	**메서드 & 프로퍼티**: PascalCase (DoSomething(), GetName())

​	•	**변수 (로컬 & 필드)**: camelCase (myVariable), private 필드는 _camelCase

​	•	**상수**: ALL_CAPS (MAX_SPEED)

​	•	**제네릭 타입**: T + PascalCase (TItem)

- **변수 선언 예시**

```csharp
public class Player
{
    private int _health;  // _camelCase (Private 필드)
    public int Health { get; set; } // PascalCase (Property)
    
    public void TakeDamage(int damageAmount) // PascalCase (Method)
    {
        int newHealth = _health - damageAmount; // camelCase (Local 변수)
    }
}
```



**4️⃣ 코드 스타일**

​	•	**4칸 공백**을 들여쓰기로 사용하고 `tab` 대신 `space`를 적용한다.



​	•	 **파일 범위 네임스페이스** 사용 (namespace MyNamespace;)

- 들여쓰기 & 네임 스페이스 예시

  ```csharp
  namespace MyNamespace;
  
  public class ExampleClass
  {
      public void DoSomething()
      {
          Console.WriteLine("Hello, World!");
      }
  }
  ```

  

​	•	**var 사용 원칙**: 기본 타입은 명시적으로 선언하고 나머지는 var 사용.

​	•	**코드 정리**: 불필요한 using 제거, Ctrl + K, Ctrl + D로 정리.

- **var 사용  예시**

```csharp
//(int, string, bool 등)은 명시적으로 선언
int count = 10;
string name = "John";

var list = new List<int>();	// 나머지는 var 선언
var player = new Player();
```

######  var이 뭐지? 👀

[[참고] C# 문법 종합판 1주차  3).6 var 키워드 사용법](https://teamsparta.notion.site/3-148becd7f58542d49a54713e25e723b9)



**5️⃣ 이벤트 & 핸들러**

​	•	이벤트는 **동작을 설명하는 동사 구**(DoorOpened)로 네이밍.

​	•	이벤트를 호출하는 메서드는 On 접두사를 사용 (OnDoorOpened()).

​	•	이벤트 핸들러는 **이벤트 소스와 언더스코어(_)를 포함** (GameEvents_DoorOpened()).

- **이벤트 & 핸들러 예시**

```csharp
public event Action DoorOpened;

public void OnDoorOpened()	//동사로 네이밍
{
    DoorOpened?.Invoke();
}

public void GameEvents_DoorOpened() // 이벤트 소스와 언더스코어(_)를 포함 (게임 이벤트's_문열림)
{
    Console.WriteLine("The door was opened!");
}
```

######  이벤트 & 핸들러가 뭐지? 👀

[[참고] C# 문법 종합반 4주차  3).1 델리게이트 ( Delegate ), 3).3 Func과 Action ](https://teamsparta.notion.site/3-LINQ-45196675ac6042169284090c46e8c32d)



**6️⃣ 메서드 및 클래스 설계**

​	•	**단일 책임 원칙 (SRP)**: 클래스와 메서드는 하나의 역할만 담당해야 한다.

​	•	**메서드 크기 제한**: 한 가지 역할만 수행하는 짧고 명확한 메서드를 작성한다.

​	•	**가독성 유지**: 불필요한 중첩을 피하고 if 문, switch 문을 정리한다.

- 가독성 유지 예시

```csharp
// ❌ 잘못된 예시 (불필요한 중첩)
if (condition)
{
    if (anotherCondition)
    {
        DoSomething();
    }
}

// ✅ 올바른 예시 (가독성 개선)
if (condition && anotherCondition)
{
    DoSomething();
}
```



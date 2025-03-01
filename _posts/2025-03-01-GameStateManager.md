---
layout: single
title: "GameStateManager"
categories: [Framework]
tags: [C#, Framework]
typora-root-url: ../

---

### **개요**

`GameStateManager` 클래스는 게임의 현재 상태를 효율적으로 관리하기 위한 유틸리티로 작성되었습니다.
  주요 특징:

1. 게임 로직에서 사용할 **게임 상태(Enum)** 정의.
2. 게임 상태의 변경을 **싱글톤 패턴**으로 관리하여 전역적으로 접근 가능.
3. 상태 변경 이벤트를 통해 상태 변화 처리(notification) 가능.
4. 특정 상태 전환에 따라 동작을 동적으로 달리 처리할 수 있는 구조 제공.

------

### **코드 구성 요소**

#### 1. **`GameState` 열거형**

```
public enum GameState
{
    MainMenu,
    Gameplay,
    Paused,
    GameOver
}
```

- 게임 내에서 사용되는 상태를 **명시적으로 구분**하기 위해 `Enum`으로 정의.
- 상태 목록:   
  - **MainMenu**: 메인 메뉴에서의 상태.
  - **Gameplay**: 게임플레이가 진행 중인 상태.
  - **Paused**: 게임이 일시 정지된 상태.
  - **GameOver**: 게임이 종료된 상태.

이처럼, 명확히 구분된 상태를 통해 코드 가독성과 상태 관리의 명확성을 높입니다.

------

#### 2. **싱글톤 패턴**

```
public static GameStateManager Instance; // 싱글톤 패턴
```

- `GameStateManager`를 **전역적으로 접근 가능**하도록 하는 정적 인스턴스 변수.
- 게임 내의 모든 코드에서 현재 게임 상태를 참조하거나 변경할 수 있도록 설계.

#### **Awake 메서드에서 싱글톤 구현**

```
private void Awake()
{
    if (Instance == null)
    {
        Instance = this;
        DontDestroyOnLoad(gameObject); // 다른 씬에서도 유지
    }
    else
    {
        Destroy(gameObject);
    }
}
```

- 싱글톤 패턴 구현:   
  1. 이미 인스턴스가 존재한다면 새로 생성된 인스턴스를 파괴(`Destroy`).
  2. 처음 호출 시에는 `Instance`를 초기화하고, 씬 전환에도 **파괴되지 않도록 유지**(`DontDestroyOnLoad`).
- 게임의 전역 상태를 한 곳에서 접근/관리할 수 있도록 보장.

------

#### 3. **현재 게임 상태 관리**

```
private GameState currentState; // 현재 상태
```

- 게임의 현재 상태를 저장하는 변수.
- 이를 통해 현재 상태를 확인하거나 변경 가능.

------

#### 4. **상태 변경 이벤트**

```
public event Action<GameState> OnGameStateChanged;
```

- game state가 변경될 때 이를 구독(subscribe)한 다른 객체들에게 알림(notification)을 보냅니다.

- `Action<GameState>`

  :   

  - `GameState`를 매개변수로 전달하는 이벤트 델리게이트.
  - 현재 상태가 변경될 때 이를 다른 스크립트나 시스템에서 동적으로 처리 가능.

------

#### 5. **SetGameState 메서드**

```
public void SetGameState(GameState newState)
{
    if (currentState != newState)
    {
        currentState = newState;
        OnGameStateChanged?.Invoke(currentState); // 상태 변경 이벤트 트리거
        Debug.Log($"Game State changed to: {currentState}");
    }
}
```

- 기능

  :   

  1. 호출된 `newState`가 현재 상태와 다를 경우만 상태를 변경합니다.
  2. 상태가 변경되면 `OnGameStateChanged` 이벤트를 트리거합니다.
  3. 상태 변경 내용을 디버그 로그로 출력.

- 핵심 로직

  :   

  - 중복 상태 변경 방지(`if (currentState != newState)`).
  - 상태 변경 시 등록된 모든 구독자들에게 알림(`OnGameStateChanged?.Invoke(currentState)`).

#### 6. **GetGameState 메서드**

```
public GameState GetGameState()
{
    return currentState;
}
```

- 현재 상태를 반환하는 getter 메서드.
- 상태 확인을 위한 간단한 유틸리티.

------

### **클래스 전체 동작 흐름**

1. **싱글톤 초기화**:
   - `Awake` 메서드에서 `Instance`를 초기화하고 씬 전환에도 파괴되지 않도록 설정.
   - 프로젝트에서 `GameStateManager.Instance`를 통해 언제든 전역적으로 접근 가능.
2. **상태 변경**:
   - `SetGameState`를 호출하여 새로운 상태를 설정.
   - 상태가 변경되면 이벤트(`OnGameStateChanged`)가 트리거됩니다.
   - 이벤트에 등록된 모든 구독자는 상태 변화를 인지하고 자신만의 로직을 실행할 수 있습니다.
   - 새롭게 설정된 상태는 `currentState`에 저장되며, 필요시 `GetGameState`로 확인 가능.
3. **상태 전환 이벤트 사용**:
   - 외부에서 `OnGameStateChanged`에 동적으로 구독을 추가하여 상태 변화에 따라 특정 작업 수행 가능.
   - 예:

```
GameStateManager.Instance.OnGameStateChanged += HandleGameStateChange;
```

------

### **장점**

1. **확장성**:
   - 새로운 상태를 추가하려면 `GameState` 열거형에 새로운 상태를 정의하면 됩니다.
   - 이벤트 기반 구조이므로 상태 변화에 따른 동작 추가가 유연.
2. **전역적 관리**:
   - 싱글톤을 사용해 프로젝트 어디서든 상태를 확인하고 변경 가능.
3. **코드 간소화**:
   - 게임 상태마다 관련 로직을 명확히 구분할 수 있어 코드의 유지보수성 향상.
4. **상태 관련 통합 이벤트 처리**:
   - 모든 상태 변경에 대해 이벤트를 트리거해 다양한 요소에서 변경 사항을 즉시 감지하고 반응 가능.

------

### **예시 사용법**

#### **상태 변경**

```csharp
GameStateManager.Instance.SetGameState(GameStateManager.GameState.Gameplay);
```

#### **현재 상태 확인**

```csharp
if (GameStateManager.Instance.GetGameState() == GameStateManager.GameState.Gameplay)
{
    Debug.Log("게임이 진행 중입니다!");
}
```

#### **상태 변화 이벤트 처리**

다른 스크립트에서 게임 상태 변화를 처리하고 싶을 때:

```csharp
void Start()
{
    GameStateManager.Instance.OnGameStateChanged += HandleGameStateChange;
}

void HandleGameStateChange(GameStateManager.GameState newState)
{
    if (newState == GameStateManager.GameState.Paused)
    {
        Debug.Log("게임이 일시 정지되었습니다!");
    }
}
```

------

### **결론**

`GameStateManager`는 게임 상태를 전역적으로 관리할 수 있는 유용한 컴포넌트입니다. 확장성과 코드 간소화를 위해 싱글톤 및 이벤트 기반으로 설계되었으며, 게임플레이 로직을 일관되게 유지하는 데 도움을 줍니다

<details markdown="1"> <summary>코드스니펫</summary>

  ```csharp
public class GameStateManager : MonoBehaviour
{
    // 게임 상태 정의
    public enum GameState
    {
        MainMenu,
        Gameplay,
        Paused,
        GameOver
    }

    public static GameStateManager Instance; // 싱글톤 패턴

    private GameState currentState; // 현재 게임 상태

    // 상태 변경 이벤트 (구독 가능)
    public event Action<GameState> OnGameStateChanged;

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject); // 다른 씬에서도 유지
        }
        else
        {
            Destroy(gameObject);
        }
    }

    // 게임 상태를 설정하고 이벤트 호출
    public void SetGameState(GameState newState)
    {
        if (currentState != newState)
        {
            currentState = newState;
            OnGameStateChanged?.Invoke(currentState); // 상태 변경 이벤트 트리거
            Debug.Log($"Game State changed to: {currentState}");
        }
    }

    // 게임 상태를 반환
    public GameState GetGameState()
    {
        return currentState;
    }
}
  ```

</details>

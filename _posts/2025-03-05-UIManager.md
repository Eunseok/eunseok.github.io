---
layout: single
title: "UIManager"
categories: [Framework]
tags: [C#, Framework]
typora-root-url: ../

---

# UIManager

`UIManager` 클래스는 게임 디스플레이와 관련된 UI를 효율적으로 관리하기 위한 Singleton 패턴 기반의 UI 관리 클래스로, HUD, Popup을 생성하고 관리하는 기능을 제공합니다.

## 주요 기능

- **HUD 및 Popup UI 관리**
  - HUD와 Popup UI를 로드하고 생성하는 기능.
  - Popup UI를 스택(Stack) 구조로 관리하여 계층적인 Popup 처리 가능.
  - UI의 `Canvas` 정렬 및 정렬 순서(`sortingOrder`) 관리.
- **UI 리소스 로드**
  - `Resources` 폴더에서 UI 프리팹을 로드하여 동적으로 인스턴스화.
- **Popup 닫기**
  - 스택에서 Popup을 제거하거나 모든 Popup을 한번에 제거.

------

## 클래스 구조

### 상수

- `HUDPath`: HUD 프리팹이 위치한 리소스 경로.
- `PopupPath`: Popup 프리팹이 위치한 리소스 경로.

### 필드

- `_currentOrder`: 현재 UI의 정렬 순서를 저장하며, 오더를 증가시키며 UI 생성.
- `_popupStack`: Popup UI를 관리하기 위한 Stack 구조.
- `_hudUI`: 현재 활성화된 HUD의 참조.

------

## 주요 메서드

### HUD 관련 메서드

- `ShowHud<T>()`
  - 제네릭 타입 `T`를 받아 특정 `UIHud`를 표시합니다.
  - 사용 방법: `ShowHud<MyHUD>();`
- `ShowHud(string hudName)`
  - HUD의 이름(`hudName`)으로 UI를 로드하고 표시합니다.
  - 사용 방법: `ShowHud("MyHUD");`

### Popup 관련 메서드

- `ShowPopup<T>()`
  - 제네릭 타입 `T`를 받아 특정 `UIPopup`를 표시합니다.
  - 사용 방법: `ShowPopup<MyPopup>();`
- `ShowPopup(string popupName)`
  - Popup의 이름(`popupName`)으로 UI를 로드하고 표시합니다.
  - 사용 방법: `ShowPopup("MyPopup");`
- `ClosePopup(UIPopup popup)`
  - 인자로 전달된 Popup을 닫습니다.
- `ClosePopup()`
  - 현재 스택 최상단에 있는 Popup을 닫습니다.
- `CloseAllPopup()`
  - 스택에 쌓인 모든 Popup을 닫습니다.

------

### UI 관리 관련 메서드

- ```csharp
  SetCanvas(GameObject go, bool sort = true)
  ```

  - 전달된 GameObject가 Canvas를 포함하도록 설정하고, 필요한 경우 정렬 순서를 적용합니다.

------

## 사용 예시

### HUD 표시

```csharp
UIManager.Instance.ShowHud<MyHUD>();
```

### Popup 표시 및 닫기

```csharp
// Popup 표시
var popup = UIManager.Instance.ShowPopup<MyPopup>();

// 특정 Popup 닫기
UIManager.Instance.ClosePopup(popup);

// 최상단 Popup 닫기
UIManager.Instance.ClosePopup();

// 모든 Popup 닫기
UIManager.Instance.CloseAllPopup();
```

------

## 구현 세부 사항

- **싱글턴 패턴**
  `UIManager`는 단일 인스턴스로 동작하며, `Singleton<T>` 기반으로 구현되어 있습니다.
- **UI 루트 관리**
  - `@UI_Root`라는 이름의 GameObject를 자동으로 생성하여 UI 계층 구조의 최상단으로 사용.
  - Root 오브젝트가 이미 존재하면 재활용.
- **리소스 로딩**
  - `Resources.Load`를 사용하여 HUD 및 Popup UI 프리팹을 동적으로 로드.
  - 경로가 잘못되었거나 리소스가 유효하지 않을 경우 에러를 출력.

------

### 참고

- HUD는 FPS, 체력 등 지속적으로 화면에 표시되는 정보를 관리합니다.
- Popup은 메시지 박스, 설정 창 등 일시적인 UI 요소를 관리합니다.

<details markdown="1"> <summary>코드스니펫</summary>

  ```csharp
public class UIManager : Singleton<UIManager>
{
    private const string HUDPath = "UI/Hud/";
    private const string PopupPath = "UI/Popup/";

    private int _currentOrder = 10; // 현재까지 최근에 사용한 오더
    private readonly Stack<UIPopup> _popupStack = new Stack<UIPopup>();
    private UIHud _hudUI;
    public UIHud HudUI => _hudUI;


    protected override void InitializeManager()
    {
        Debug.Log("UIManager Initialized");
    }

    private GameObject Root
    {
        get
        {
            var root = GameObject.Find("@UI_Root") ?? new GameObject { name = "@UI_Root" };
            return root;
        }
    }

    public void SetCanvas(GameObject go, bool sort = true)
    {
        var canvas = go.GetOrAddComponent<Canvas>();
        canvas.renderMode = RenderMode.ScreenSpaceOverlay;
        canvas.overrideSorting = true;
        canvas.sortingOrder = sort ? _currentOrder++ : 0;
    }

    public T ShowHud<T>() where T : UIHud
    {
        return  ShowHud(typeof(T).Name) as T;
    }

    public UIHud ShowHud(string hudName)
    {
        var prefab = LoadUIResource(HUDPath, hudName);
        if (prefab == null)
            return null;

        return CreateHudInstance(prefab);
    }

    private UIHud CreateHudInstance(GameObject prefab)
    {
        var instance = Instantiate(prefab, Root.transform);
        return EnableUIComponent<UIHud>(instance);
    }

    public T ShowPopup<T>() where T : UIPopup
    {
        return ShowPopup(typeof(T).Name) as T;
    }

    public UIPopup ShowPopup(string popupName)
    {
        var prefab = LoadUIResource(PopupPath, popupName);
        if (prefab == null)
            return null;

        return CreatePopupInstance(prefab);
    }

    private UIPopup CreatePopupInstance(GameObject prefab)
    {
        var instance = Instantiate(prefab, Root.transform);
        return EnableUIComponent<UIPopup>(instance);
    }

    private GameObject LoadUIResource(string path, string resourceName)
    {
        var resource = Resources.Load($"{path}{resourceName}", typeof(GameObject)) as GameObject;
        if (resource == null)
            Debug.LogError($"UI Resource '{resourceName}' not found in path '{path}'");
        return resource;
    }

    private T EnableUIComponent<T>(GameObject obj) where T : UIBase
    {
        var component = obj.GetComponent<T>();
        if (component is UIPopup popup)
            _popupStack.Push(popup);
        else if (component is UIHud hud)
            _hudUI = hud;

        obj.SetActive(true);
        return component;
    }

    public void ClosePopup(UIPopup popup)
    {
        if (_popupStack.Count == 0 || _popupStack.Peek() != popup)
        {
            Debug.LogWarning("Close Popup Failed: Mismatched popup or empty stack.");
            return;
        }

        ClosePopup();
    }

    public void ClosePopup()
    {
        if (_popupStack.Count == 0)
            return;

        var popup = _popupStack.Pop();
        Destroy(popup.gameObject);
        _currentOrder--;
    }

    public void CloseAllPopup()
    {
        while (_popupStack.Count > 0)
            ClosePopup();
    }
}
  ```

</details>

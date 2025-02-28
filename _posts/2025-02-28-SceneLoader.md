---
layout: single
title: "SceneLoader"
categories: [Framework]
tags: [C#, Framework]
typora-root-url: ../
---

### 개요

`SceneLoader` 클래스는 Unity 프로젝트에서 사용되는 씬 전환(씬 관리)과 관련된 유틸리티입니다. 이 클래스는 다음과 같은 주요 기능을 제공합니다:

1. 씬을 비동기적으로 로드하며 로딩 화면 및 진행률 표시 기능 포함.
2. 페이드 인/아웃 효과를 통한 부드러운 화면 전환.
3. 전환 후 로드 완료 시 콜백 지원.
4. 싱글톤 패턴을 통해 전역적으로 접근 가능.

------

### 코드 구성 요소

#### 1. **변수**

```
public static SceneLoader Instance;
```

- 싱글톤 패턴 구현을 위한 정적 변수.
- `SceneLoader`가 프로젝트 전반에서 전역적으로 사용될 수 있도록 보장.

```
[SerializeField] private CanvasGroup fadeCanvasGroup;
[SerializeField] private float fadeDuration = 1f;
[SerializeField] private GameObject loadingScreen;
[SerializeField] private Slider progressBar;
[SerializeField] private TextMeshProUGUI progressText;
```

- **`fadeCanvasGroup`**: 페이드 인/아웃 효과를 위한 `CanvasGroup` (알파 값 조절).
- **`fadeDuration`**: 페이드 효과의 지속 시간.
- **`loadingScreen`**: 로딩 중 활성화될 UI 오브젝트.
- **`progressBar`**: 씬 로딩 진행률을 시각적으로 나타내는 슬라이더.
- **`progressText`**: 텍스트로 로딩 진행률을 퍼센트(% 형식)로 표시.

```
private Action onSceneLoadedCallback;
```

- 씬 로드 완료 시 실행될 콜백 함수.
- 호출 방식에 따라 로드 완료 후 특정 로직을 실행 가능.

------

#### 2. **Awake 메서드**

```
private void Awake()
{
    if (Instance == null)
    {
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
    else
    {
        Destroy(gameObject);
    }
}
```

- 싱글톤 패턴

  :   

  - 인스턴스가 없는 경우(`Instance == null`) 현재 오브젝트를 싱글톤으로 설정.
  - 이미 인스턴스가 존재하면 새로 생성하는 오브젝트를 제거(`Destroy`).

- `DontDestroyOnLoad`

  :   

  - 씬 전환 시에도 파괴되지 않고 유지되는 오브젝트로 설정.

------

#### 3. **LoadScene 메서드들**

##### **`LoadScene(string sceneName, Action onSceneLoaded = null)`**

```
public void LoadScene(string sceneName, Action onSceneLoaded = null)
{
    onSceneLoadedCallback = onSceneLoaded;
    StartCoroutine(LoadSceneAsync(sceneName));
}
```

- 문자열 기반으로 씬 이름(`sceneName`)을 받아 해당 씬을 비동기적으로 로드.
- 로드 완료 후 실행될 콜백(선택 사항)을 설정.
- **코루틴(`StartCoroutine`)**을 사용해 비동기 로드 방식으로 실행.

##### **`LoadScene(int buildIndex, Action onSceneLoaded = null)`**

```
public void LoadScene(int buildIndex, Action onSceneLoaded = null)
{
    onSceneLoadedCallback = onSceneLoaded;
    StartCoroutine(LoadSceneAsync(SceneManager.GetSceneByBuildIndex(buildIndex).name));
}
```

- 빌드 인덱스 기반으로 씬 로드.
- **UnityEngine.SceneManagement.SceneManager**에서 빌드 인덱스를 사용해 씬 이름 추출.

------

#### 4. **LoadSceneAsync 메서드 (비동기 씬 전환)**

```
private IEnumerator LoadSceneAsync(string sceneName)
```

비동기 로드를 수행하며 로딩 화면과 진행률 바를 업데이트하는 핵심 로직입니다.

1. **로딩 UI 활성화**:
   - `loadingScreen.SetActive(true);`: 로딩 화면 활성화.
   - `StartCoroutine(Fade(1f));`: 화면 페이드 아웃(알파 = 1).
2. **비동기 씬 로드**:

```
AsyncOperation operation = SceneManager.LoadSceneAsync(sceneName);
```

1. **로딩 진행률 계산 및 표시**:

```
float progress = Mathf.Clamp01(operation.progress / 0.9f);
progressBar.value = progress;
progressText.text = $"{Mathf.RoundToInt(progress * 100)}%";
```

- 비동기 로드의 **`operation.progress`** 값을 기반으로 진행률 계산.
- `0.9`는 Unity의 특성으로 로딩 값이 항상 최대 `0.9`까지만 전달됨.
- 이에 따라 `progressText`와 `progressBar`에 로딩 퍼센트를 반영.

1. **씬 활성화**:

```
if (operation.progress >= 0.9f)
{
    operation.allowSceneActivation = true;
}
```

- 비동기 로드가 완료되면(`progress >= 0.9f`) 씬 활성화.

1. **페이드 인 및 로드 완료**:

```
yield return StartCoroutine(Fade(0f));
loadingScreen.SetActive(false);
onSceneLoadedCallback?.Invoke();
```

- 페이드 인(알파 = 0) 후 로딩 UI를 비활성화.
- 씬 로드 완료 콜백 호출(`onSceneLoadedCallback`).

------

#### 5. **Fade 메서드 (페이드 효과)**

```
private IEnumerator Fade(float targetAlpha)
```

UI 화면의 **투명도(알파)**를 부드럽게 전환시키는 메서드.

1. 초기 알파 값 설정:

```
float startAlpha = fadeCanvasGroup.alpha;
```

현재 `CanvasGroup`의 알파 값을 시작값으로 저장.

1. 시간 기반 전환:

```
fadeCanvasGroup.alpha = Mathf.Lerp(startAlpha, targetAlpha, elapsedTime / fadeDuration);
```

- `Mathf.Lerp`를 통해 초기 알파 값에서 목표 알파 값(`targetAlpha`)으로 선형 보간.
- 경과 시간(`elapsedTime / fadeDuration`)을 기반으로 점진적 전환 수행.

------

### 클래스 전체 요약 및 활용

#### **특징**

1. **싱글톤 구현**: 싱글톤 패턴으로 Unity 프로젝트 내 전역 관리 도구로 사용.
2. **로딩 진행률 표시**: 퍼센트 텍스트와 슬라이더를 포함한 직관적인 진행률 UI 제공.
3. **페이드 처리**: 자연스러운 화면 전환을 위한 페이드 인/아웃 효과.
4. **비동기 로드 지원**: `SceneManager.LoadSceneAsync`를 활용한 비동기 씬 관리.

#### **사용법**

1. 전역 접근:

```
SceneLoader.Instance.LoadScene("SceneName", OnSceneLoaded);
```

또는

```
SceneLoader.Instance.LoadScene(1, OnSceneLoaded);
```

1. 로드 완료 콜백:

```
void OnSceneLoaded()
{
    Debug.Log("씬 로드 완료!");
}
```

이와 같이, 이 클래스는 플레이어의 게임 플레이 경험을 향상시킬 수 있는 완성도 높은 씬 관리 유틸리티입니다. 🚀

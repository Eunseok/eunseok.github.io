---
layout: single
title: "전략패턴(StrategyPattern)을 사용해 스킬 만들기"
categories: [DesignPattern]
tags: [C#, StrategyPattern, GoF]
typora-root-url: ../
---

## 스킬 전략 다이어그램

![image-20250406003539652](/Images/2025-04-09-StrategyPattern//image-20250406003539652.png)

- Hero는 **전략 인터페이스만** 알고 있음 → 느슨한 결합(Decoupling)
- 각 전략은 **ScriptableObject**로 구현

-0

## 플레이어 및 버튼 세팅

테스트를 위해 화면에 버튼을 구성한다.

![image-20250406003042890](/Images/2025-04-09-StrategyPattern//image-20250406003042890.png)

```csharp
public class HeadsUpDisplay : MonoBehaviour
{
    [SerializeField] Button[] buttons;

    public static event Action<int> OnButtonClicked;

    void Awake()
    {
        for (int i = 0; i < buttons.Length; i++)
        {
            int index = i;
            buttons[i].onClick.AddListener(() => OnButtonClicked?.Invoke(index));
        }
    }
}
```

```csharp
public class Hero : MonoBehaviour
{
    [SerializeField] SpellStrategy[] spells;

    void OnEnable()
    {
        HeadsUpDisplay.OnButtonClicked += CastSpell;
    }

    void OnDisable()
    {
        HeadsUpDisplay.OnButtonClicked -= CastSpell;
    }
    void CastSpell(int index)
    {
        spells[index].CastSpell(transform);
    }
}
```

플레이어는 자신이 스킬을 쓸 수 있다는 것만 알고 그 스킬들이 어떻게 작동되는지는 모른다.

## SpellStrategy

```csharp
public abstract class SpellStrategy : ScriptableObject
{
    public abstract void CastSpell(Transform origin);
}
```

`SpellStrategy`는 모든 스킬 전략들이 상속 받을 추상 클래스이며 `CastSpeel` 단일 메서드만 포함되어 있다.

## 첫 번째 전략: 쉴드**(Shield Strategy)**

![화면 기록 2025-04-06 오전 12.26.10](/Images/2025-04-09-StrategyPattern/화면 기록 2025-04-06 오전 12.26.10.gif)

```csharp
[CreateAssetMenu(menuName = "Spells/ShieldSpellStrategy", fileName = "Shield")]
public class ShieldSpellStrategy : SpellStrategy
{
    public GameObject shieldPrefab;
    public float duration = 10f;

    public override void CastSpell(Transform origin)
    {
        var shield =  Instantiate(shieldPrefab, origin.position, Quaternion.identity, origin);
        Destroy(shield, duration);
    }
}
```

플레이어가 버튼을 누르면 단순히 생성되었다가 지정된 시간에 삭제되는 간단한 스킬이다.

## 두 번째 전략: 발사체 (Projectile Strategy)

![화면 기록 2025-04-06 오전 12.26.10 (1)](/Images/2025-04-09-StrategyPattern/화면 기록 2025-04-06 오전 12.26.10 (1).gif)

```csharp
[CreateAssetMenu(menuName = "Spells/ProjectileSpellStrategy", fileName = "ProjectileSpellStrategy")]
public class ProjectileSpellStrategy : SpellStrategy
{
    public GameObject projectilePrefab;
    public float speed = 10f;
    public float duration = 5f;

    public override void CastSpell(Transform origin)
    {
        new ProjectileBuilder()
            .WithProjectilePrefab(projectilePrefab)
            .WithSpeed(speed)
            .WithDuration(duration)
            .Build(origin);
    }
}
```

```csharp
public class ProjectileBuilder
{
    GameObject projectilePrefab;
    float speed;
    float duration;

    public ProjectileBuilder WithProjectilePrefab(GameObject projectilePrefab)
    {
        this.projectilePrefab = projectilePrefab;
        return this;
    }
    public ProjectileBuilder WithSpeed(float speed)
    {
        this.speed = speed;
        return this;
    }

    public ProjectileBuilder WithDuration(float duration)
    {
        this.duration = duration;
        return this;
    }

    public GameObject Build(Transform origin)
    {
        Vector3 instantiatePosition = origin.position + origin.forward * 2f;

        var projectile = Object.Instantiate(projectilePrefab, instantiatePosition, Quaternion.identity);
        ParticleMover mover = projectile.AddComponent<ParticleMover>();
        SelfDestruct selfDestruct = projectile.AddComponent<SelfDestruct>();

        mover.Initialize(speed);
        selfDestruct.Initialize(duration);

        return projectile;
    }
}
```

Projectile Strategy는 비교적 복잡하여 초기화 과정을 Builer 패턴으로 추상화 하였고 `ParticleMove`,`SelfDestruct`는 각각 파티클 이동과 지정된 시간 뒤에 자폭하는 컨포넌트이다.

## 세 번째 전략: 궤도형 오브(Orbital Strategy)

![화면 기록 2025-04-06 오전 12.26.10 (2)](/Images/2025-04-09-StrategyPattern/화면 기록 2025-04-06 오전 12.26.10 (2).gif)

```csharp
[CreateAssetMenu(menuName = "Spells/OrbitalSpellStrategy", fileName = "OrbitalSpellStrategy")]
public class OrbitalSpellStrategy : SpellStrategy
{
    public GameObject orbPrefab; // 오브 생성에 사용할 프리팹
    public int numberOfOrbs = 3; // 생성할 오브 개수
    public float radius = 3f; // 오브의 궤도 반지름
    public float rotationSpeed = 0.3f; // 오브의 궤도 회전 속도
    public float duration = 5f; // 오브 효과의 지속 시간
    public float verticalMovementRange = 3f; // 오브가 수직으로 이동할 범위

    // 마법을 시전할 때 호출되는 메서드
    public override void CastSpell(Transform origin)
    {
        // 오브들을 관리하기 위한 부모 객체 생성
        Transform orbParent = CreateOrbParent(origin);

        // 부모 객체에 회전 애니메이션 적용
        RotateOrbParent(orbParent);

        // numberOfOrbs만큼 오브를 생성하여 궤도에 배치
        for (int i = 0; i < numberOfOrbs; i++)
        {
            SpawnOrb(origin, orbParent, i);
        }

        // duration만큼 시간이 지난 후 부모 객체와 자식 오브들을 제거
        Destroy(orbParent.gameObject, duration);
    }

    // 오브 생성 및 초기화를 담당
    void SpawnOrb(Transform origin, Transform orbParent, int i)
    {
        // 오브의 초기 위치 계산 및 생성
        var orb = Instantiate(orbPrefab, CalculateOrbitalPosition(origin, i), Quaternion.identity, orbParent);

        // 오브의 수직 이동 애니메이션 추가
        MoveOrbVertically(orb, origin);
    }

    // 오브의 궤도상 위치 계산
    Vector3 CalculateOrbitalPosition(Transform origin, int i)
    {
        // 현재 오브의 궤도에서 각도 계산 (0°~360° 분배)
        float angle = i * 2f * Mathf.PI / numberOfOrbs;

        // 각도에 따라 오브의 위치를 계산 (궤도 반지름을 기준으로)
        return origin.position + new Vector3(Mathf.Cos(angle), 0, Mathf.Sin(angle)) * radius;
    }

    // 오브의 부모 객체에 회전 애니메이션 적용
    void RotateOrbParent(Transform orbParent)
    {
        // 1초에 360도 * rotationSpeed만큼 회전
        float rotationRate = 360f * rotationSpeed;

        // DOTween으로 회전 애니메이션 설정
        DOTween.To(
            () => orbParent.rotation.eulerAngles, // 현재 회전 값
            x => orbParent.rotation = Quaternion.Euler(x), // 타겟 회전 값
            new Vector3(0, rotationRate, 0), // 최종 타겟 (Y축 중심 회전)
            1f // 1초 동안 애니메이션 실행
        )
            .SetLoops(-1, LoopType.Incremental) // 무한 루프 및 지속 증가
            .SetEase(Ease.Linear); // 선형으로 회전
    }

    // 부모 객체 생성 및 초기화
    Transform CreateOrbParent(Transform origin)
    {
        // 부모 객체를 위한 새로운 GameObject 생성
        var orbParent = new GameObject("OrbParent").transform;

        // 부모 객체의 위치 및 회전을 origin의 Transform에 맞춤
        orbParent.position = origin.position;
        orbParent.rotation = origin.rotation;

        // 부모 객체를 origin의 자식으로 설정
        orbParent.SetParent(origin);

        return orbParent;
    }

    // 오브에 수직 이동 애니메이션 적용
    void MoveOrbVertically(GameObject orb, Transform origin)
    {
        // 수직 이동에 소요될 무작위 시간 계산 (0.2초 ~ 0.5초)
        float verticalMovementDuration = Random.Range(0.2f, 0.5f);

        // 수직 이동 애니메이션 설정 (Y축으로 verticalMovementRange만큼 왕복 이동)
        orb.transform.DOMoveY(orb.transform.position.y + verticalMovementRange, verticalMovementDuration)
            .SetLoops(-1, LoopType.Yoyo) // 왕복 루프 설정
            .OnUpdate(() =>
                      {
                          // 애니메이션 중 오브가 origin을 바라보게 회전 설정
                          orb.transform.rotation = Quaternion.LookRotation(
                              Vector3.Reflect(orb.transform.position - origin.position, Vector3.up)
                          );
                      });
    }
}
```

비교적 궤도형 오브스킬은 DOTween으로 애니메이션 되어 있으며 비교적 복잡하므로 각 주요 메서드들을 더 알아보자

---



```csharp
Vector3 CalculateOrbitalPosition(Transform origin, int i)
{
    // 현재 오브의 궤도에서 각도 계산 (0°~360° 분배)
    float angle = i * 2f * Mathf.PI / numberOfOrbs;

    // 각도에 따라 오브의 위치를 계산 (궤도 반지름을 기준으로)
    return origin.position + new Vector3(Mathf.Cos(angle), 0, Mathf.Sin(angle)) * radius;
}
```

`CalculateOrbitalPosition` 메서드는 오브젝트를 특정 궤도에 균일하게 배치하기 위해 사용된다. 선택된 코드에서 수행되는 작업을 요약하면 다음과 같다:

### 설명

1. **각도 계산**:
   - `float angle = i * 2f * Mathf.PI / numberOfOrbs;`
   - `i`는 현재 오브의 인덱스
   - `numberOfOrbs`로 나누어 각 오브가 궤도를 따라 일정한 간격(360°/`numberOfOrbs`)으로 배치되도록 각도를 계산한다.
   - `2f * Mathf.PI`는 360°(라디안 값)를 나타낸다.
2. **궤도 위치 계산**:
   - `return origin.position + new Vector3(Mathf.Cos(angle), 0, Mathf.Sin(angle)) * radius;`
   - `Mathf.Cos(angle)`과 `Mathf.Sin(angle)`은 계산한 각도에 따라 X축(`Cos`)과 Z축(`Sin`)상의 위치를 구한다.
   - 결과에 `radius`를 곱해 오브들이 궤도 중심에서 `radius`만큼 떨어진 위치에 놓이도록 한다.
   - 최종적으로 궤도의 중심이 되는 `origin.position`에 더해 실제 월드 좌표를 계산한다.

### 주요 역할

- 이 메서드는 궤도를 따라 고르게 분산된 오브의 위치를 결정
- 반환된 `Vector3` 값은 오브가 배치될 정확한 월드 좌표를 나타낸다.

### 사용하는 컨텍스트

- `CastSpell` 메서드에서 각 오브를 생성할 때 호출되며, `SpawnOrb` 메서드를 통해 개별 오브가 배치됨.

---



```csharp
void RotateOrbParent(Transform orbParent)
{
    // 1초에 360도 * rotationSpeed만큼 회전
    float rotationRate = 360f * rotationSpeed;

    // DOTween으로 회전 애니메이션 설정
    DOTween.To(
        () => orbParent.rotation.eulerAngles, // 현재 회전 값
        x => orbParent.rotation = Quaternion.Euler(x), // 타겟 회전 값
        new Vector3(0, rotationRate, 0), // 최종 타겟 (Y축 중심 회전)
        1f // 1초 동안 애니메이션 실행
    )
        .SetLoops(-1, LoopType.Incremental) // 무한 루프 및 지속 증가
        .SetEase(Ease.Linear); // 선형으로 회전
}
```

`RotateOrbParent` 메서드는 주어진 `Transform` 객체(즉, `orbParent`)에 지속적인 회전 애니메이션을 적용하기 위해 사용된다.

------

### 상세 설명

1. **회전 속도 계산**:

```csharp
float rotationRate = 360f * rotationSpeed;
```

- `rotationSpeed`: 초당 회전 속도를 결정하는 값.
- `rotationRate`: 초당 회전 각도(360°의 비율).

1. **애니메이션 생성**:

```csharp
DOTween.To(
    () => orbParent.rotation.eulerAngles, // 현재의 회전 값
    x => orbParent.rotation = Quaternion.Euler(x), // 변경된 회전 값을 반영
    new Vector3(0, rotationRate, 0), // 대상 회전 값 (Y축 회전)
    1f // 애니메이션 지속 시간 (1초)
)
```

- `DOTween.To` 메서드는 애니메이션을 생성
- **현재 값**: `orbParent.rotation.eulerAngles`
- **목표 값**: Y축을 기준으로 계산된 `rotationRate`만큼 회전
- **지속 시간**: 1초 동안 대상 값에 도달하도록 설정.

1. **반복 동작 설정**:

```csharp
.SetLoops(-1, LoopType.Incremental)
```

- `-1`은 **무한 루프**를 의미
- `LoopType.Incremental`은 기존 회전 값에 지속적으로 증가 값을 더해 회전을 계속 이어간다.

1. **회전 곡선 설정**:

```csharp
.SetEase(Ease.Linear);
```

- `Ease.Linear`를 사용하여 일정한 속도로 부드럽게 회전하도록 설정.

------

### 역할

- 이 메서드는 `orbParent` 객체(오브들이 속한 부모 객체)를 지속적으로 회전시켜, 자식인 오브들이 중심을 기준으로 궤도를 따라 원형으로 회전하는 효과를 구현한다.

------

```csharp
void MoveOrbVertically(GameObject orb, Transform origin)
{
    // 수직 이동에 소요될 무작위 시간 계산 (0.2초 ~ 0.5초)
    float verticalMovementDuration = Random.Range(0.2f, 0.5f);

    // 수직 이동 애니메이션 설정 (Y축으로 verticalMovementRange만큼 왕복 이동)
    orb.transform.DOMoveY(orb.transform.position.y + verticalMovementRange, verticalMovementDuration)
        .SetLoops(-1, LoopType.Yoyo) // 왕복 루프 설정
        .OnUpdate(() =>
                  {
                      // 애니메이션 중 오브가 origin을 바라보게 회전 설정
                      orb.transform.rotation = Quaternion.LookRotation(
                          Vector3.Reflect(orb.transform.position - origin.position, Vector3.up)
                      );
                  });
}
```

`MoveOrbVertically` 메서드는 특정 오브젝트(`orb`)가 Y축(수직 방향)으로 왕복 운동을 수행하도록 애니메이션을 설정한다. 또한 애니메이션 동안 해당 오브젝트가 중심(`origin`)을 향해 회전하게끔 구현

------

### 상세 설명

**수직 이동 애니메이션**:

```csharp
orb.transform.DOMoveY(orb.transform.position.y + verticalMovementRange, verticalMovementDuration)
```

- `orb.transform.position.y + verticalMovementRange`:  현재 오브의 Y축 위치에서 `verticalMovementRange`만큼 위로 이동하도록 설정.
- `DOMoveY` 메서드는 오브의 Y축 위치를 목표값으로 이동.
- 랜덤으로 정해진 `verticalMovementDuration` 동안 애니메이션이 실행.

**왕복 루프 설정**:

```csharp
.SetLoops(-1, LoopType.Yoyo)
```

- `-1`: 무한 반복을 의미.
- `LoopType.Yoyo`: 목표 위치로 이동한 후, 다시 원래 위치로 돌아오도록 왕복.
- 이 설정으로 오브는 계속 위-아래로 반복적인 움직임을 수행.

1. **오브의 회전 업데이트**:

```csharp
.OnUpdate(() =>
{
    orb.transform.rotation = Quaternion.LookRotation(
        Vector3.Reflect(orb.transform.position - origin.position, Vector3.up)
    );
});
```

- `OnUpdate`: 애니메이션 실행 중 매 프레임마다 호출되는 콜백 함수.
- `Quaternion.LookRotation`: 오브가 중심의 위치(`origin.positioncsharp`)를 지속적으로 바라보도록 설정.   
- 중심에서 오브의 위치를 계산한 벡터(`orb.transform.position - origin.position`)를 뒤집기 위해 `Vector3.Reflect`를 사용.
- 이로 인해 수직 이동 중에도 오브는 중심을 향한 자연스러운 방향을 유지.

------

### 역할

- 오브들이 궤도상에서 수직 방향으로 왕복하며 중심을 향해 회전하는 애니메이션을 구현.

---
layout: single
title: "탑다운 슈팅게임 제작"
categories: []
tags: [C#]
typora-root-url: ../

---

## 프로젝트 생성 및 기초 설정

#### :one: 프로젝트 생성

Unity Hub실행 > Projects 탭에서 New Project 버튼 클릭 >

2D 템플릿 > 프로젝트이름 TopDown_Project(추천) > Create 버튼을 눌러 프로젝트 생성



#### :two: 기본 환경 설정

1. 2D 환경 설정 확인
   - 상단 ToolBar에서 2D 버튼 활성화 확인
     - 2D 모드에서 작업이 용이해집니다.
2. 카메라 설정
   - Main Camera 선택 후 Inspector에서 아래 설정 확인
     - Projection: Othograpjic(원근감 없는 카메라)
     - Size: 6 (필요에 따라 조정 가능)
     - Background Color: 게임 배경에 맞는 색상으로 설정
3. 게임 뷰 설정
   - Aspect Ratio: 16:9

---

## 스프라이트 추가

#### :one: 스프라이트 & 소리파일 준비

- 프로젝트에 사용할 에셋을 다운로드 또는 준비

#### :two: 스프라이트 임포트

- 준비된 스프라이트 파일을 Assets/Externals(`외부`) 폴더에 드래그 앤 드롭

- TillSet_Sprite -> frames 전체 선택
- Inspector에서 아래 설정 확인 및 수정
  - Pixels per Unit: 16(16X16 이미지라서)
  - Filter Mode: Ponit (no filter)

- Apply 클릭

---

## 플레이어 오브젝트 추가

#### :one: 플레이어 오브젝트 생성

- Hierachy에서 새 오브젝트 생성 : 우클릭 -> Create Empty
  - 이름: "Player"
  - Add Component: "Rigidbody 2D"
    - Gravity Scale: 0 (중력 X)
    - Constraints -> Freeze Rotation -> Z: True (회전 물리적용 X)
  - Add Component: "Box Collider 2D"

#### :two: 플레이어 하위 오브젝트 생성

- Hierachy에서 Player 오브젝트 우클릭 ​-> Create Empty
  - 이름: "MainSprite"
  - Add Componet: "Sprite Renderer"
    - Sprite: 캐릭터 스프라이트
    - Additional Settings -> Order in Layer: 100
  - Hierachy에서 Player 오브젝트 우클릭 -> Create Empty
    - 이름: "WeaponPivot"

---

## 플레이어 이동 구현

#### :one: BaseController 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성 `BaseController.cs`

- BaseController 스크립트 작성

  ```csharp
  using UnityEngine;
  
  public class BaseController : MonoBehaviour
  {
    protected Rigidbody2D _rigidbody; //코드에서 자동 할당
  
    // 자식 오브젝트에 위치하고 여러개 있을 수 있어 Inspector에서 설정
    [SerializeField] private SpriteRenderer characterRenderer;
    [SerializeField] private Transform weaponPivot; 
  
    // 상속받은 클래스에서도 수정할 수 있도록 protected 사용
    protected Vector2 movementDirection = Vector2.zero; // 이동 방향 벡터
    // 외부에서는 읽기만 가능하게 하고, 내부에서만 수정할 수 있도록 백킹 필드 사용  
    public Vector2 MovementDirection { get { return movementDirection; } }
  
    protected Vector2 lookDirection = Vector2.zero; // 바라보는 방향 벡터
    public Vector2 LookDirection { get { return lookDirection; } }
  
  
    private Vector2 knockback = Vector2.zero; // 넉백 벡터
    private float knockbackDuration = 0.0f; // 넉백 지속 시간
  
    protected virtual void Awake()
    {
      _rigidbody = GetComponent<Rigidbody2D>();
    }
  
    protected virtual void Start()
    {
    }
  
    protected virtual void Update()
    {
      HandleAction(); // 입력 또는 행동 처리
      Rotate(lookDirection); // 캐릭터 회전 처리
    }
  
    protected virtual void FixedUpdate()
    {
      Movment(movementDirection); // 물리 이동 처리
  
      if (knockbackDuration > 0.0f)
      {
        knockbackDuration -= Time.fixedDeltaTime; // 넉백 시간 감소
      }
    }
  
    protected virtual void HandleAction()
    {
      // 하위 클래스에서 동작을 정의할 수 있도록 가상 함수로 선언
    }
  
    private void Movment(Vector2 direction)
    {
      direction = direction * 5; // 5는 추후 추가될 StatHandler.cs에 의해 변경
  
      if (knockbackDuration > 0.0f)
      {
        direction *= 0.2f; // 넉백 중에는 이동 속도 감소
        direction += knockback; // 넉백 힘 추가 적용
      }
  
      _rigidbody.velocity = direction; // Rigidbody2D에 속도 적용
    }
  
    private void Rotate(Vector2 direction)
    {
      float rotZ = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg; // 방향을 회전 각도로 변환
      bool isLeft = Mathf.Abs(rotZ) > 90f; // 왼쪽 방향 여부 체크
  
      characterRenderer.flipX = isLeft; // 캐릭터 좌우 반전 적용
  
      if (weaponPivot != null)
      {
        weaponPivot.rotation = Quaternion.Euler(0, 0, rotZ); // 무기 회전 적용
      }
    }
  
    public void ApplyKnockback(Transform other, float power, float duration)
    {
      knockbackDuration = duration; // 넉백 지속 시간 설정
      knockback = -(other.position - transform.position).normalized * power; // 충돌 방향을 기준으로 넉백 힘 계산
    }    
  }
  
  ```

#### :four: PlayerController 스크립트 생성

- Assets/Scripts/Entity 폴더에 새 스크립트 생성 `PlayerController.cs`

- Player 오브젝트에 스크립트 드래그하여 연결

- PlayerController 스크립트 작성

  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class PlayerController : BaseController
  {
      private Camera camera;
  
      protected override void Start()
      {
          base.Start();
          camera = Camera.main;
      }    
  
      protected override void HandleAction()
      {
          float horizontal = Input.GetAxisRaw("Horizontal");
          float vertical = Input.GetAxisRaw("Vertical");
          movementDirection = new Vector2(horizontal, vertical).normalized;
  
          Vector2 mousePosition = Input.mousePosition;
          Vector2 worldPos = camera.ScreenToWorldPoint(mousePosition);
          lookDirection = (worldPos - (Vector2)transform.position);
  
          if (lookDirection.magnitude < .9f)
          {
              lookDirection = Vector2.zero;
          }
          else
          {
              lookDirection = lookDirection.normalized;
          }
      }
  }
  ```

  

---

## 타일맵 추가

:one: 맵 루트 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "Level"

:two: Hierarchy에서 Level 오브젝트 생성: 우클릭 Create -> 2D Object -> Tilemap -> Rectangular

- 이름: "Floor"
- Additional Setting -> Order in Layer: 0

:three: Hierarchy에서 Level 오브젝트 생성: 우클릭 Create -> 2D Object -> Tilemap -> Rectangular

- 이름: “BackDesign”
- Additional Setting -> Order in Layer: 10

:four: Hierarchy에서 Level 오브젝트 생성: 우클릭 Create -> 2D Object -> Tilemap -> Rectangular

- 이름: “ForeDesign”
- Additional Setting -> Order in Layer: 200

:five: Hierarchy에서 Level 오브젝트 생성: 우클릭 Create -> 2D Object -> Tilemap -> Rectangular

- 이름: “Collision”
- Additional Setting -> Order in Layer: 0

---

## 타일 그리기

:one: 상단 메뉴에서 Window -> 2D -> Tile Palete 클릭

:two: Create New Palette 클릭

- 이름: "Dungeon"
- ArtWork\Level 폴더에 저장

:three: Sprite 드래그 하여 추가

- Artwork\Level 폴더에 저장

:four: 바닥 그리기

- Floor 오브젝트 클릭
- Floor Tile 선택
-  그리기

:five: 원경 그리기

- BackDesign 오브젝트 클릭
- 그리기

:six: 전경 그리기

- ForeDesign 오브젝트 클릭
- 그리기

:seven: 충돌체 그리기

- collision 오브젝트 클릭
- Add Componet: "Tilemap Collider 2D"
- Tilemap -> Color -> A 0
- 그리기

---

## 애니메이션 생성

:one: 상단메뉴에서 Window -> Animation -> Animation 클릭

- Player -> MainSprite 클릭

:two: Idle 애니메이션 생성

- Create 클릭

- Animations/Player 폴더에 Idle.anim 으로 저장

- Sample: 12

  > Unity의 **Animation 창(Animation Window)** 에서 “Sample”은 **애니메이션의 샘플링 속도(FPS, Frame Per Second)** 를 의미합니다.

- Idle 스프라이트 추가

:three: Move 애니메이션 생성

- Create New Clip 클릭
- Animations/Player 폴더에 Move.anim으로 저장
- Sample:12
- Move  스프라이트 추가

:four: Damage 애니메이션 생성

- Create New Clip 클릭
- Animations/Player 폴더에 Move.anim으로 저장
- Sample:12
- Add property -> Sprite Renderer -> Color 추가(히트시 빨간색)
- Damage  스프라이트 추가

---

## 애니메이션 설정

:one: 다음과 같이 Transition을 연결

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fd50944fe-7d21-450b-84b1-9b90124ba95a%2Fimage.png?table=block&id=17f2dc3e-f514-81a3-94d1-ff1d90e973ac&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=850&userId=&cache=v2)

- Has Exit Time: False

- Transition Duration, Transition Offset: 0

  | 설정                   | 의미                                 | 결과                          |
  | ---------------------- | ------------------------------------ | ----------------------------- |
  | Has Exit Time: False   | 애니메이션이 끝나지 않아도 전환 가능 | 즉시 다음 애니메이션으로 전환 |
  | Transition Duration: 0 | 애니메이션 블렌딩 없이 즉시 전환     | 부드러운 전환 없이 바로 변경  |
  | Transition Offset: 0   | 애니메이션이 처음부터 재생됨         | 항상 처음부터 시작            |

:two: Parameter 추가

- IsMove (bool)
- IsDamage(bool)

:three: Transition 설정

- Idle -> Damage
  - Condition
    - IsDamage: True
- Damage -> Idle
  - Condition
    - IsDamage: False
- Idle -> Move
  - Condition
    - IsMove: True
- Move -> Idle
  - Condition
    - IsMove: False
- Move -> Damage
  - Condition
    - IsDamage: True

---

## 애니메이션 코드 구현

:one: AnimationHandler 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성 `AnimationHandler.cs`
- Player 오브젝트에 스크릅트를 드래그하여 연결

:two: AnimationHandler 스크립트 작성

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AnimationHandler : MonoBehaviour
{
  //애니메이터 파라미터 캐싱
    private static readonly int IsMoving = Animator.StringToHash("IsMove");
    private static readonly int IsDamage = Animator.StringToHash("IsDamage");

    protected Animator animator;

    protected virtual void Awake()
    {
        animator = GetComponentInChildren<Animator>();
    }

    public void Move(Vector2 obj)
    {
        animator.SetBool(IsMoving, obj.magnitude > .5f);
    }

    public void Damage()
    {
        animator.SetBool(IsDamage, true);
    }
		
  	//Invincibility: 무적
    public void InvincibilityEnd()
    {
        animator.SetBool(IsDamage, false);
    }
}
```

:three: BaseController 스크립트 수정

```csharp
//...(기존코드)

protected AnimationHandler animationHandler;

protected virtual void Awake()
{
    _rigidbody = GetComponent<Rigidbody2D>();
    animationHandler = GetComponent<AnimationHandler>(); //추가
}

//...(기존코드)

private void Movment(Vector2 direction)
{
    direction = direction * 5;
    if(knockbackDuration > 0.0f)
    {
        direction *= 0.2f;
        direction += knockback;
    }
    
    _rigidbody.velocity = direction;
    animationHandler.Move(direction); //추가
}
```

---

## 스탯 시스템 기초구현

:one: StatHandler 스크립트 생성

- Assets/Scripts/Entity 폴더에 새 스크립트 생성: StatHandler.cs
- Player 오브젝트에 스크립트를 드래그하여 연결

:two: StatHandler 스크립트 작성

```csharp
using UnityEngine;

public class StatHandler : MonoBehaviour
{
  	//현재는 체력과 이동속도만 필요하므로 필요시 추가
    [Range(1,100)] [SerializeField] private int health = 10;
    public int Health{
        get => health;
        set => health = Mathf.Clamp(value,0,100);
    } 
    
    [Range(1f, 20f)] [SerializeField] private float speed = 3;
    public float Speed{
        get => speed;
        set => speed = Mathf.Clamp(value,0,20);
    }
}
```

:three: BaseController 스크립트 수정

```csharp
...(기존코드)

protected StatHandler statHandler;

protected virtual void Awake()
{
    _rigidbody = GetComponent<Rigidbody2D>();
    animationHandler = GetComponent<AnimationHandler>();
    statHandler = GetComponent<StatHandler>(); //추가
}

...(기존코드)

private void Movment(Vector2 direction)
{
    direction = direction * statHandler.Speed; //수정
    if(knockbackDuration > 0.0f)
    {
        direction *= 0.2f;
        direction += knockback;
    }
    
    _rigidbody.velocity = direction;
    animationHandler.Move(direction);
}
```

---

## 캐릭터 자원 구현

:one: ResourceContorller 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성: ResourceController.cs
- Player 오브젝트에 스크립트를 드래그하여 연결

:two: ResourceController 스크립트 작성

```csharp
using UnityEngine;

public class ResourceController : MonoBehaviour
{
    [SerializeField] private float healthChangeDelay = .5f;

    private BaseController baseController;
    private StatHandler statHandler;
    private AnimationHandler animationHandler;
    
    private float timeSinceLastChange = float.MaxValue; // 체력변화가 있고난 다음 시간(초과 상태로 설정)

  	//StatHandler의 Health대신 자원으로 사용할 CurrentHealth
    public float CurrentHealth { get; private set; }
    public float MaxHealth => statHandler.Health;

    private void Awake()
    {
        statHandler = GetComponent<StatHandler>(); //스탯 정보 가져오기
        animationHandler = GetComponent<AnimationHandler>(); //자원 변화에 따른 애니메이션 재생
        baseController = GetComponent<BaseController>(); // 넉백이나 Death시에 조작 금지하기
    }

   
    private void Start()
    {
        CurrentHealth = statHandler.Health; 	//최대 체력으로 초기화
    }

  	//무적 시간 처리
    private void Update()
    {
        if (timeSinceLastChange < healthChangeDelay)
        {
            timeSinceLastChange += Time.deltaTime;
            if (timeSinceLastChange >= healthChangeDelay)
            {
                animationHandler.InvincibilityEnd();
            }
        }
    }

    public bool ChangeHealth(float change)
    {
        if (change == 0 || timeSinceLastChange < healthChangeDelay)
        {
            return false; // 체력 변화가 없거나 무적 시간 중이면 변경 불가
        }

        timeSinceLastChange = 0f; // 무적 시간 초기화
        CurrentHealth += change; // 체력 변경
        CurrentHealth = CurrentHealth > MaxHealth ? MaxHealth : CurrentHealth;
        CurrentHealth = CurrentHealth < 0 ? 0 : CurrentHealth;

        if (change < 0)
        {
            animationHandler.Damage(); //변화 값이 음수면 피격 애니메이션 실행
        }

        if (CurrentHealth <= 0f)
        {
            Death(); // 체력이 0이 되면 사망 처리
        }

        return true; // 체력 변경 성공
    }

    private void Death()
    {

    }

}
```

---

## 원거리 공격 구현

:one:  무기 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create  Empty
  - 이름: "P_Bow_EquipWeapon"

:two: 무기 하위 오브젝트 생성

- Hierarchy에서 P_Bow_EquipWeapon 오브젝트 우클릭 -> Create Empty
  - 이름: "WeaponSprite"
  - Position: (0.3, 0, 0)
  - Add Component: "Sprite Renderer"
    - Sprite: 무기 스프라이트 추가
    - Additional Settings -> Order in Layer:120
- Hierarchy에서 P_Bow_EquipWeapon 오브젝트 우클릭 -> Create Empty
  - 이름: "BulletSpawnPoint"
  - Position: (0.5, 0, 0)

---

## 무기 애니메이션 생성

:one: 상단메뉴에서 Window -> Animation -> Animation 클릭

- P_Bow_EquipWeapon -> WeaponSprite 클릭

:two: Idle 애니메이션 생성

- Create 클릭
- Animations/Weapon 폴더에 Bow_Idle.anim 으로 저장
- Sample: 8
- 자유롭게 만들되 0:8에 끝맺음

:three: Attack 애니메이션 생성

- Create New Clip 클릭
- Animations/Weapon 폴더에 Bow_Attack.anim 으로 저장
- Sample: 8
- 자유롭게 만들되 0:8에 끝맺음

---

## 애니메이터 설정

:one: 다음과 같이 Transition을 연결

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F42381a01-91db-4974-bbb0-c7246586f1ec%2Fimage.png?table=block&id=17f2dc3e-f514-81f1-8b77-fda5a4e0d5c7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=670&userId=&cache=v2)

:two: Parameter 추가

- IsAttack (Trigger)

:three: Transition 설정

- Idle -> Attack
  - Has Exit Time: false
  - Transition Duration, Transition Offset: 0
- Attack -> Idle
  - Has Exit Time: True
  - Transition Duration, Transition Offset: 0

---

## 무기 동작 구현

:one: WeaponHandler 스크립트 생성

- Assets/Scripts/Weapon 폴더에서 새 스크립트 생성: `WeaponHander.cs`

:two: WeaponHandler 스크립트 작성

```csharp
using UnityEngine;

public class WeaponHandler : MonoBehaviour
{
    [Header("Attack Info")]
    [SerializeField] private float delay = 1f; // 공격 딜레이
    public float Delay { get => delay; set => delay = value; }
    
    [SerializeField] private float weaponSize = 1f; // 무기 크기
    public float WeaponSize { get => weaponSize; set => weaponSize = value; }
    
    [SerializeField] private float power = 1f; // 공격력
    public float Power { get => power; set => power = value; }
    
    [SerializeField] private float speed = 1f; // 공격 속도
    public float Speed { get => speed; set => speed = value; }

    [SerializeField] private float attackRange = 10f; // 공격 범위
    public float AttackRange { get => attackRange; set => attackRange =value; }

    public LayerMask target; // 공격 대상 레이어

    [Header("Knock Back Info")]
    [SerializeField] private bool isOnKnockback = false; // 넉백 여부
    public bool IsOnKnockback { get => isOnKnockback; set => isOnKnockback = value; }
    
    [SerializeField] private float knockbackPower = 0.1f; // 넉백 파워
    public float KnockbackPower { get => knockbackPower; set => knockbackPower = value; }
    
    [SerializeField] private float knockbackTime = 0.5f; // 넉백 지속 시간
    public float KnockbackTime { get => knockbackTime; set => knockbackTime = value; }
    
    private static readonly int IsAttack = Animator.StringToHash("IsAttack"); // 공격 애니메이션 트리거
    
    public BaseController Controller { get; private set; } // 무기를 가진 캐릭터의 컨트롤러
    
    private Animator animator;
    private SpriteRenderer weaponRenderer;

    protected virtual void Awake()
    {
        Controller = GetComponentInParent<BaseController>(); // 부모 오브젝트의 BaseController 가져오기
        animator = GetComponentInChildren<Animator>(); // 자식 오브젝트의 Animator 가져오기
        weaponRenderer = GetComponentInChildren<SpriteRenderer>(); // 자식 오브젝트의 SpriteRenderer 가져오기
        
        animator.speed = 1.0f / delay; // 공격 딜레이에 따라 애니메이션 속도 조정
        transform.localScale = Vector3.one * weaponSize; // 무기 크기 조정
    }

    protected virtual void Start()
    {
        
    }
    
    public virtual void Attack()
    {
        AttackAnimation(); // 공격 실행
    }

    public void AttackAnimation()
    {
        animator.SetTrigger(IsAttack); // 공격 애니메이션 실행
    }

    public virtual void Rotate(bool isLeft)
    {
        weaponRenderer.flipY = isLeft; // 좌우 방향에 따라 무기 반전
    }
}

```



:three: RangeWeaponHandler 스크립트 생성

- Assets/Scripts/Weapon 폴더에서 새 스크립트 생성: RangeWeaponHandler.cs
- P_Bow_EquipWeapon 오브젝트에 스크립트를 드래그하여 연결

:four: RangeWeaponHandler 스크립트 작성

```csharp
using UnityEngine;
using Random = UnityEngine.Random;

public class RangeWeaponHandler : WeaponHandler
{

    [Header("Ranged Attack Data")]
    [SerializeField] private Transform projectileSpawnPosition; // 발사체 생성 위치
    
    [SerializeField] private int bulletIndex; // 발사체 인덱스
    public int BulletIndex { get { return bulletIndex; } }
    
    [SerializeField] private float bulletSize = 1; // 발사체 크기
    public float BulletSize { get { return bulletSize; } }
    
    [SerializeField] private float duration; // 발사체 지속 시간
    public float Duration { get { return duration; } }
    
    [SerializeField] private float spread; // 발사체 퍼짐 정도
    public float Spread { get { return spread; } }
    
    [SerializeField] private int numberofProjectilesPerShot; // 한 번에 발사되는 투사체 개수
    public int NumberofProjectilesPerShot { get { return numberofProjectilesPerShot; } }
    
    [SerializeField] private float multipleProjectilesAngel; // 투사체 간 각도
    public float MultipleProjectilesAngel { get { return multipleProjectilesAngel; } }
    
    [SerializeField] private Color projectileColor; // 발사체 색상
    public Color ProjectileColor { get { return projectileColor; } }

    public override void Attack()
    {
        base.Attack(); // 기본 공격 실행
        
        float projectilesAngleSpace = multipleProjectilesAngel;
        int numberOfProjectilesPerShot = numberofProjectilesPerShot;

        float minAngle = -(numberOfProjectilesPerShot / 2f) * projectilesAngleSpace; // 중앙을 기준으로 각도 설정

        for (int i = 0; i < numberOfProjectilesPerShot; i++)
        {
            float angle = minAngle + projectilesAngleSpace * i; // 각도 조정
            float randomSpread = Random.Range(-spread, spread); // 랜덤한 퍼짐 효과 추가
            angle += randomSpread;
            CreateProjectile(Controller.LookDirection, angle); // 발사체 생성
        }
    }
    
    private void CreateProjectile(Vector2 _lookDirection, float angle)
    {
        // 발사체 생성 로직 구현 예정
    }
    
    private static Vector2 RotateVector2(Vector2 v, float degree)
    {
        return Quaternion.Euler(0, 0, degree) * v; // 벡터를 주어진 각도로 회전
    }
}

```



:five: RangeWeaponHandler 연결 및 설정

	- RangeWeaponHandler Inspecter에서 ProjectileSpawnPosition: `BulletSpawnPoint`
	- Prefabs/Weapon 폴더에 프리팹 저장
	- Scene에서 삭제

---

## 무기 생성 코드 구현

:one: BaseController 스크립트 수정

```csharp
[SerializeField] public WeaponHandler WeaponPrefab; // 무기 프리팹
protected WeaponHandler weaponHandler; // 현재 장착된 무기

protected bool isAttacking; // 공격 중인지 여부
private float timeSinceLastAttack = float.MaxValue; // 마지막 공격 후 경과 시간

protected virtual void Awake()
{
    _rigidbody = GetComponent<Rigidbody2D>(); // Rigidbody2D 가져오기
    animationHandler = GetComponent<AnimationHandler>(); // 애니메이션 핸들러 가져오기
    statHandler = GetComponent<StatHandler>(); // 스탯 핸들러 가져오기
    
    if(WeaponPrefab != null)
        weaponHandler = Instantiate(WeaponPrefab, weaponPivot); // 무기 프리팹 인스턴스화
    else
        weaponHandler = GetComponentInChildren<WeaponHandler>(); // 자식 오브젝트에서 무기 찾기
}

protected virtual void Update()
{
    HandleAction(); // 플레이어 입력 처리
    Rotate(lookDirection); // 캐릭터 회전 처리
    HandleAttackDelay(); // 공격 딜레이 처리
}

private void Rotate(Vector2 direction)
{
    float rotZ = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg; // 방향을 각도로 변환
    bool isLeft = Mathf.Abs(rotZ) > 90f; // 좌우 반전 여부 확인
    
    characterRenderer.flipX = isLeft; // 캐릭터 방향 반전 적용
    
    if (weaponPivot != null)
    {
        weaponPivot.rotation = Quaternion.Euler(0, 0, rotZ); // 무기 방향 설정
    }
    
    weaponHandler?.Rotate(isLeft); // 무기 반전 적용
}

private void HandleAttackDelay()
{
    if (weaponHandler == null)
        return;

    if(timeSinceLastAttack <= weaponHandler.Delay)
    {
        timeSinceLastAttack += Time.deltaTime; // 공격 딜레이 증가
    }
    
    if(isAttacking && timeSinceLastAttack > weaponHandler.Delay)
    {
        timeSinceLastAttack = 0; // 공격 타이머 초기화
        Attack(); // 공격 실행
    }
}
    
protected virtual void Attack()
{
    if(lookDirection != Vector2.zero)
        weaponHandler?.Attack(); // 무기 공격 실행
}

```

:two: PlayerController 스크립트 수정

```csharp
protected override void HandleAction()
{
		(기존코드)
    
    isAttacking = Input.GetMouseButton(0);
}
```

:three: 무기 장착

- Player  Controller/WeaponPrefab: `P_Bow_EquipWeapon` 프리팹 연결

---

## 투사체 스크립트 준비

:one: 투사체 매니저 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: `ProjectileManager`

:two: ProjectileManager 스크립트 생성

- Assets/Scripts/Manager 폴더에서 새 스크립트 생성: ProjectileManager.cs
- ProjectileManger 오브젝트에 스크립트를 드래그하여 연결

:three: ProjectileManager 스크립트 작성

```csharp
using UnityEngine;

public class ProjectileManager : MonoBehaviour
{
    private static ProjectileManager instance; // 싱글톤 인스턴스
    public static ProjectileManager Instance { get { return instance; } } // 싱글톤 접근자

    [SerializeField] private GameObject[] projectilePrefabs; // 발사체 프리팹 배열

    private void Awake()
    {
        instance = this; // 싱글톤 인스턴스 설정
    }
    
    public void ShootBullet(RangeWeaponHandler rangeWeaponHandler, Vector2 startPostiion, Vector2 direction)
    {
        GameObject origin = projectilePrefabs[rangeWeaponHandler.BulletIndex]; // 해당 무기의 발사체 프리팹 가져오기
        GameObject obj = Instantiate(origin, startPostiion, Quaternion.identity); // 발사체 생성
        
        ProjectileController projectileController = obj.GetComponent<ProjectileController>(); // 발사체 컨트롤러 가져오기
        projectileController.Init(direction, rangeWeaponHandler); // 발사체 초기화
    }
}

```

:four: ProjectileController 스크립트 생성

- Assets/Scripts/Weapon 폴더에서 새 스크립트 생성: ProjectileController.cs

:five: ProjectileController 스크립트 작성

```csharp
using UnityEngine;

public class ProjectileController : MonoBehaviour
{
    [SerializeField] private LayerMask levelCollisionLayer; // 충돌 감지 레이어

    private RangeWeaponHandler rangeWeaponHandler; // 무기 핸들러 참조
    
    private float currentDuration; // 발사체 유지 시간
    private Vector2 direction; // 이동 방향
    private bool isReady; // 발사 가능 여부
    private Transform pivot; // 발사체 피벗
    
    private Rigidbody2D _rigidbody;
    private SpriteRenderer spriteRenderer;

    public bool fxOnDestory = true; // 파괴 시 FX 생성 여부
    
    private void Awake()
    {
        spriteRenderer = GetComponentInChildren<SpriteRenderer>(); // 발사체 스프라이트 가져오기
        _rigidbody = GetComponent<Rigidbody2D>(); // Rigidbody2D 가져오기
        pivot = transform.GetChild(0); // Pivot 가져오기
    }

    private void Update()
    {
        if (!isReady) 
        { 
            return; // 준비되지 않으면 실행 안 함
        }

        currentDuration += Time.deltaTime; // 유지 시간 증가

        if(currentDuration > rangeWeaponHandler.Duration)
        {
            DestroyProjectile(transform.position, false); // 유지 시간이 지나면 삭제
        }

        _rigidbody.velocity = direction * rangeWeaponHandler.Speed; // 방향에 따라 이동
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if(levelCollisionLayer.value == (levelCollisionLayer.value | (1 << collision.gameObject.layer)))
        {
            DestroyProjectile(collision.ClosestPoint(transform.position) - direction * .2f, fxOnDestory); // 장애물 충돌 시 삭제
        }
        else if(rangeWeaponHandler.target.value == (rangeWeaponHandler.target.value | (1 << collision.gameObject.layer)))
        {
            DestroyProjectile(collision.ClosestPoint(transform.position), fxOnDestory); // 타겟 충돌 시 삭제
        }
    }

    public void Init(Vector2 direction, RangeWeaponHandler weaponHandler)
    {
        rangeWeaponHandler = weaponHandler;
        
        this.direction = direction;
        currentDuration = 0;
        transform.localScale = Vector3.one * weaponHandler.BulletSize; // 발사체 크기 설정
        spriteRenderer.color = weaponHandler.ProjectileColor; // 발사체 색상 설정

        transform.right = this.direction; // 방향 설정
        
        if (this.direction.x < 0)
            pivot.localRotation = Quaternion.Euler(180, 0, 0); // 왼쪽 방향이면 회전
        else
            pivot.localRotation = Quaternion.Euler(0, 0, 0); // 오른쪽 방향

        isReady = true; // 발사 준비 완료
    }
    
    private void DestroyProjectile(Vector3 position, bool createFx)
    {
        Destroy(this.gameObject); // 발사체 삭제
    }
}

```

---

## 투사제 오브젝트 준비

:one: 투사체 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "Arrow"
  - Add Component: "Rigidbody 2D"
    - Body Type: "Kinemeatic"
  - Add Component: "BoxColider 2D"
    - Size: 이미지와 같은 크기로 지정
    - Is Trigger: True

:two: 투사체 하위 오브젝트 생성

- Hierarchy에서 Arrow 오브젝트 우클릭 -> Create Empty
  - 이름: "Pivot"
- Hierarchy에서 Pivot 오브젝트 우클릭 -> Create Empty
  - 이름: "WeaponRenderer"
  - Rotation: (0, 0, -90) (스프라이트가 오른쪽을 바라보게)
  - Add Component: "Sprite Renderer"
    - Sprite: "weapon_arrow"
    - Additional Settings -> Order in Layer: 110

:three: 투사체 스크립트 연결

- Arrow 오브젝트에 ProjectileController 스크립트를 드래그하여 연결

:four: 프리팹 저장

- Prefabs/Projectile에 프리팹 저장
- Scene에서 삭제

:five: 추가 프리팹

- 동일한 방법으로 "Skull" 생성
- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "Skull"
  - Add Component: "Rigidbody 2D"
    - Body Type: "Kinemeatic"
  - Add Component: "BoxColider 2D"
    - Size: 이미지와 같은 크기로 지정
    - Is Trigger: True
- Hierarchy에서 Arrow 오브젝트 우클릭 -> Create Empty
  - 이름: "Pivot"
- Hierarchy에서 Pivot 오브젝트 우클릭 -> Create Empty
  - 이름: "WeaponRenderer"
  - Rotation: (0, 180, 0) (스프라이트가 오른쪽을 바라보게)
  - Add Component: "Sprite Renderer"
    - Sprite: "weapon_arrow"
    - Additional Settings -> Order in Layer: 110

---

## 레이어 설정

:one: Player, Enemy, Level 레이어 추가

:two: P_Bow_EqupWeapon 오브젝트 Target -> "Enemy"

:three: 투사체 -> Projectile Controller -> Level Collision Layer ->"Level"

:four: Grid -> Collision 오브젝트 -> "Level" Layer 적용

---

#### 투사체 매니저 설정

![image-20250220214350890](/Images/Untitled//image-20250220214350890.png)

---

## 투사체 생성

:one: RangeWeaponHandler 스크립트 수정

```csharp
private ProjectileManager projectileManager;
protected override void Start()
{
    base.Start();
    projectileManager = ProjectileManager.Instance;
}

(기존코드)

private void CreateProjectile(Vector2 _lookDirection, float angle)
{
    projectileManager.ShootBullet(
        this,
        projectileSpawnPosition.position,
        RotateVector2(_lookDirection, angle));
}

(기존코드)
```

---

#### 이동 파티클 구현

:one: 파티클 오브젝트 생성

- Hierarchy에서 Player 우클릭 -> Effects -> Paricle System
  - 이름 : "DustParticle"
  - Position: (0, -0.5, 0)

:two: 파티클 컴포넌트 설정

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F76e97c6a-88c1-4bac-9e45-cb9bfd3bc14b%2Fimage.png?table=block&id=17f2dc3e-f514-8181-8b85-f082ff35378a&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=820&userId=&cache=v2)

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fd363cf4f-a30a-4d1f-aef9-d66084362312%2Fimage.png?table=block&id=17f2dc3e-f514-810a-9b0a-d54200a8a8a7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=820&userId=&cache=v2)

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F9fb2881f-842c-4566-9cd8-065e0721b99e%2Fimage.png?table=block&id=17f2dc3e-f514-81c6-a49f-f2d9f5e6ced7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=820&userId=&cache=v2)

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F29116510-cfee-48a4-b2b4-08ea7f6465c4%2Fimage.png?table=block&id=17f2dc3e-f514-81f5-ba52-ca4de6dae18f&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1290&userId=&cache=v2)

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F933a3507-f247-4e71-a83f-d817ff3c088f%2Fimage.png?table=block&id=17f2dc3e-f514-8191-aa6a-e862b658d3b7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=830&userId=&cache=v2)

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F65a2ffe6-239b-45d8-bb37-2a9cb1de353e%2Fimage.png?table=block&id=17f2dc3e-f514-81eb-823e-f1409f4c27df&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=820&userId=&cache=v2)

:three: DustParticleControl 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성: `DustParticleControl.cs`
- Player/MainSprite 오브젝트에 스크립트를 드래그하여 연결

:four: DustParticleControl 스크립트 작성

```csharp
using UnityEngine;

public class DustParticleControl : MonoBehaviour
{
    [SerializeField] private bool createDustOnWalk = true;
    [SerializeField] private ParticleSystem dustParticleSystem;

    public void CreateDustParticles()
    {
        if (createDustOnWalk)
        {
            dustParticleSystem.Stop();
            dustParticleSystem.Play();
        }
    }
}
```

:five: Dust Particle Control 설정

![image-20250220221321601](/Images/Untitled//image-20250220221321601.png)

:six: Player/MainSprite의 Animation에 이벤트 추가

![image-20250220221403424](/Images/Untitled//image-20250220221403424.png)

---

#### 투사체 파티클 구현

:one: 파티클 오브젝트 생성

- Hierarchy에서 Player 우클릭 -> Effects -> Paricle System
  - 이름 : "ImpactParticleSystem"

:two: 파티클 컴포넌트 설정

:three: ProjectileManager 스크립트 수정

```c#
[SerializeField] private ParticleSystem impactParticleSystem; //추가

(기존코드)

public void ShootBullet(RangeWeaponHandler rangeWeaponHandler, Vector2 startPostiion, Vector2 direction )
{
    GameObject origin = projectilePrefabs[rangeWeaponHandler.BulletIndex];
    GameObject obj = Instantiate(origin,startPostiion,Quaternion.identity);
    
    ProjectileController projectileController = obj.GetComponent<ProjectileController>(); 
    projectileController.Init(direction, rangeWeaponHandler, this);
}

public void CreateImpactParticlesAtPostion(Vector3 position, RangeWeaponHandler weaponHandler)
{
    impactParticleSystem.transform.position = position; // 파티클 위치 설정
    
    // 파티클 시스템의 Emission 모듈 가져오기 및 버스트 설정
    ParticleSystem.EmissionModule em = impactParticleSystem.emission;
    em.SetBurst(0, new ParticleSystem.Burst(0, Mathf.Ceil(weaponHandler.BulletSize * 5))); // 탄환 크기에 따라 파티클 개수 조정
    
    // 파티클 시스템의 Main 모듈 가져오기 및 속도 설정
    ParticleSystem.MainModule mainModule = impactParticleSystem.main;
    mainModule.startSpeedMultiplier = weaponHandler.BulletSize * 10f; // 탄환 크기에 따라 파티클 속도 조정
    
    impactParticleSystem.Play(); // 파티클 효과 재생
}
```

:four: ProjectileController 스크립트 수정

```csharp
private ProjectileManager projectileManager; // 발사체 매니저 참조

public void Init(Vector2 direction, RangeWeaponHandler weaponHandler, ProjectileManager projectileManager)
{
    this.projectileManager = projectileManager; // 발사체 매니저 설정
    
    rangeWeaponHandler = weaponHandler; // 발사 무기 핸들러 설정
    
    this.direction = direction; // 발사 방향 설정
    currentDuration = 0; // 발사체 지속 시간 초기화
    transform.localScale = Vector3.one * weaponHandler.BulletSize; // 발사체 크기 설정
    spriteRenderer.color = weaponHandler.ProjectileColor; // 발사체 색상 설정

    transform.right = this.direction; // 발사 방향에 맞게 회전
    
    // 발사 방향이 왼쪽이면 발사체를 반전
    if (this.direction.x < 0)
        pivot.localRotation = Quaternion.Euler(180, 0, 0);
    else
        pivot.localRotation = Quaternion.Euler(0, 0, 0);

    isReady = true; // 발사 준비 완료
}

private void DestroyProjectile(Vector3 position, bool createFx)
{
    if(createFx)
    {
        projectileManager.CreateImpactParticlesAtPostion(position, rangeWeaponHandler); // 충돌 이펙트 생성
    }
    
    Destroy(this.gameObject); // 발사체 삭제
}

```

:five: Projectile Manager 설정

![image-20250220222849349](/Images/Untitled//image-20250220222849349.png)

---

#### 적 오브젝트 구현

:one: 적 오브젝트 구현

![image-20250220223052831](/Images/Untitled//image-20250220223052831.png)

:two: 컴포넌트 추가

- Box Collider 2D
  - 스프라이트의 원점에 맞춰 크기지정
- Rigidbody 2D
  - Gravity Sclae: 0
  - Constraints -> Freeze Rotation -> Z: True
- Resource Controller
- Animation Handler
- Stat Handler
  - Speed: "2"

:three: 오브젝트 레이어 변경

- Layer: "Enemy"

:four: 프리팹 저장 후 Scene에서 삭제

- Prefabs/Enemy 폴더에 드래그 앤 드랍으로 프리팹 생성
- Scene에서 오브젝트 삭제

---

#### 적 로직 구현

:one: EnemyController 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성: `EnemyController.cs`
- 각 적 프리팹에 스크립트를 드래그 하여 연결

:two: EnemyController 스크립트 작성

```csharp
using UnityEngine;

public class EnemyController : BaseController
{
    private EnemyManager enemyManager; // 적 관리 매니저
    private Transform target; // 목표 대상

    [SerializeField] private float followRange = 15f; // 추적 범위

    // 적 초기화 메서드
    public void Init(EnemyManager enemyManager, Transform target)
    {
        this.enemyManager = enemyManager;
        this.target = target;
    }

    // 목표와의 거리 반환
    protected float DistanceToTarget()
    {
        return Vector3.Distance(transform.position, target.position);
    }

    // 적의 행동 처리
    protected override void HandleAction()
    {
        base.HandleAction();

        // 무기 또는 대상이 없으면 이동을 멈춤
        if (weaponHandler == null || target == null)
        {
            if (!movementDirection.Equals(Vector2.zero))
                movementDirection = Vector2.zero;
            return;
        }

        float distance = DistanceToTarget();
        Vector2 direction = DirectionToTarget();
        isAttacking = false;

        // 목표가 추적 범위 내에 있을 경우
        if (distance <= followRange)
        {
            lookDirection = direction;

            // 공격 범위 내에 있을 경우
            if (distance <= weaponHandler.AttackRange)
            {
                int layerMaskTarget = weaponHandler.target;

              // 공격 가능한 대상 확인
              RaycastHit2D hit = Physics2D.Raycast(
                transform.position,   // 시작 위치 (적의 현재 위치)
                direction,            // 발사 방향 (목표 방향)
                weaponHandler.AttackRange * 1.5f, // 레이 길이 (공격 범위의 1.5배)
                (1 << LayerMask.NameToLayer("Level")) | layerMaskTarget // 충돌할 레이어 설정
              );

                if (hit.collider != null &&
                    layerMaskTarget == (layerMaskTarget | (1 << hit.collider.gameObject.layer)))
                {
                    isAttacking = true;
                }

                movementDirection = Vector2.zero;
                return;
            }

            // 공격 범위 밖이면 목표 방향으로 이동
            movementDirection = direction;
        }
    }

    // 목표 방향 벡터 반환
    protected Vector2 DirectionToTarget()
    {
        return (target.position - transform.position).normalized;
    }
}
```

:five: EnemyController 스크립트 설정

![image-20250220224545365](/Images/Untitled//image-20250220224545365.png)

---

#### 적 매니저 및 생성 로직 구현

:one: EnemyManager 스크립트 생성

- Assets/Scripts/Entity 폴더에서 새 스크립트 생성: EnemyManager.cs

:two: EnemyManager 스크립트 작성

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Random = UnityEngine.Random;

public class EnemyManager : MonoBehaviour
{
    private Coroutine waveRoutine;
        
    [SerializeField]
    private List<GameObject> enemyPrefabs; // 생성할 적 프리팹 리스트

    [SerializeField]
    private List<Rect> spawnAreas; // 적을 생성할 영역 리스트

    [SerializeField]
    private Color gizmoColor = new Color(1, 0, 0, 0.3f); // 기즈모 색상

    private List<EnemyController> activeEnemies = new List<EnemyController>(); // 현재 활성화된 적들

    private bool enemySpawnComplite;
    
    [SerializeField] private float timeBetweenSpawns = 0.2f;
    [SerializeField] private float timeBetweenWaves = 1f;

    public void StartWave(int waveCount)
    {
        if(waveRoutine != null) 
            StopCoroutine(waveRoutine); // 코루틴이 null이 아니라면 기존에 실행 중이던 코루틴을 중지
        waveRoutine =  StartCoroutine(SpawnWave(waveCount));
    }

    public void StopWave()
    {
        StopAllCoroutines();
    }

    private IEnumerator SpawnWave(int waveCount)
    {
        enemySpawnComplite = false;
        yield return new WaitForSeconds(timeBetweenWaves);
        for (int i = 0; i < waveCount; i++)
        {
            yield return new WaitForSeconds(timeBetweenSpawns); 
            SpawnRandomEnemy();
        }

        enemySpawnComplite = true;
    }

    private void SpawnRandomEnemy()
    {
        if (enemyPrefabs.Count == 0 || spawnAreas.Count == 0)
        {
            Debug.LogWarning("Enemy Prefabs 또는 Spawn Areas가 설정되지 않았습니다.");
            return;
        }

        // 랜덤한 적 프리팹 선택
        GameObject randomPrefab = enemyPrefabs[Random.Range(0, enemyPrefabs.Count)];

        // 랜덤한 영역 선택
        Rect randomArea = spawnAreas[Random.Range(0, spawnAreas.Count)];

        // Rect 영역 내부의 랜덤 위치 계산
        Vector2 randomPosition = new Vector2(
            Random.Range(randomArea.xMin, randomArea.xMax),
            Random.Range(randomArea.yMin, randomArea.yMax)
        );

        // 적 생성 및 리스트에 추가
        GameObject spawnedEnemy = Instantiate(randomPrefab, new Vector3(randomPosition.x, randomPosition.y), Quaternion.identity);
        EnemyController enemyController = spawnedEnemy.GetComponent<EnemyController>();

        activeEnemies.Add(enemyController);
    }

    // 기즈모를 그려 영역을 시각화 (선택된 경우에만 표시)
    private void OnDrawGizmosSelected()
    {
        if (spawnAreas == null) return;

        Gizmos.color = gizmoColor;
        foreach (var area in spawnAreas)
        {
            Vector3 center = new Vector3(area.x + area.width / 2, area.y + area.height / 2);
            Vector3 size = new Vector3(area.width, area.height);
            Gizmos.DrawCube(center, size);
        }
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            StartWave(1);
        }
    }
}

```

:three: EnemyManager 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty 

  - 이름: "EnemyManager"
  - 다음 이미지와 같이 프리팹, Rect 설정

  

  ![image-20250220230006170](/Images/Untitled//image-20250220230006170.png)

  > **코루틴(Coroutine)이란?**
  >
  > **코루틴**은 Unity에서 **시간 기반 작업**이나 **비동기적 작업**을 처리하기 위해 사용되는 메서드입니다. 일반적인 함수와 달리, **코루틴은 실행을 중단하고 나중에 다시 실행을 재개**할 수 있습니다.
  >
  > **특징**
  >
  > 1. 비동기적 작업
  >    - 프레임 단위로 나뉘어 실행되므로, 한 프레임에 모든 작업을 처리하지 않아도 됩니다.
  >    - 게임이 멈추지 않고 연속적으로 작동할 수 있습니다.
  > 2. `yield` 키워드를 사용
  >    - `yield return`을 통해 특정 조건이나 시간을 기다렸다가 작업을 재개할 수 있습니다.
  >
  > - **코루틴에서 사용되는 `yield return`의 종류**
  >   1. **`yield return null`**
  >      - **설명**: 다음 프레임까지 대기
  >   2. **`yield return new WaitForSeconds(시간)`**
  >      - **설명**: 지정된 시간(초)만큼 대기
  >   3. **`yield return new WaitForSecondsRealtime(시간)`**
  >      - **설명**: 실제 시간(Real Time) 기준으로 대기
  >        - `Time.timeScale`의 영향을 받지 않음
  >   4. **`yield return new WaitUntil(조건)`**
  >      - **설명**: 조건이 참(True)이 될 때까지 대기
  >   5. **`yield return new WaitWhile(조건)`**
  >      - **설명**: 조건이 거짓(False)이 될 때까지 대기
  >   6. **`yield break`**
  >      - **설명**: 코루틴 실행을 즉시 종료

---

#### 피격 로직 구현

:one: ProjectileContorller 스크립트 수정

```csharp
using UnityEngine;

public class ProjectileController : MonoBehaviour
{
    [SerializeField] private LayerMask levelCollisionLayer; // 장애물 충돌 감지 레이어

    private RangeWeaponHandler rangeWeaponHandler; // 발사 무기 핸들러
    
    private float currentDuration; // 발사체 지속 시간
    private Vector2 direction; // 발사 방향
    private bool isReady; // 발사 가능 여부
    private Transform pivot; // 발사체 피벗
    
    private Rigidbody2D _rigidbody;
    private SpriteRenderer spriteRenderer;

    public bool fxOnDestory = true; // 충돌 시 파티클 효과 생성 여부
    
    private ProjectileManager projectileManager; // 발사체 매니저 참조
    
    private void Awake()
    {
        spriteRenderer = GetComponentInChildren<SpriteRenderer>(); // 자식 오브젝트에서 스프라이트 렌더러 가져오기
        _rigidbody = GetComponent<Rigidbody2D>(); // Rigidbody2D 가져오기
        pivot = transform.GetChild(0); // 첫 번째 자식(피벗) 가져오기
    }

    private void Update()
    {
        if (!isReady) 
        { 
            return; // 발사 준비가 되지 않았다면 실행하지 않음
        }

        currentDuration += Time.deltaTime; // 발사체 지속 시간 증가

        if(currentDuration > rangeWeaponHandler.Duration)
        {
            DestroyProjectile(transform.position, false); // 지속 시간이 지나면 삭제
        }

        _rigidbody.velocity = direction * rangeWeaponHandler.Speed; // 발사체 이동 속도 설정
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        // 장애물과 충돌했는지 확인
        if(levelCollisionLayer.value == (levelCollisionLayer.value | (1 << collision.gameObject.layer)))
        {
            DestroyProjectile(collision.ClosestPoint(transform.position) - direction * .2f, fxOnDestory); // 약간 뒤쪽에서 삭제하여 자연스럽게 보이도록 처리
        }
        // 타겟(적 또는 플레이어)과 충돌했는지 확인
        else if(rangeWeaponHandler.target.value == (rangeWeaponHandler.target.value | (1 << collision.gameObject.layer)))
        {
            ResourceController resourceController = collision.GetComponent<ResourceController>(); // 체력 관리 컴포넌트 가져오기
            if(resourceController != null)
            {
                resourceController.ChangeHealth(-rangeWeaponHandler.Power); // 공격력만큼 체력 감소
                
                // 넉백 효과가 활성화되어 있으면 적용
                if(rangeWeaponHandler.IsOnKnockback)
                {
                    BaseController controller = collision.GetComponent<BaseController>();
                    if(controller != null)
                    {
                        controller.ApplyKnockback(transform, rangeWeaponHandler.KnockbackPower, rangeWeaponHandler.KnockbackTime); // 넉백 적용
                    }
                }
            }
            
            DestroyProjectile(collision.ClosestPoint(transform.position), fxOnDestory); // 적과 충돌 시 삭제
        }
    }

    public void Init(Vector2 direction, RangeWeaponHandler weaponHandler, ProjectileManager projectileManager)
    {
        this.projectileManager = projectileManager; // 발사체 매니저 저장
        
        rangeWeaponHandler = weaponHandler; // 무기 핸들러 저장
        
        this.direction = direction; // 방향 설정
        currentDuration = 0; // 지속 시간 초기화
        transform.localScale = Vector3.one * weaponHandler.Size; // 발사체 크기 설정
        spriteRenderer.color = weaponHandler.ProjectileColor; // 발사체 색상 설정

        transform.right = this.direction; // 발사 방향 설정
        
        // 발사 방향에 따라 피벗 회전
        if (this.direction.x < 0)
            pivot.localRotation = Quaternion.Euler(180, 0, 0);
        else
            pivot.localRotation = Quaternion.Euler(0, 0, 0);

        isReady = true; // 발사 준비 완료
    }
    
    private void DestroyProjectile(Vector3 position, bool createFx)
    {
        if(createFx)
        {
            projectileManager.CreateImpactParticlesAtPostion(position, rangeWeaponHandler); // 충돌 이펙트 생성
        }
        
        Destroy(this.gameObject); // 발사체 삭제
    }
}
```

---

#### 사망 판단 구현

:one: BaseController 스크립트 수정

```csharp
public virtual void Death()
{
    _rigidbody.velocity = Vector3.zero; // 사망 시 이동을 멈춤

    // 모든 자식 SpriteRenderer의 투명도를 낮춤
    foreach (SpriteRenderer renderer in transform.GetComponentsInChildren<SpriteRenderer>())
    {
        Color color = renderer.color;
        color.a = 0.3f; // 반투명 효과 적용
        renderer.color = color;
    }

    // 모든 자식 Behaviour 컴포넌트 비활성화 (예: AI, 입력, 애니메이션 등)
    foreach (Behaviour component in transform.GetComponentsInChildren<Behaviour>())
    {
        component.enabled = false;
    }

    Destroy(gameObject, 2f); // 2초 후 오브젝트 삭제
}

```

:two: PlayerController 스크립트 수정

```csharp
public override void Death()
{
    base.Death();
    gameManager.GameOver();
}
```

:three: EnemyController 스크립트 수정

```csharp
public override void Death()
{
    base.Death();
}
```

:four: ResourceController 스크립트 수정

```csharp
private void Death()
{
    baseController.Death();
}
```

---

#### 추가 무기 프리팹 생성

:one: 추가 무기 프리팹 생성

- 기존의 "P_Bow_EquipWeapon" 프리팹 복제
- 이름: "E_Fast Bow_EquipWapon"
- 아래의 이미지와 같이 수정

![image-20250220234509989](/Images/Untitled//image-20250220234509989.png)

:two: 추가 무기 프리팹 생성

- 기존의 "P_Bow_EquipWeapon" 프리팹 복제
- 이름: "E_Slow Staff_EquipWapon"
- 아래의 이미지와 같이 수정
  - WeaponSprite -> Sprite: "weapon_green_magic_staff"

![image-20250220234650960](/Images/Untitled//image-20250220234650960.png)

---

#### 무기 장착 설정

:one: 무기장착

![image-20250220234723181](/Images/Untitled//image-20250220234723181.png)

:two: 무기장착

![image-20250220234742308](/Images/Untitled//image-20250220234742308.png)

---

#### 근거리 무기 로직 구현

:one: MeleeWeaponHandler 스크립트 생성

- Assets/Scripts/Weapon 폴더에서 새 스크립트 생성: `MeleeWeaponHandler.cs`

:two: MeleeWeaponHandler 스크립트 작성

```csharp
```

---

#### 무기 오브젝트 생성

:one: 무기 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "E_Knife_EqupWeapon"
  - Add Component: "MeleeWeaponHadler"

:two: 무기 하위 오브젝트 생성

- Hierarchy에서 E_Knife_EquipWeapon 오브젝트 우클릭 -> Create Empty
  - 이름: "WeaponSprite"
  - Position: (0.3, 0, 0)
  - Add Component: "Sprite Renderer"
    - Sprite: "weapon_knife"
    - Additional  Settings -> Order in Layer: 120

---

##### 무기 애니메이션 생성

:one: 챕터 4-6: 원거리 공격 구현 참고하여 무기 애니메이션 제작

![image-20250220235821418](/Images/Untitled/근거리_공격.gif)

---

근거리 적 설정

:one: E_Knife_EquipWeapon 오브젝트 설정

- 다음 이미지와 같이 설정

- 프리팹으로 저장

- Scene에서 삭제

  ![image-20250221000102896](/Images/Untitled//image-20250221000102896.png)

  

  

:two: 적 복제 및 근거리 생성

- 적 프리팹 복제

- Weapon Prefab 변경

  ![image-20250221000151234](/Images/Untitled//image-20250221000151234.png)

:three: EnemyManager/Enemy Prefabs에 복제된 Goblin1 추가

---

#### 사운드 소스 오브젝트 구현

:one: 사운드소스 오브젝트 생성

- Hierarchy에서 새 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "SoundSource"
  - Add Component: "Audio Source"
    - Play On Awake: False;

:two: SoundSource 스크립트 생성

- Assets/Scripts/Manager 폴더에서 새 스크립트 생성: `SoundSource.cs`
- SoundSource 오브젝트에 스크립트를 드래그하여 연결

:three: SoundSource 스크립트 작성

```csharp
using UnityEngine;

public class SoundSource : MonoBehaviour
{
    private AudioSource _audioSource; // 오디오 소스 컴포넌트

    public void Play(AudioClip clip, float soundEffectVolume, float soundEffectPitchVariance)
    {
        if (_audioSource == null)
            _audioSource = GetComponent<AudioSource>(); // 오디오 소스 가져오기 (초기화)

        CancelInvoke(); // 기존에 예약된 Invoke 취소
        _audioSource.clip = clip; // 재생할 오디오 클립 설정
        _audioSource.volume = soundEffectVolume; // 볼륨 설정
        _audioSource.Play(); // 오디오 재생
        
        // 피치(음 높낮이) 설정 (랜덤 변동 적용)
        _audioSource.pitch = 1f + Random.Range(-soundEffectPitchVariance, soundEffectPitchVariance);

        // 오디오 클립 길이 + 2초 후 자동 비활성화
        Invoke("Disable", clip.length + 2);
    }

    public void Disable()
    {
        _audioSource.Stop(); // 오디오 정지
        Destroy(this.gameObject); // 오브젝트 삭제
    }
}

```

:four: 프리팹 생성

- Assets/Prefabs/Global 폴더에서 프리팹 생성
- Scene에서 삭제

---

#### 사운드 매니저 생성

:one: 사운드 매니저 오브젝트 생성

- Hierarchy에서 GameManager 오브젝트 생성: 우클릭 -> Create Empty
  - 이름: "SoundManager"
  - Add Component:  "Audio Source"

:two: SoundManager 스크립트 생성

- Assets/Scripts/Manager 폴더에서 새 스크립트 생성: `SoundManager.cs`
- SoundManager 오브젝트에 스크립트를 드래그하여 연결

:three: SoundManager 스크립트 작성

```csharp
using UnityEngine;

public class SoundManager : MonoBehaviour
{
    public static SoundManager instance; // 싱글톤 인스턴스

    [SerializeField][Range(0f, 1f)] private float soundEffectVolume; // 효과음 볼륨
    [SerializeField][Range(0f, 1f)] private float soundEffectPitchVariance; // 효과음 피치 변동 범위
    [SerializeField][Range(0f, 1f)] private float musicVolume; // 배경 음악 볼륨

    private AudioSource musicAudioSource; // 배경 음악 재생을 위한 오디오 소스
    public AudioClip musicClip; // 현재 배경 음악 클립

    public SoundSource soundSourcePrefab; // 효과음 재생을 위한 사운드 소스 프리팹

    private void Awake()
    {
        instance = this; // 싱글톤 인스턴스 설정
        musicAudioSource = GetComponent<AudioSource>(); // 오디오 소스 가져오기
        musicAudioSource.volume = musicVolume; // 배경 음악 볼륨 설정
        musicAudioSource.loop = true; // 배경 음악 반복 재생 설정
    }

    private void Start()
    {
        ChangeBackGroundMusic(musicClip); // 게임 시작 시 배경 음악 변경
    }

    public void ChangeBackGroundMusic(AudioClip clip)
    {
        musicAudioSource.Stop(); // 현재 배경 음악 정지
        musicAudioSource.clip = clip; // 새로운 배경 음악 설정
        musicAudioSource.Play(); // 배경 음악 재생
    }

    public static void PlayClip(AudioClip clip)
    {
        SoundSource obj = Instantiate(instance.soundSourcePrefab); // 새로운 사운드 소스 오브젝트 생성
        SoundSource soundSource = obj.GetComponent<SoundSource>(); // 사운드 소스 가져오기
        soundSource.Play(clip, instance.soundEffectVolume, instance.soundEffectPitchVariance); // 효과음 재생
    }
}

```

:four: SoundManager 설정

- Prefab에 SoundSource 연결
- Music Clip에 Goblins_den_(Regular) 연결

![image-20250221001411011](/Images/Untitled//image-20250221001411011.png)



---

### 무기 사운드 구현

:one: WeaponHandler 스크립트 수정

```csharp
public AudioClip attackSoundClip;
   
public virtual void Attack()
{
    AttackAnimation();
    
    if (attackSoundClip != null)
        SoundManager.PlayClip(attackSoundClip);
}
```

:two: 사운드 클립 설정

- P_Bow_EquipWeapon → 03_crate_open_2
- E_Slow Staff_EquipWeapon → 15_human_dash_2
- E_Knife_EquipWeapon → 17_orc_atk_sword_2
- E_Fast Bow_EquipWeapon → 03_crate_open_2

---

#### 피격 사운드 구현

:one: ResourceController 스크립트 수정

```csharp
public AudioClip damageClip;
   
public bool ChangeHealth(float change)
{
    if (change == 0 || timeSinceLastChange < healthChangeDelay)
    {
        return false;
    }

    timeSinceLastChange = 0f;
    CurrentHealth += change;
    CurrentHealth = CurrentHealth > MaxHealth ? MaxHealth : CurrentHealth;
    CurrentHealth = CurrentHealth < 0 ? 0 : CurrentHealth;

    if (change < 0)
    {
        animationHandler.Damage();
        
        if (damageClip != null)
            SoundManager.PlayClip(damageClip);
    }

    if (CurrentHealth <= 0f)
    {
        Death();
    }

    return true;
}
```

:two: 사운드 클립 설정

- Player -> 11_human_damage_3
- Orc_Shaman, Goblin, Goblin 1 -> 21_orc_damage_3

---

#### UI 준비하기 구현

:one: UI Canvas 생성

- Hierarchy에서  새 오브젝트 생성: 우클릭 -> UI -> Canvase
  - Canvas Scaler
    - UI Scale Mode: Scale With Screen Szie
    - Reference Resoulution: (1920, 1080)

![image-20250221003338170](/Images/Untitled//image-20250221003338170.png)

:two: UI Object 생성

- Hierarchy에서 Canvas 오브젝트 우클릭 -> Create Empty
  - 이름: "HomeUI"
  - 이름: "GameUI"
  - 이름: "GameOverUI"

:three: BaseUI 스크립트 작성

- Assets/Scripts/UI 폴더에서 새로운 스크립트 생성: `BaseUI.cs`

:four: BaseUI 스크립트 작성

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class BaseUI : MonoBehaviour
{
    protected UIManager uiManager;

    public virtual void Init(UIManager uiManager)
    {
        this.uiManager = uiManager;
    }
    
    protected abstract UIState GetUIState(); 
    public void SetActive(UIState state)
    {
        this.gameObject.SetActive(GetUIState() == state);
    }
}
```

:five: ​UIManager 스크립트 작성

- Assets/Scripts/Manager 폴더에서 새로운 스크립트 생성: `UIManager.cs`
- Canvas 오브젝트에 스크립트를 드래그하여 연결

:six: UIManager 스크립트 작성

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum UIState
{
    Home,
    Game,
    GameOver,
}

public class UIManager : MonoBehaviour
{   
    HomeUI homeUI;
    GameUI gameUI;
    GameOverUI gameOverUI;
    private UIState currentState;

    private void Awake()
    {
     		//여기서 true는 비활성화된(Deactivated) 자식 오브젝트에서도 컴포넌트를 찾을 수 있도록 허용하는 옵션입니다.
        homeUI = GetComponentInChildren<HomeUI>(true);
        homeUI.Init(this);
        gameUI = GetComponentInChildren<GameUI>(true);
        gameUI.Init(this);
        gameOverUI = GetComponentInChildren<GameOverUI>(true);
        gameOverUI.Init(this);
        
        ChangeState(UIState.Home);
    }

    public void SetPlayGame()
    {
        ChangeState(UIState.Game);
    }
    
    public void SetGameOver()
    {
        ChangeState(UIState.GameOver);
    }

    public void ChangeWave(int waveIndex)
    {
        gameUI.UpdateWaveText(waveIndex);
    }

    public void ChangePlayerHP(float currentHP, float maxHP)
    {
        gameUI.UpdateHPSlider(currentHP/ maxHP);
    }
    
    public void ChangeState(UIState state)
    {
        currentState = state;
        homeUI.SetActive(currentState);
        gameUI.SetActive(currentState);
        gameOverUI.SetActive(currentState);
    }
}
```

> **UI 앵커란?**
>
> **UI 앵커(UI Anchor)**는 Unity에서 **Canvas** 상의 UI 요소가 화면 크기나 해상도가 변경되더라도 일정한 위치와 크기를 유지하도록 돕는 기능입니다.
>
> 앵커는 UI 요소와 부모 객체(Canvas 또는 다른 UI 요소) 간의 위치 관계를 정의하며, 유연하고 반응형 디자인을 구현하는 데 핵심적인 역할을 합니다.

---

#### UI 스크립트 준비

:one: HomeUI 스크립트 작성

- **Assets/Scripts/UI** 폴더에서 새로운 스크립트 생성: `HomeUI.cs`
- HomeUI 오브젝트에 스크립트를 드래그하여 연결 

:two:  HomeUI 스크립트 작성

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class HomeUI : BaseUI
{
    [SerializeField] private Button startButton;
    [SerializeField] private Button exitButton;

    public override void Init(UIManager uiManager)
    {
        base.Init(uiManager);
        startButton.onClick.AddListener(OnClickStartButton);
        exitButton.onClick.AddListener(OnClickExitButton);
    }
    
    public void OnClickStartButton()
    {
        GameManager.instance.StartGame();
    }

    public void OnClickExitButton()
    {
        Application.Quit();
    }
    
    protected override UIState GetUIState()
    {
        return UIState.Home;
    }
}
```

:three: GameUI 스크립트 작성

- Assets/Scripts/UI 폴더에서 새로운 스크립트 생성: `GameUI.cs`
- GameUI 오브젝트에 스크립트를 드래그하여 연결

:four: GameUI 스크립트 작성

```csharp
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class GameUI : BaseUI
{
    [SerializeField] private TextMeshProUGUI waveText;
    [SerializeField] private Slider hpSlider;

    private void Start()
    {
        UpdateHPSlider(1);
    }
    
    public void UpdateHPSlider(float percentage)
    {
        hpSlider.value = percentage;
    }

    public void UpdateWaveText(int wave)
    {
        waveText.text = wave.ToString();
    }

    protected override UIState GetUIState()
    {
        return UIState.Game;
    }
}
```

:five: GameOverUI 스크립트 작성

- **Assets/Scripts/UI** 폴더에서 새로운 스크립트 생성: `GameOverUI.cs`
- GameOverUI 오브젝트에 스크립트를 드래그하여 연결

:six: GameOverUI 스크립트 작성

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.UI;

public class GameOverUI : BaseUI
{
    [SerializeField] private Button restartButton;
    [SerializeField] private Button exitButton;

    public override void Init(UIManager uiManager)
    {
        base.Init(uiManager);
        restartButton.onClick.AddListener(OnClickRestartButton);
        exitButton.onClick.AddListener(OnClickExitButton);
    }
    
    public void OnClickRestartButton()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    public void OnClickExitButton()
    {
        Application.Quit();
    }

    protected override UIState GetUIState()
    {
        return UIState.GameOver;
    }
}
```

---

#### Home UI 만들기

:one: HomeUI Anchor

- Alt 키를 누른 상태로 앵커지정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F0a2fb261-4ec5-4888-85c3-2a2b1f9a64e3%2Fimage.png?table=block&id=08ed1751-ffcf-4388-888d-3a04d5509cbf&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=890&userId=&cache=v2)

:two: Start Button 생성

- Hierarchy에서 GameOverUI 오브젝트 우클릭 -> UI -> Button -TextMeshPro

  - 이름: "StartButton"

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F34157d4b-8d31-4469-bc3d-925b42bc7428%2Fimage.png?table=block&id=2707d485-0f3b-481a-b8ad-1ef596b0c11e&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=550&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fcf5e3a05-3b49-4cf3-bed4-07a33b4bc8a5%2Fimage.png?table=block&id=3968a3b1-cef2-4299-b4b6-f618d4e3b951&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=570&userId=&cache=v2)

:three: Exit Button 생성

- Hierarchy에서 GameOverUI 오브젝트 우클릭 -> UI -> Button -TextMeshPro

  - 이름: "StartButton"

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F34157d4b-8d31-4469-bc3d-925b42bc7428%2Fimage.png?table=block&id=bfd93e2c-d2e6-48ac-ad05-2cf41d14f366&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=550&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F2d3bbf61-aa39-45d1-8cae-790fe4f60b73%2Fimage.png?table=block&id=46c00a80-8b76-48cc-9042-984ad52622f7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=580&userId=&cache=v2)

  :four: 생성된 버튼들을 자유롭게 꾸며주세요.

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fc0b9bdc7-3be0-472d-8e6a-1ba92b9285fa%2Fimage.png?table=block&id=4a985d99-01a1-4109-badd-781394f53082&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1110&userId=&cache=v2)	

  :five: Home UI 설정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F6ff97844-33db-4124-afb7-4a1f4f151813%2Fimage.png?table=block&id=3d3f6ba1-0103-4bfb-92a0-1756d55fc023&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1290&userId=&cache=v2) 

  ---

  #### Home UI 로직 구현

  :one: GameManager 스크립트 수정

  ```csharp
  private UIManager uiManager;
  public static bool isFirstLoading = true;
  
  private void Awake()
  {
      instance = this;
      player = FindObjectOfType<PlayerController>();
      player.Init(this);
  
      uiManager = FindObjectOfType<UIManager>();
  
      enemyManager = GetComponentInChildren<EnemyManager>();
      enemyManager.Init(this);
  }
  
  private void Start()
  {
      if (!isFirstLoading)
      {
          StartGame();
      }
      else
      {
          isFirstLoading = false;
      }
  }
  
  public void StartGame()
  {
      uiManager.SetPlayGame();
      StartNextWave();
  }
  
      
  private void Update()
  {
      if (Input.GetKeyDown(KeyCode.Space))
      {
          StartGame();
      }
  }
  ```

  

---

#### Game UI 만들기

:one: GameUI Anchor

- Alt 키를 누른 상태로 앵커 지정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F0a2fb261-4ec5-4888-85c3-2a2b1f9a64e3%2Fimage.png?table=block&id=1bd874c3-5d35-4526-9ecb-2e8413f41832&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=890&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fbf269151-819e-429c-9922-26481c3ce9ed%2Fimage.png?table=block&id=c353bf39-cbe0-47f2-aad8-53e5174d49f1&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1120&userId=&cache=v2)

:two: GameUI 하위 빈 오브젝트 만들기

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Feee59a51-ed0b-4c90-9ab0-856d6ca959ce%2Fimage.png?table=block&id=783614f8-8201-4f1c-8529-e95411ba840a&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=290&userId=&cache=v2) 

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fff5b51ad-2e42-4210-a445-47cafd2f4f90%2Fimage.png?table=block&id=b3dd10f4-ec39-4e67-879b-73eea459e9fb&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=930&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F3614ee41-0274-4d0e-bab3-90cf87dab8e0%2Fimage.png?table=block&id=d6e86bda-dd35-4736-8861-06a0c05e1a3e&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=930&userId=&cache=v2)

:three: Wave Background 이미지 만들기

- Hierarchy에서 Wave 오브젝트 우클릭 -> UI -> Image

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F54773c65-bc93-4863-82e0-7815cf6d3280%2Fimage.png?table=block&id=daef478a-01c2-4523-8ad2-beb659b77040&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=920&userId=&cache=v2) 

:four: Wave Text 만들기

- Hierarchy에서 Wave 오브젝트 우클릭 -> UI -> Text -> TextMeshPro

- WaveTile

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fbbd369e8-07ce-4189-90af-7582d976f993%2Fimage.png?table=block&id=10595e37-c105-44ae-bd55-eae9ff9818c4&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=920&userId=&cache=v2)	

- WaveText

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fc7ec46a6-2fef-45ba-a462-af99f827117b%2Fimage.png?table=block&id=c346446c-83bf-47cd-8f31-fdaaebd07769&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=910&userId=&cache=v2)	

:five: **Hp Bar 만들기**

- **Hierarchy**에서 HPBar 오브젝트 우클릭 → UI → Slider

- Interactable: False

- Value: 1

- Handle Slide Area 삭제

- **Alt** 키를 누른 상태로 앵커 지정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F0a2fb261-4ec5-4888-85c3-2a2b1f9a64e3%2Fimage.png?table=block&id=f9452b49-e49e-4664-841b-1dab16bf1501&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=890&userId=&cache=v2)	

- Slider/Background

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F255c24a1-4124-40a8-a57a-bde1a1c586e3%2Fimage.png?table=block&id=b31aadb1-8e2e-4501-b91c-b93f010ccbbd&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=930&userId=&cache=v2)	

- Slider/FillArea/Fill

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F6051e3fa-1811-432e-a262-1e38d46ef49f%2Fimage.png?table=block&id=f47907b5-cbb9-48ac-bdeb-aaa23d3ef703&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=920&userId=&cache=v2)	

:five: Game UI 설정

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fe2756fae-7f98-477e-9c61-64ec0bf06175%2Fimage.png?table=block&id=767672bc-cdb2-4afc-bc1f-9f37f3531b47&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1290&userId=&cache=v2)	

---

#### Game UI 로직 구현

:one: ResourceController 스크립트 수정

```csharp
private Action<float, float> OnChangeHealth;

public bool ChangeHealth(float change)
{
    if (change == 0 || timeSinceLastChange < healthChangeDelay)
    {
        return false;
    }

    timeSinceLastChange = 0f;
    CurrentHealth += change;
    CurrentHealth = CurrentHealth > MaxHealth ? MaxHealth : CurrentHealth;
    CurrentHealth = CurrentHealth < 0 ? 0 : CurrentHealth;
    
    OnChangeHealth?.Invoke(CurrentHealth, MaxHealth);
    
    if (change < 0)
    {
        animationHandler.Damage();
        
        if (damageClip)
            SoundManager.PlayClip(damageClip);
    }

    if (CurrentHealth <= 0f)
    {
        Death();
    }

    return true;
}

public void AddHealthChangeEvent(Action<float, float> action)
{
    OnChangeHealth += action;
}

public void RemoveHealthChangeEvent(Action<float, float> action)
{
    OnChangeHealth -= action;
}
```

:two: GameManager 스크립트 수정

```csharp
private void Awake()
{
    instance = this;
    player = FindObjectOfType<PlayerController>();
    player.Init(this);

    uiManager = FindObjectOfType<UIManager>();

    _playerResourceController = player.GetComponent<ResourceController>();
    _playerResourceController.RemoveHealthChangeEvent(uiManager.ChangePlayerHP);
    _playerResourceController.AddHealthChangeEvent(uiManager.ChangePlayerHP);
    
    enemyManager = GetComponentInChildren<EnemyManager>();
    enemyManager.Init(this);
}

void StartNextWave()
{
    currentWaveIndex += 1;
    uiManager.ChangeWave(currentWaveIndex);
    enemyManager.StartWave(1 + currentWaveIndex / 5);
}
```

---

#### GameOver UI 만들기

:one: GameOverUI Anchor

- Alt 키를 누른 상태로 앵커 지정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F0a2fb261-4ec5-4888-85c3-2a2b1f9a64e3%2Fimage.png?table=block&id=37b72002-5eb2-4162-b973-df82adbdb4c7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=890&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fbf269151-819e-429c-9922-26481c3ce9ed%2Fimage.png?table=block&id=d7d77c24-1916-4126-b96a-a6e4cdeb2f1c&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1120&userId=&cache=v2)

:two: 배경 이미지 생성

- Hierarchy에서 GameOverUI 오브젝트 우클릭 -> UI -> Image

- Alt 키를 누른 상태로 앵커지정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F0a2fb261-4ec5-4888-85c3-2a2b1f9a64e3%2Fimage.png?table=block&id=f98098e5-087d-45cf-890e-3350631dc2c9&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=890&userId=&cache=v2)	

:three: GameOver Text 생성

- Hierarchy에서 GameOverUI 오브젝트 우클릭 -> UI -> Text - TextMeshPro

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fc33665b1-dcbd-46bf-aa4a-a50fdb42a176%2Fimage.png?table=block&id=ac1f2651-868c-4e29-8b27-202bfb282eb7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=910&userId=&cache=v2)	

:four: **Start Button 생성**

- Hierarchy

  에서 GameOverUI 오브젝트 우클릭 → UI → Button - TextMeshPro

  - 이름: “StartButton”

    ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F34157d4b-8d31-4469-bc3d-925b42bc7428%2Fimage.png?table=block&id=5f185ba6-c668-4ba4-800f-0413b3171716&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=550&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fcf5e3a05-3b49-4cf3-bed4-07a33b4bc8a5%2Fimage.png?table=block&id=bd52440f-f1be-426e-8b53-27a1da49af98&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=570&userId=&cache=v2)

:five: **Exit Button 생성**

- Hierarchy

  에서 GameOverUI 오브젝트 우클릭 → UI → Button - TextMeshPro

  - 이름: “ExitButton”

    ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F34157d4b-8d31-4469-bc3d-925b42bc7428%2Fimage.png?table=block&id=531aa2f5-b97e-4b69-a886-f09e633f41c3&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=550&userId=&cache=v2)	

:six: 생성된 버튼들을 자유롭게 꾸며주세요.

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fcb3eb869-f6be-4967-b343-3d062b9e7965%2Fimage.png?table=block&id=fa926575-aa7d-4e12-b74e-ead385535099&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1290&userId=&cache=v2)

:seven: GameOver UI 설정

![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fe89fdd7a-0035-4fa7-abfb-e58a95bf03ae%2Fimage.png?table=block&id=570b0b4b-13bd-47c5-a274-e2466cebb617&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1290&userId=&cache=v2)



> **UI 렌더링 순서: Hierarchy와 Canvas의 관계**
>
> 1. **Canvas의 레이어 순서**
>
>    Unity의 UI는 **Canvas**를 기준으로 렌더링됩니다.
>
>    Hierarchy에서 위쪽에 있는 요소가 먼저 렌더링되며, 아래쪽에 있는 요소가 그 위에 겹쳐서 그려집니다.
>
> 2. **UI의 기본 렌더링 규칙**
>
>    - **Hierarchy 상위 요소** → **하위 요소** 순서로 그려집니다.
>    - 동일한 부모 아래에서는 **위쪽에 있는 항목**이 먼저 렌더링되고, 아래 항목이 위를 덮습니다.
>
> Hierarchy 예시
>
> ```
> Canvas
> ├── Panel A
> │   ├── Text A1
> │   └── Button A2
> ├── Panel B
> │   ├── Text B1
> │   └── Button B2
> 
> ```
>
> 1. Panel A
>
>    : 가장 먼저 렌더링됩니다.
>
>    - Text A1, Button A2가 Panel A 위에 그려집니다.
>
> 2. Panel B
>
>    : Panel A 위에 겹쳐서 렌더링됩니다.
>
>    - Text B1, Button B2는 Panel B 위에 표시됩니다.
>
> **Hierarchy 순서 변경**
>
> 1. Hierarchy에서 순서 변경
>    - **드래그 앤 드롭**으로 요소의 순서를 변경하여 렌더링 순서를 조정할 수 있습니다.
>    - 예를 들어, Panel B를 Panel A 위로 이동하면 Panel B의 요소들이 Panel A의 요소들 위에 렌더링됩니다.
> 2. Sibling Index 변경 (스크립트로 제어)
>    - UI의 렌더링 순서를 스크립트로 제어하려면 `Transform.SetSiblingIndex`를 사용합니다.
>
> ```csharp
> // Panel B를 Hierarchy의 맨 위로 이동
> GameObject panelB = GameObject.Find("Panel B");
> panelB.transform.SetSiblingIndex(0);
> 
> ```
>
> **Canvas의 추가 설정**
>
> 1. Sorting Layer
>    - Canvas 간의 렌더링 순서를 제어하려면 `Sort Order`를 설정합니다.
>    - 숫자가 높은 Canvas가 위에 그려집니다.

---

#### 인풋 시스템 패키지 설치

:one: **패키지 설치**

- 상단메뉴에서 Window → PackageManager → Unity Registry

  - Input System 검색
  - Install

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fc4985d87-6f14-45df-b64f-c00acfc44a3b%2Fimage.png?table=block&id=29dad59d-dfaf-4d40-a9f3-b451620e10c7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=410&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F33f746dc-fd4c-4d43-a2a4-14e26bf5e73b%2Fimage.png?table=block&id=0037eb48-1756-4047-ba4e-2fdc3f358c6c&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1230&userId=&cache=v2)

:two: **Input Actions 생성**

- Assets/Input

   폴더에서 Create → Input Actions

  - 이름: “Player Input Controls”

:three: **Action Map 추가**

- “+” → Player 입력

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F72e61f00-eb68-451b-96df-615d90a64343%2Fimage.png?table=block&id=86307cf5-e917-4cd2-ae1e-a693cc1eb683&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=480&userId=&cache=v2)	

:four: **Action 추가**

- Action을 추가하여 다음과 같이 이름 작성

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F71d4111e-f9a0-4b31-9653-7ae2c58235eb%2Fimage.png?table=block&id=39cb7b2c-ba86-43e3-8781-890bd3ae3036&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=840&userId=&cache=v2)	

:five: **Move 설정**

- Move Action 값 설정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F33023366-9347-479c-b2a3-8e27651ffed3%2Fimage.png?table=block&id=c95d016a-5311-4910-a71a-86b76f3bf684&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=920&userId=&cache=v2)	

- “+” → Add Up Down Left Right Composite 클릭

- No Binding 삭제

- 각 키에 맞는 입력 Path 설정

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F2dcbfdc2-60fc-43b0-935e-88491cac7d70%2Fimage.png?table=block&id=48272247-6158-4146-b537-5ec105744f31&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=1020&userId=&cache=v2)	

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F72e8a754-5bf1-4d90-a35a-e9d79c694941%2Fimage.png?table=block&id=65a65c11-c36b-419c-99e9-0a70e1331be1&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=460&userId=&cache=v2)![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2Fc00376e3-feb8-4125-a447-f78bca81b61c%2Fimage.png?table=block&id=13648f43-ca8c-4835-8d90-5e2a5d0b9ea7&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=450&userId=&cache=v2)

  :six: **Look 설정**

  - Look Action 값 설정

  - Path 설정

    ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F5f3bbced-2e88-4593-8fad-797c6580cfb9%2Fimage.png?table=block&id=70ca7394-1021-423a-b8df-d92beb13369d&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=950&userId=&cache=v2)	

  :seven:  **Fire 추가**

  - Fire Action 값 설정

  - Path 설정

    ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F9d446129-2c3a-4460-9940-0abb78a59ec6%2Fimage.png?table=block&id=b9b1223d-bdf5-4264-8d5d-b2d71b9e5126&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=920&userId=&cache=v2)	

  :eight: Save Asset 클릭

  ---

  

  > **Unity Input System**
  >
  > Unity의 **Input System**은 사용자의 입력(키보드, 마우스, 게임패드, 터치 등)을 처리하는 시스템입니다. Unity는 기존의 **Legacy Input Manager**와 **새로운 Input System Package** 두 가지 방식으로 입력을 처리할 수 있습니다.
  >
  > **Input System**
  >
  > **특징**
  >
  > - Unity의 새로운 입력 처리 방식(Package Manager에서 설치 가능).
  > - 다양한 입력 장치를 쉽게 통합 가능.
  > - 키 매핑 변경이 간단하며, 이벤트 기반 구조.
  >
  > **구조**
  >
  > 1. Input Actions Asset
  >    - 입력 처리를 설정하는 에셋.
  >    - 키보드, 마우스, 게임패드 등 여러 입력 장치를 설정 가능.
  > 2. Player Input 컴포넌트
  >    - Input Actions를 쉽게 적용할 수 있는 컴포넌트.
  >    - Unity 이벤트를 통해 입력 처리.



---



#### Player Input 적용

:one: **Player 오브젝트 Player Input 컴포넌트 추가**

- Actions: Player Input Controls

  ![img](https://teamsparta.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F83c75a39-3aba-4ba4-a792-7aefe4b07895%2F7727026e-dfeb-48f1-8f30-8a1efcc3e55a%2Fimage.png?table=block&id=757234fd-a774-4d7b-89e7-73d06a12df64&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=910&userId=&cache=v2)	

:two: PlayerController 스크립트 수정

```csharp
protected override void HandleAction()
{
    // 캐릭터의 행동 처리 (현재는 비어 있음, 필요 시 구현)
}

void OnMove(InputValue inputValue)
{
    movementDirection = inputValue.Get<Vector2>(); // 이동 입력 값 가져오기
    movementDirection = movementDirection.normalized; // 이동 방향 정규화 (속도 일정하게 유지)
}

void OnLook(InputValue inputValue)
{
    Vector2 mousePosition = inputValue.Get<Vector2>(); // 마우스 위치 가져오기
    Vector2 worldPos = camera.ScreenToWorldPoint(mousePosition); // 마우스 좌표를 월드 좌표로 변환
    lookDirection = (worldPos - (Vector2)transform.position); // 플레이어와 마우스 위치 간의 방향 벡터 계산

    if (lookDirection.magnitude < .9f) // 너무 가까우면 방향 초기화
    {
        lookDirection = Vector2.zero;
    }
    else
    {
        lookDirection = lookDirection.normalized; // 방향 벡터 정규화
    }
}

void OnFire(InputValue inputValue)
{
    if (EventSystem.current.IsPointerOverGameObject()) // UI 요소 위에서 클릭 시 무시
        return;

    isAttacking = inputValue.isPressed; // 공격 버튼 입력 상태 저장
}

```

---


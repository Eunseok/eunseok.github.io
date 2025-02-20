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
     - Size: 6 (필요에 다라 조정 가능)
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




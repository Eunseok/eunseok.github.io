---
layout: single
title: "상태 패턴(State Pattern)"
categories: [DesignPattern]
tags: [C#, StatePattern, GoF]
typora-root-url: ../
---

# 책임 연쇄 패턴 (Chain of Responsibility Pattern) - Unity 예제

## 1. 개요
책임 연쇄 패턴은 **요청을 여러 핸들러(Handler)들이 체인으로 연결되어 순차적으로 처리하는 디자인 패턴**입니다.
각 핸들러는 요청을 처리할 수 있는지 확인하고, 처리하지 못하면 다음 핸들러로 넘깁니다.

이 패턴을 **Unity 게임 개발**에서 활용하여 **입력 검증, 캐릭터 상태 검사, 행동 실행** 등을 체계적으로 관리할 수 있습니다.

---

## 2. 예제 시나리오 (점프 로직 적용)
플레이어가 점프를 시도할 때, 여러 조건을 확인해야 합니다:

1. **입력 확인** → 점프 키가 눌렸는가?
2. **스태미나 체크** → 충분한 스태미나가 있는가?
3. **지면 체크** → 공중이 아니라면 점프 가능
4. **점프 실행** → 모든 검사를 통과하면 점프

이러한 검사를 **책임 연쇄 패턴으로 구현**하여 각 조건을 독립적인 핸들러로 만들고, 순차적으로 점검할 수 있도록 합니다.

---

## 3. Unity에서 구현

### 1️⃣ 핸들러 인터페이스 (IJumpHandler)
```csharp
public interface IJumpHandler
{
    void SetNext(IJumpHandler nextHandler); // 다음 핸들러 설정
    void HandleJump(Player player); // 점프 처리
}
```

### 2️⃣ 기본 핸들러 (BaseJumpHandler)
```csharp
public abstract class BaseJumpHandler : IJumpHandler
{
    private IJumpHandler _nextHandler;

    public void SetNext(IJumpHandler nextHandler)
    {
        _nextHandler = nextHandler;
    }

    public virtual void HandleJump(Player player)
    {
        _nextHandler?.HandleJump(player);
    }
}
```

### 3️⃣ 개별 핸들러 구현

#### ✅ 입력 핸들러 (JumpInputHandler)
```csharp
public class JumpInputHandler : BaseJumpHandler
{
    public override void HandleJump(Player player)
    {
        if (!player.Input.jump)
        {
            Debug.Log("❌ 점프 키를 누르지 않았음");
            return;
        }
        Debug.Log("✅ 점프 키 입력 감지!");
        base.HandleJump(player);
    }
}
```

#### ✅ 스태미나 핸들러 (StaminaCheckHandler)
```csharp
public class StaminaCheckHandler : BaseJumpHandler
{
    public override void HandleJump(Player player)
    {
        if (player.Stamina < 10f)
        {
            Debug.Log("❌ 스태미나 부족!");
            return;
        }
        Debug.Log("✅ 스태미나 충분함!");
        base.HandleJump(player);
    }
}
```

#### ✅ 지면 감지 핸들러 (GroundCheckHandler)
```csharp
public class GroundCheckHandler : BaseJumpHandler
{
    public override void HandleJump(Player player)
    {
        if (!player.IsGrounded)
        {
            Debug.Log("❌ 공중에서 점프할 수 없음!");
            return;
        }
        Debug.Log("✅ 지면에 있음! 점프 가능!");
        base.HandleJump(player);
    }
}
```

#### ✅ 점프 실행 핸들러 (PerformJumpHandler)
```csharp
public class PerformJumpHandler : BaseJumpHandler
{
    public override void HandleJump(Player player)
    {
        Debug.Log("🚀 점프 실행!");
        player.Jump();
    }
}
```

### 4️⃣ Player 클래스
```csharp
public class Player : MonoBehaviour
{
    public InputHandler Input;
    public float Stamina = 100f;
    public bool IsGrounded = true;
    private Rigidbody _rb;

    private void Start()
    {
        _rb = GetComponent<Rigidbody>();
    }

    public void Jump()
    {
        _rb.AddForce(Vector3.up * 5f, ForceMode.Impulse);
        Stamina -= 10f;
    }
}
```

### 5️⃣ 핸들러 체인 연결
```csharp
public class JumpManager : MonoBehaviour
{
    private IJumpHandler _jumpChain;

    public void Start()
    {
        var inputHandler = new JumpInputHandler();
        var staminaHandler = new StaminaCheckHandler();
        var groundHandler = new GroundCheckHandler();
        var performJumpHandler = new PerformJumpHandler();

        inputHandler.SetNext(staminaHandler);
        staminaHandler.SetNext(groundHandler);
        groundHandler.SetNext(performJumpHandler);

        _jumpChain = inputHandler; // 체인의 시작점
    }

    public void TryJump(Player player)
    {
        _jumpChain.HandleJump(player);
    }
}
```

---

## 4. 실행 흐름

### 🎯 점프 시도 (성공)
```
✅ 점프 키 입력 감지!
✅ 스태미나 충분함!
✅ 지면에 있음! 점프 가능!
🚀 점프 실행!
```

### 🎯 스태미나 부족 (실패)
```
✅ 점프 키 입력 감지!
❌ 스태미나 부족!
```

### 🎯 공중에서 점프 시도 (실패)
```
✅ 점프 키 입력 감지!
✅ 스태미나 충분함!
❌ 공중에서 점프할 수 없음!
```

---

## 5. 책임 연쇄 패턴의 장점
✅ **SRP(단일 책임 원칙) 준수** → 각 핸들러가 하나의 책임만 가짐  
✅ **OCP(개방/폐쇄 원칙) 준수** → 새로운 점프 조건 추가 가능 (`DoubleJumpHandler` 등)  
✅ **코드 재사용성 증가** → 다른 행동에도 유사한 패턴 적용 가능  

책임 연쇄 패턴을 유니티에서 적용하면 **유연하고 확장 가능한 게임 로직을 만들 수 있습니다!** 🚀  
점프뿐만 아니라 **공격, 아이템 사용, 문 열기** 등 다양한 행동에 활용할 수 있습니다. 🔥


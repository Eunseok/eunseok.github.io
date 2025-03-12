---
layout: single
title: "전략 패턴 (Strategy Pattern)"
categories: [DesignPattern]
tags: [C#, StrategyPattern, GoF]
typora-root-url: ../
---

- # 전략 패턴 (Strategy Pattern) - Unity 예제

  ## 1. 개요
  전략 패턴(Strategy Pattern)은 **여러 알고리즘을 개별 클래스로 분리하고, 런타임에 알고리즘을 교체할 수 있도록 하는 디자인 패턴**입니다.

  이 패턴을 적용하면 특정 동작(예: 공격 방식, 이동 방식 등)을 별도의 전략 클래스로 만들고, 필요에 따라 전략을 변경할 수 있습니다.

  전략 패턴을 활용하면 **클래스 내부의 복잡한 조건문을 제거**하고, **유지보수성을 향상**시킬 수 있습니다.

  ---

  ## 2. 전략 패턴 적용 전 문제점

  전략 패턴을 적용하지 않은 경우, **if-else 또는 switch 문**을 이용하여 여러 알고리즘을 한 클래스에서 처리해야 합니다. 이는 여러 가지 문제를 유발할 수 있습니다.

  ### ❌ 1) 가독성이 낮고 유지보수가 어렵다
  다양한 동작을 한 클래스에서 관리할 경우, **if-else 문이 계속 늘어나며 코드가 복잡해집니다**. 새로운 동작을 추가할 때 기존 로직을 수정해야 하므로, 코드 변경이 어렵습니다.

  ```csharp
  public class Character : MonoBehaviour
  {
      public enum AttackType { Melee, Ranged, Magic }
      public AttackType attackType;
  
      public void Attack()
      {
          if (attackType == AttackType.Melee)
          {
              Debug.Log("근접 공격!");
          }
          else if (attackType == AttackType.Ranged)
          {
              Debug.Log("원거리 공격!");
          }
          else if (attackType == AttackType.Magic)
          {
              Debug.Log("마법 공격!");
          }
      }
  }
  ```

  위 코드에서는 공격 방식을 변경하려면 `if-else` 문을 추가해야 합니다. 이는 **확장성 부족** 문제를 야기합니다.

  ### ❌ 2) 새로운 기능 추가 시 코드 수정 필요
  새로운 공격 방식(예: `StealthAttack`)을 추가하려면 기존 코드에 새로운 조건을 추가해야 합니다. 기존 코드에 영향을 줄 가능성이 높아지고, 유지보수가 어렵습니다.

  ### ❌ 3) 코드 중복 발생 가능성
  공격 방식이 여러 캐릭터에서 사용될 경우, **비슷한 if-else 로직이 여러 클래스에서 반복될 가능성이 큽니다**. 이를 해결하기 위해 전략 패턴을 적용하면 **공격 방식을 독립적인 클래스로 분리할 수 있습니다.**

  ---

  ## 3. 예제 시나리오 (공격 방식 적용)
  플레이어 캐릭터가 다양한 공격 방식을 가질 수 있도록 구현합니다:

  1. **근접 공격 (Melee Attack)**
  2. **원거리 공격 (Ranged Attack)**
  3. **마법 공격 (Magic Attack)**

  이 공격 방식을 전략 패턴을 활용하여 관리합니다.

  ---

  ## 4. Unity에서 구현

  ### 1️⃣ 전략 인터페이스 (IAttackStrategy)
  ```csharp
  public interface IAttackStrategy
  {
      void Attack(); // 공격 실행 메서드
  }
  ```

  ### 2️⃣ 개별 전략 구현 (공격 방식 분리)

  #### ✅ 근접 공격 (Melee Attack)
  ```csharp
  public class MeleeAttack : IAttackStrategy
  {
      public void Attack()
      {
          Debug.Log("🔪 근접 공격 실행!");
      }
  }
  ```

  #### ✅ 원거리 공격 (Ranged Attack)
  ```csharp
  public class RangedAttack : IAttackStrategy
  {
      public void Attack()
      {
          Debug.Log("🏹 원거리 공격 실행!");
      }
  }
  ```

  #### ✅ 마법 공격 (Magic Attack)
  ```csharp
  public class MagicAttack : IAttackStrategy
  {
      public void Attack()
      {
          Debug.Log("✨ 마법 공격 실행!");
      }
  }
  ```

  ### 3️⃣ 플레이어 클래스 (Character)
  ```csharp
  public class Character : MonoBehaviour
  {
      private IAttackStrategy _attackStrategy;
  
      public void SetAttackStrategy(IAttackStrategy attackStrategy)
      {
          _attackStrategy = attackStrategy;
      }
  
      public void PerformAttack()
      {
          if (_attackStrategy != null)
              _attackStrategy.Attack();
          else
              Debug.Log("❌ 공격 전략이 설정되지 않음!");
      }
  }
  ```

  ### 4️⃣ 전략 변경 (CharacterManager)
  ```csharp
  public class CharacterManager : MonoBehaviour
  {
      private Character player;
  
      private void Start()
      {
          player = new Character();
  
          // 기본 공격 전략 설정 (근접 공격)
          player.SetAttackStrategy(new MeleeAttack());
      }
  
      private void Update()
      {
          if (Input.GetKeyDown(KeyCode.Alpha1))
          {
              player.SetAttackStrategy(new MeleeAttack());
              Debug.Log("⚔️ 근접 공격으로 변경!");
          }
          else if (Input.GetKeyDown(KeyCode.Alpha2))
          {
              player.SetAttackStrategy(new RangedAttack());
              Debug.Log("🏹 원거리 공격으로 변경!");
          }
          else if (Input.GetKeyDown(KeyCode.Alpha3))
          {
              player.SetAttackStrategy(new MagicAttack());
              Debug.Log("✨ 마법 공격으로 변경!");
          }
  
          if (Input.GetKeyDown(KeyCode.Space))
          {
              player.PerformAttack();
          }
      }
  }
  ```

  ---

  ## 5. 실행 흐름
  ### 🎯 실행 시 예상 로그
  ```
  ⚔️ 근접 공격으로 변경!
  🔪 근접 공격 실행! (스페이스바 누름)
  🏹 원거리 공격으로 변경! (1 → 2 키 입력)
  🏹 원거리 공격 실행! (스페이스바 누름)
  ✨ 마법 공격으로 변경! (2 → 3 키 입력)
  ✨ 마법 공격 실행! (스페이스바 누름)
  ```

  ---

  ## 6. 전략 패턴의 장점
  ✅ **SRP(단일 책임 원칙) 준수** → 각 전략이 하나의 역할만 가짐  
  ✅ **OCP(개방/폐쇄 원칙) 준수** → 새로운 공격 방식 추가 가능 (`StealthAttack` 등)  
  ✅ **코드 재사용성 증가** → 여러 개의 캐릭터에 적용 가능  
  ✅ **가독성 향상** → 전략별 로직이 분리되어 유지보수 용이  

  전략 패턴을 적용하면 **공격 방식을 독립적으로 관리할 수 있으며, 새로운 방식 추가도 쉽습니다!**

  이제 **캐릭터의 공격 방식을 런타임에 쉽게 변경할 수 있고, 코드가 훨씬 유연해집니다!** 🔥

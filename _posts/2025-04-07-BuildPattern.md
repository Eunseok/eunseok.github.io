---
layout: single
title: "언제 Builder 패턴을 써야 할까?"
categories: [DesignPattern]
tags: [C#, BuildParttern, Fluetn Builder, GoF]
typora-root-url: ../
---

## 코드 스멜

```csharp
public class Enemy : MonoBehaviour
{
    public string Name { get; private set; }
    public int Health { get; private set; }
    public float Speed { get; private set; }
    public int Damage { get; private set; }
    public bool IsBoss { get; private set; }

    public Enemy(string name)
    {
        Name = name;
    }
    
    public Enemy(string name, int health) : this(name)
    {
        Health = health;
    }

    public Enemy(string name, int health, float speed) : this(name, health)
    {
        Speed = speed;
    }

    public Enemy(string name, int health, float speed, int damage) : this(name, health, speed)
    {
        Damage = damage;
    }

    public Enemy(string name, int health, float speed, int damage, bool isBoss) : this(name, health, speed, damage)
    {
        IsBoss = isBoss;
    }
    
    // Do Enemy Stuff...
}
```



이 Enemy클래스에는 2가지 코드 스멜이 존재한다.

1. **너무 많은 생성자 파라미터**
   - 생성자에 4개 이상의 파라미터가 있다면 -> Builder 고려
   - 가독성과 유지보수에 안 좋음
2. **텔레스코핑 생성자 (Telescoping Constructor)**
   - 생성자 오버로딩으로 선택적 매개변수를 하나씩 추가
   - 점점 복작해지는 생성자 체인 -> 유지보수가 어려워짐

> **텔레스코핑 생성자란?**
>
> 같은 클래스 내에서 **매개변수를 점점 늘려가며 생성자를 오버로딩(overloading)** 하는 패턴을 말합니다.

## Fluent Builder 사용

위 예제를 Fluent Builder를 사용하여 리팩토링하는 과정을 살펴보자

1. 먼저 Enemy Builder 클래스를 작성한다.

```csharp
public class EnemyBuilder
{
    string name;
    int health;
    float speed;
    int damage;
    bool isBoss;

    public EnemyBuilder WithName(string name)
    {
        this.name = name;
        return this;
    }

    public EnemyBuilder WithHealth(int health)
    {
        this.health = health;
        return this;
    }

    public EnemyBuilder WithSpeed(float speed)
    {
        this.speed = speed;
        return this;
    }
    
    public EnemyBuilder WithDamage(int damage)
    {
        this.damage = damage;
        return this;
        
    }

    public EnemyBuilder WithIsBoss(bool isBoss)
    {
        this.isBoss = isBoss;
        return this;
    }
    
    public Enemy Build()
    {
       return new Enemy(name, health, speed, damage, isBoss);
    }
}
```

EnemyBuilder 클래스를 작성함으로써 메서드 체이닝`(SetName().SetHealth()...)` 으로 각 속성을 설정할수 있게 되었다.

그러나 이코드에는 문제가 있다.

```csharp
public Enemy Build()
{
    return new Enemy(name, health, speed, damage, isBoss);
}
```

Enemy는 Monobehavior라 new로 생성이 불가하며 GameObject 생성후 Enemy 컴포넌트 추가해야 한다.

```csharp
public Enemy Build()
{
    var enemy = new GameObject("Enemy").AddComponent<Enemy>();
    enemy.Name = name;
    enemy.Health = health;
    enemy.Speed = speed;
    enemy.Damage = damage;
    enemy.IsBoss = isBoss;
    return enemy;
}
```

문제가 해결된것 처럼 보이지만 사실은 enemy에 각 속성들이 private setter를 가졌기 때문에 위방식은 컴파일 에러를 발생시킬 것이다. 

그렇다면 각속성의 setter를 public 으로 바꿔서 캡슐화를 포기해야할까? 답은 그렇지 않다. 캡슐화를 지키며 이문제를 해결하는 방법은 아래에 나와있다. 

2. EnemyBuilder를 Enemy 내부 클래스로 작성

먼저 Enemy Builder를 Builder로 이름을 변경후 Enemy클래스 내부로 이동시킨다.

```csharp
public class Enemy : MonoBehaviour
{
    public string Name { get; private set; }
    public int Health { get; private set; }
    public float Speed { get; private set; }
    public int Damage { get; private set; }
    public bool IsBoss { get; private set; }

    //기존 생성자는 필요가 없어졌으니 삭제한다.

    // Do Enemy Stuff...

    public class Builder
    {
        string name = "Enemy";
        int health = 100;
        float speed = 5f;
        int damage = 10;
        bool isBoss = false;

        public Builder WithName(string name)
        {
            this.name = name;
            return this;
        }

        public Builder WithHealth(int health)
        {
            this.health = health;
            return this;
        }

        public Builder WithSpeed(float speed)
        {
            this.speed = speed;
            return this;
        }

        public Builder WithDamage(int damage)
        {
            this.damage = damage;
            return this;

        }

        public Builder WithIsBoss(bool isBoss)
        {
            this.isBoss = isBoss;
            return this;
        }

        public Enemy Build()
        {
            var enemy = new GameObject("Enemy").AddComponent<Enemy>();
            enemy.Name = name;
            enemy.Health = health;
            enemy.Speed = speed;
            enemy.Damage = damage;
            enemy.IsBoss = isBoss;
            return enemy;
        }
    }
}
```

이렇게 하면 private set을 유지하면서 builder패턴을 사용하여 객체를 생성할수 있게 되었다. 

#### 빌더사용 객체 생성 예제

```csharp
public class GameManager : MonoBehaviour
{        
    void Start()
    {
        Enemy enemy = new Enemy.Builder()
            .WithName("Goblin")
            .WithHealth(100)
            .WithSpeed(5f)
            .WithDamage(10)
            .Build();

        Instantiate(enemy);
        // or put in object pool, etc.
    }
}
```

이런식으로 속성별로 값을 설정 할수 있으며 가독성과 필요한 속성만 작성하면 되니 코드길이도 짧아졌다. (위코드에선 `WithIsBoss`를 사용하지 않았다)

이렇게 생성된 객체를 `Instatiate`로 바로 게임오브젝트로 생성하거나 오브젝트풀에 넣어서 사용이 가능하다.

## Fluent Builder vs 일반 Builder

그렇다면 Fluent Builder 대신 일반 Builder는 언제 사용해야 할까?

일반 Builder는 보통 Director 클래스와 함께 사용한다.

```csharp
public class EnemyDirector
{
    EnemyBuilder builder;
    public EnemyDirector(EnemyBuilder builder)
    {
        this.builder = builder;
    }

    public Enemy Construct(EnemyData data)
    {

        builder.AddWeaponComponent();
        builder.AddWeaponStrategy(data.weaponStrategy);
        builder.AddHealthComponent();

        return builder.Build();
    }
}

public class EnemyBuilder
{
    Enemy enemy = new GameObject("Enemy").AddComponent<Enemy>();

    public void AddWeaponComponent()
    {
        enemy.gameObject.AddComponent<EnemyWeapon>();
    }

    public void AddWeaponStrategy(WeaponStrategy strategy)
    {
        if (enemy.gameObject.TryGetComponent<EnemyWeapon>(out var weapon))
        {
            weapon.SetWeaponStrategy(strategy);
        }
    }

    public void AddHealthComponent()
    {
        enemy.gameObject.AddComponent<Health>();
    }

    public Enemy Build()
    {
        Enemy builtEnemy = enemy;
        enemy = new GameObject("Enemy").AddComponent<Enemy>();

        return builtEnemy;
    }

```

```csharp
public class GameManager : MonoBehaviour
{
    [SerializeField] EnemyData enemyData;

    EnemyDirector enemyDirector;

    void Start()
    {
        enemyDirector = new EnemyDirector(new EnemyBuilder());
        var enemy = enemyDirector.Construct(enemyData);

        Instantiate(enemy);
        // or put in object pool, etc.
    }
}
```

구조로는 EnemyBuilder와 EnemyDirector로 되어있고

EnemyBuider는 GameObject 생성 및 컴포넌트 추가를 담당하며

EnemyDirector는 Builderf를 통해 순서대로 구성한다.

이점으론 Builder 재사용이 가능하다는 것과 EnemyDirector를 캐싱해서 효율성이 향상 되었다는 것이다.

정리하자면 다음과 같다.

### **Fluent Builder vs 일반 Builder**

| Fluent Builder | 일반 Builder          |
| -------------- | --------------------- |
| 메서드 체이닝  | 순차적 메서드 호출    |
| 유연함         | 순서를 강제할 수 있음 |
| 가독성 ↑       | 복잡한 로직 제어 가능 |

**💡 일반 Builder는 언제?**

- **특정 순서**로 메서드를 호출해야 하는 경우
- 컴포넌트 생성 → 구성 순서가 중요할 때
- 보통 **Director 클래스**와 함께 사용



## (번외) 인터페이스 기반 Builder

```csharp
public interface IEnemyBuilder
{
    IWeaponEnemyBuilder AddWeaponComponent();
}
public interface IWeaponEnemyBuilder
{
    IHealthEnemyBuilder AddWeaponStrategy(WeaponStrategy strategy);
}
public interface IHealthEnemyBuilder
{
    IFinalEnemyBuilder AddHealthComponent();
}
public interface IFinalEnemyBuilder
{
    Enemy Build();
}

public class EnemyDirector
{
    IEnemyBuilder builder;
    public EnemyDirector(IEnemyBuilder builder)
    {
        this.builder = builder;
    }
    public Enemy Construct(EnemyData data)
    {
        return builder
            .AddWeaponComponent()
            .AddWeaponStrategy(data.weaponStrategy)
            .AddHealthComponent()
            .Build();
    }
}
public class EnemyBuilder : IEnemyBuilder, IWeaponEnemyBuilder, IHealthEnemyBuilder, IFinalEnemyBuilder
{
    Enemy enemy = new GameObject("Enemy").AddComponent<Enemy>();

    public IWeaponEnemyBuilder AddWeaponComponent()
    {
        enemy.gameObject.AddComponent<EnemyWeapon>();
        return this;
    }

    public IHealthEnemyBuilder AddWeaponStrategy(WeaponStrategy strategy)
    {
        if (enemy.gameObject.TryGetComponent<EnemyWeapon>(out var weapon))
        {
            weapon.SetWeaponStrategy(strategy);
        }
        return this;
    }

    public IFinalEnemyBuilder AddHealthComponent()
    {
        enemy.gameObject.AddComponent<Health>();
        return this;
    }

    public Enemy Build()
    {
        Enemy builtEnemy = enemy;
        enemy = new GameObject("Enemy").AddComponent<Enemy>();

        return builtEnemy;
    }

}
```

인터페이스를 기반으로한 Builder 예제 이다.

각 인터페이스는 다음으로 호출되어 하는 인터페이스의 반환값을 가진 메서드 하나만 가지고 있고 정해진 순서대로 단계를 지키지 않으면 빌드가 불가능 하기때문에 다른 Builder 클래스나 Director를 만들때 강제성을 부여할 수 있다.

단점은 지금은 인터페이스가 4개뿐이라 괜찮지만 인터페이스가 더 많아진다면 복잡 해질 우려가 있다.

----

### **✅ 결론**

- Builder는 복잡한 객체 생성에서 큰 장점
- 생성자 파라미터가 많거나, 설정 옵션이 복잡하거나, 구성 순서를 강제해야 할 때 적합
- Unity와 같은 프레임워크에서 GameObject 설정에 특히 유용
- **Fluent Builder**: 가독성 중심
- **일반 Builder + Director**: 제어 중심
- **인터페이스 Builder**: 완전한 순서 강제 

---

### **🧠 팁**

- 디펜던시 인젝션(DI)처럼 활용 가능 (전략 컴포넌트 주입 등)
- 기본값 세팅도 Builder에서 관리하면 깔끔함

---
layout: single
title: "언제 팩토리패턴과 추상팩토리 패턴을 써야할까?"
categories: [DesignPattern]
tags: [C#, BuildParttern, Fluetn Builder, GoF]
typora-root-url: ../
---

## Static Factory Mesthod

```csharp
public class NetworkPlayer
{
    public string Name { get; private set; }
    public int Health { get; private set; }

    public NetworkPlayer(string name, int health)
    {
        Name = name;
        Health = health;
        // 코드를 보지 않는 한, 생성자에서 생성뿐만 아닌 다른 일이 일어나고 있다는것을 전현 알 수 없을 것이다.
        // 생성자가 단순 객체 생성만 안 하고 추가 로직 포함 시 → 코드 스멜
        GameManager.RegisterPlayer(this);       
    }


}
```

위 코드는 생성자에서 객체 생성 뿐만 아니라 GameManager에 플레이어를 등록하는 게임 로직 또한 포함하고 있다. 

이 코드를 보지 않는한 생성자에서 생성뿐만 아니라 다른 일이 일어 나고 있다는 것을 전혀 알 수 없을 것이다.

#### 해결: 정적 팩토리 메서드 생성

```csharp
public class NetworkPlayer
{
    public string Name { get; private set; }
    public int Health { get; private set; }

    // 생성자를 private 으로 설정하여 NetworkPlayer를 만드는 유일한 방법은 CreateAndRegister()를 호출하는 것
    NetworkPlayer(string name, int health)
    {
        Name = name;
        Health = health; 
    }

    // 해결 생성 정적 팩토리 메서드 생성 
    // 생성 & 초기화 과정을 메서드에 명시적 이름 부여
    public static NetworkPlayer CreateAndRegister(string name, int health)
    {
        NetworkPlayer player = new NetworkPlayer(name, health);
        GameManager.RegisterPlayer(player);
        return player;
    }
}
```

이를 정적 팩토리 메서드로 추출하여 생성과등록`CreateAndRegister`을 명시적으로 메서드 명으로 표시하여 해결 내부 코드를 보지 않고도 무슨일이 일어나는지 알 수있게 하였다.

또한 생성자를 private으로 변경해 `NetworkPlayer`를 생성하는 유일한 방법은 정적 메서드를 호출하는것 뿐이다.

> **Tip**
>
> private 생성자를 발견하면 대게 어딘가에 정적 생성 메서드가 존재 할 수 있고
>
> 이를 사용해야 할 것이다.

---

## Factory 패턴

#### 코드 스멜

```csharp
public class Knight : MonoBehaviour
{
    [SerializeField] string weaponType = "Sword";
    IWeapon weapon;

    void Start()
    {
        // Switch OR if/else 문으로 객체 생성 (결합도 증가)
        // SRP(단일 책임 원칙 위반)-Knight가 무기 생성까지 책임짐
        // 문자열 기반 무기 타입 -> 오타위험
        switch (weaponType)
        {
            case "Sword":
                weapon = new Sword();
                break;
            case "Bow":
                weapon = new Bow();
                break;
        }
    }

    public void Attack() => weapon?.Attack();
}
```

위 코드는 조금 극단적으로 표현한 코드 스멜이 나타나는 Knight 클래스 이다.

여기엔 3가지 코드 스멜이 있다.

1. SWITH OR IF/ELSE 문으로 객체 생성
   - 만약 `Bow`를 `LongBow`와 `ShortBow`로 나눈다면 `Knight` 뿐만 아니라관련 있는 모든 클래스(`Archor, Soldier`)등 모든 스크립트를 수정해야 할것이다.
2. SRP(단일 책임 원칙) 
   - Knight가 무기 생성 까지 책임지므로 단일 책임 원칙에 어긋난다.
3. 문자열 기반 무기 타입 
   - 오타 위험이 존재한다.

#### 해결

````csharp
public abstract class WeaponFactory 
{
    public abstract IWeapon CreateWeapon();
}

public class BowFactory : WeaponFactory
{
    public override IWeapon CreateWeapon()
    {
        return new Bow();
    }
}

public class SwordFactory : WeaponFactory
{
    public override IWeapon CreateWeapon()
    {
        return new Sword();
    }
}
````

```csharp
WeaponFactory weaponFactory; // 문자열을 제거하고 무기 팩토리를 참조
IWeapon weapon = IWeapon.CreateDefault();

void Start()
{
    weaponFactory = new SwordFactory();
    if(weaponFactory != null)
        weapon = weaponFactory.CreateWeapon();

    Attack();
}

public void Attack() => weapon?.Attack();
}
```

팩토리 패턴에서 팩토리의 책임은 제품을 소비자에게 전달하는 것이다. 따라서 이경우엔

`Knight`가 소비자이면 전달할 제품은 `IWeapon`이다.

각 무기별 전용 Factory 클래스를 작성하여 무기생성책임을 Factory에게 위임하여  많은 부분이 해결 되었지만 여전히 Knight가 Factory 생성을 하고있다.

이를 해결하기 위해선 여러 방법이 있지만 가장 간단한 방법은 팩토리 클래스를 ScriptableObject로 변경하는 것이다.

#### ScriptableObject로 변경

```csharp
public abstract class WeaponFactory : ScriptableObject
{
    public abstract IWeapon CreateWeapon();
}
```

```csharp
[CreateAssetMenu(fileName = "SwordFactory", menuName = "Weapon Factory/Sword")]
public class SwordFactory : WeaponFactory
{
    public override IWeapon CreateWeapon()
    {
        return new Sword();
    }
}
```

```csharp
[CreateAssetMenu(fileName = "BowFactory", menuName = "Weapon Factory/Bow")]
public class BowFactory : WeaponFactory
{
    public override IWeapon CreateWeapon()
    {
        return new Bow();
    }
}
```

```csharp
public class Knight : MonoBehaviour
{
    [SerializeField] WeaponFactory weaponFactory;
    IWeapon weapon;

    void Start()
    {
        weapon = weaponFactory.CreateWeapon();

        Attack();
    }

    public void Attack() => weapon?.Attack();
}
```

비로서 Knight는 모든 생성 책임에서 벗어날 수 있게 되었다.

그러나 여기에는 한가지 우려되는 상황이 있는데 **게임 디자이너가 `Knight`프리팹에 Factory를 할당하는것을 잊어버린 경우**다. 이경우 8번 라인에서 `NullReference` 오류가 발생할 것이다.  이를 방지하기 위해 예외처리를 해보자.

```csharp
public class Knight : MonoBehaviour
{       
    [SerializeField] WeaponFactory weaponFactory;
    IWeapon weapon = new Weapon();

    void Start()
    {
        if(weaponFactory != null)
            weapon = weaponFactory.CreateWeapon();

        Attack();
    }

    public void Attack() => weapon?.Attack();
}
```

게임 디자이너가 Factory를 할당하지 않았을 경우를 대비해 `null`체크를 하고 `weaponFactory`가 `null`일 경우 기본 무기를 `Weapon`으로 지정하였다.

그러나 다시 또 `new Wepon()`을`Knight`에서 처리하므로 처음과 같은 문제가 생겨 벼렸다. 이를 해결 하기위에선 `IWeapon` 인터페이스에 처음 설명했던 **정적 팩토리 메서드**를 추가하는 것이다.

```csharp
public interface IWeapon
{
    void Attack();
    static IWeapon CreateDefault()
    {
        return new Sword();
    }
}
```

```csharp
public class Knight : MonoBehaviour
{
    [SerializeField] WeaponFactory weaponFactory;
    IWeapon weapon = IWeapon.CreateDefault();

    void Start()
    {
        if(weaponFactory != null)
            weapon = weaponFactory.CreateWeapon();

        Attack();
    }

    public void Attack() => weapon?.Attack();
}
```

---

## 추상 팩토리(Abstract Factory)

> 추상 팩토리 패턴은 서로 관련되거나 의존적인 객체들의 **집합(family)**를 생성한다.
>
> **When? 여러 개의 팩토리 또는 서비스를 묶어서 사용해야 할 때**

우리 게임에 무기 뿐만 아니라 방패도 추가되었고 `Soldier`는 무기와 방패를 전부 사용한다 가정해보자. 가장 먼저 방패와 방패 팩토리를 만들어야 할 것이다.

```csharp
public interface IShield
{
    void Defend();
    static IShield CreateDefault()
    {
        return new Shield();
    }
}

public class Shield : IShield
{
    public void Defend()
    {
        Debug.Log("막기");
    }
}
```

```csharp
public abstract class ShieldFactory : ScriptableObject
{
    public abstract IShield CreateShield();
}
```

```csharp
[CreateAssetMenu(menuName = "Shield Factory/Generic", fileName = "GenericShieldFactory")]
public class GenericShieldFactory : ShieldFactory
{
    public override IShield CreateShield()
    {
        return new Shield();
    }
}
```

다음은 새롭게 추가된 방패를 사용할 유닛을 작성해보자.

```csharp
public class Soldier : MonoBehaviour
{
    [SerializeField] WeaponFactory weaponFactory;
    [SerializeField] ShieldFactory shieldFactory;

    IWeapon weapon;
    IShield shield;

    void Start()
    {
        weapon = weaponFactory.CreateWeapon();
        shield = shieldFactory.CreateShield();
    }

    public void Attack() => weapon?.Attack();
    public void Defend() => shield?.Defend();
}
```

`Knigt`와는 다르게 2가지의 팩토리를 가진다. 지금은 전혀 문제가 없어 보이지만 새로운 팩토리가 추가될때마다 프리팹에 해당 팩토리를 넣어줘야 할것이고 프리팹에 갯수가 많아진다면 어떨까? 이를 해결하게 위해 추상 팩토리 `Equipment Factory`를 만들어보자

``` csharp
[CreateAssetMenu(menuName = "Equipment Factory", fileName = "EquipmentFactory")]
public class EquipmentFactory : ScriptableObject
{
    public WeaponFactory weaponFactory;
    public ShieldFactory shieldFactory;

    public IWeapon CreateWeapon()
    {
        return weaponFactory != null ? weaponFactory.CreateWeapon() : IWeapon.CreateDefault();

    }

    public IShield CreateShield()
    {
        return shieldFactory != null ? shieldFactory.CreateShield() : IShield.CreateDefault();
    }
}
```

```csharp
public class Soldier : MonoBehaviour
{
    [SerializeField] EquipmentFactory equipmentFactory;

    IWeapon weapon;
    IShield shield;

    void Start()
    {
        weapon = equipmentFactory.CreateWeapon();
        shield = equipmentFactory.CreateShield();
    }

    public void Attack() => weapon?.Attack();
    public void Defend() => shield?.Defend();
}
```

이제 `Solider`는 하나의 팩토리만 가지고 이 팩토리는 관려있는 팩토리들의 **집합**이다.

#### (번외) 추가 리팩토링

`wepon` 과 `shield`를 캐싱할 필요가 없다면 조금 더 극단적으로 코드를 줄 일 수 있다.

```csharp
public class Soldier : MonoBehaviour
{
    [SerializeField] EquipmentFactory equipmentFactory;

    public void Attack() => equipmentFactory?.CreateWeapon.Attack();
    public void Defend() => equipmentFactory?.CreateShield.Defend();
}
```

이때 Create 관련 메서드에서 매번 객채를 생성하지 않고 한번만 생성하고 캐싱해두도록 바꾸고, 메서드명 또한 `ProvideWeapon`으로 명시적이게 바꾸자

```csharp
[CreateAssetMenu(menuName = "Shield Factory/Generic", fileName = "GenericShieldFactory")]
public class GenericShieldFactory : ShieldFactory
{
    IShield shield;
    public override IShield ProvideWeapon()
    {
        return shield ??= new Shield();
    }
}
```

```csharp
[CreateAssetMenu(fileName = "SwordFactory", menuName = "Weapon Factory/Sword")]
public class SwordFactory : WeaponFactory
{
    IWeapon weapon;
    public override IWeapon ProvideWeapon()
    {
       return weapon ??= new Sword();
    }
}
```

```csharp

public class Soldier : MonoBehaviour
{
    [SerializeField] EquipmentFactory equipmentFactory;

    public void Attack() => equipmentFactory?.ProvideWeapon.Attack();
    public void Defend() => equipmentFactory?.ProvideWeapon.Defend();
}
```

---

## **✅ 결론**

| 패턴                  | 핵심 기능                    | 사용 시기                                   |
| --------------------- | ---------------------------- | ------------------------------------------- |
| Static Factory Method | 생성자 대신 객체 + 로직 생성 | 로직이 섞인 생성자가 있을 때                |
| Factory               | 구체 타입 분리               | SRP위반,Switch,if/else문,타입 변경이 많을때 |
| Abstract Factory      | 관련 Factory나 Service 묶음  | 여러 요소(무기+방패) 같이 생성시            |

---

## **📌 요약**

- Factory 패턴은 **의존성 역전**과 **단일 책임 원칙** 실현에 탁월
- 코드 유연성 향상, 신규 무기/장비 도입 쉬움
- Unity에서는 **ScriptableObject + Inspector 설정** 조합이 강력함
- **‼️주의: 팩토리가 제공하는 제품은 꼭 게임 객체가 아니여도 된다**. 

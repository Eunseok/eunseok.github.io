---
layout: single
title: "오픈소스 UGS사용하기"
categories: [DesignPattern]
tags: [C#, GoogleSpreadsheet,Unity]
typora-root-url: ../
---

## Git URL을 통해 프로젝트에 패키지 설치 하기

`https://github.com/shlifedev/uni-google-sheets?path=src/`

## Apps Scripts Setup

1. [AppsScript](https://script.google.com/home/projects/1Hn0u-_Wg9mEADDcmFpvMfMn-wLyOqlx_3b5wHBbJpBkyEH89hI32shfr/edit) 접속 

2. 사본만들기

3.  (설정)스크립트 속성 추가

   | password | 사용할 비밀번호 입력 |
   | -------- | -------------------- |
   | read     | true                 |
   | write    | true                 |

   >  각 속성은 시트에 접근하기 위한 비밀번호, 시트 읽기 쓰기에 허용 여부입니다.  비밀번호는 원하는것을특입력하시고  나머지는 특별한 경우가 아닌이상 `true`

#### 유니티 설정 (비밀번호)

HamsterLib->UGS->Setting 탭의 Password에 설정한 값을 입력합니다.



## 스크립트 배포과정

- 스크립트 우측 상단에서 `새 배포` 클릭 -> 액세스 승인

> 웹앱, 나(email), 모든사용자 항목이 일치하는지 확인

- Advanced -> Go to Copy of UnityGoogleSheet Released (unsafe) -> Allow
-  배포된 URL 복사 -> UnityGoogleSheet 세팅에 추가



## GoogleDrive Setup

- [GoogleDrive](https://drive.google.com/drive/my-drive) 이동 -> 새폴더 생성 -> URL에서 folders/뒤에 폴더 ID복사

- UGS -> Stting 에서 구글폴더 ID항목에 붙여넣기

- 폴더엔에 폴더 2개를 만들어 잘만들어 졌는지 확인

- UGS -> Manager 오류없이 열리면 성공



## 시트 생성 및 데이터 만들기

- UGS -> Manager로 매니저 윈도우 열기
- Create Default Sheet 번트을 눌러 기본 테이블 샘플을 만들고 더블클릭
- Assts/UGS.Generated 경로에 폴더와 json,c#파일이 생기면 성공

## 로컬에서 불러오기

- 빈 스크립트 생성

```csharp
void Awake()
{
    UnityGoogleSheet.LoadAllData(); 
    // UnityGoogleSheet.Load<DefaultTable.Data.Load>(); it's same!
    // or call DefaultTable.Data.Load(); it's same!
}
    
void Start()
{ 
    foreach (var value in DefaultTable.Data.DataList)
    {
        Debug.Log(value.index + "," + value.intValue + "," + value.strValue);
    } 
    var dataFromMap = DefaultTable.Data.DataMap[0];
    Debug.Log("dataFromMap : " + dataFromMap.index + ", " + dataFromMap.intValue + "," + dataFromMap.strValue);
}
```

> Log가 출력되면 성공



## 시트에서 불러오기(LiveLoad)

> 이 기능은 인반 릴리즈에 포함시켜서는 안됨. 구글 api는 호출 제한횟수와 보안문제 때문

HamsterLib->UGS->Manager 에서 TestSheet 생성->데이터 Generate

아래 코드 입력후 구글 시트의 데이터의 데이터를 읽는지 확인

```csharp
    void Start()
    {
        UnityGoogleSheet.LoadFromGoogle<int, TestSheet.Data>((list, map) => {
            list.ForEach(x => {
                Debug.Log(x.intValue);
            });
        }, true); 

    }
```



## 시트에 데이터 쓰기(LiveWirte)

> 게임 보안상 문제가 될 수 있으므로 아래의 과정을 거쳐야 이 기능을 사용할 수 있음

- 권환을 허용하기 위해 Apps Script로 이동한 후 File -> Project Properties -> Script Properties에 wirte 속성을 추가

  write 속성을 추가하고 값을 true로 설정

  password 속성을 추가하고 비밀번호을 지정

아래의 코드를 작성하여 테스트

```csharp
 void Start()
    {
        var newData = new TestSheet.Data();
        newData.index = 200;
        newData.intValue = 300;
        newData.strValue = "test";

        UnityGoogleSheet.Write<TestSheet.Data>(newData);
    }
```



## Enum 사용방법

enum에 UGS 어트리뷰트를 붙여주기

```csharp
    [UGS(typeof(TestType))]
    public enum TestType
    {
        A, B, C
    }
```





## Generate 대상 제외

컬럼이나 시트 이름 맨 앞에 #을 붙이면 Generate 대상에서 제외

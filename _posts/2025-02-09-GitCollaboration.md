---
layout: single
title: "Git 협업 가이드"
categories: [TextRPG]
tags: [C#]
typora-root-url: ../

---

## 1) 새로운 작업 시작 준비

**1. 해당 리포지터리 상단탭 > Projects > 해당 프로젝트 이동**

![image-20250209180714557](/Images/20205-02-09-GitCollaboration//image-20250209180714557.png)

**2. 해당  아이템 이름 클릭**

![image-20250209181742990](/Images/20205-02-09-GitCollaboration//image-20250209181742990.png)

**3. Assignees (배정자)에 본인 계정 등록 > Convert to Issue > 리포지토리 선택**

![image-20250209200528591](/Images/20205-02-09-GitCollaboration//image-20250209200528591.png)

**4. Create a branch > Branch name 작성 > Branch source `develop` 지정** 

![image-20250209182643981](/Images/20205-02-09-GitCollaboration//image-20250209182643981.png)     

![image-20250209183142249](/Images/20205-02-09-GitCollaboration//image-20250209183142249.png)

---

## 2) 작업 진행

**1. Todo > In Progress 드래그 이동**

![image-20250209185225759](/Images/20205-02-09-GitCollaboration//image-20250209185225759.png)

**2. branch 이동 > 작업 시작**

![image-20250209184146651](/Images/20205-02-09-GitCollaboration//image-20250209184146651.png)

---

## 3) 작업 완료 (Pull Request, PR)

**1. `develop` branch 이동 > Fetch origin(최신화)**

![image-20250209185856520](/Images/20205-02-09-GitCollaboration//image-20250209185856520.png)

**2. 작업중이던 Branch 이동 > `develop` branch 랑 Merge****

![image-20250209191337670](/Images/20205-02-09-GitCollaboration//image-20250209191337670.png)

**3. Push > Changes > Preview Pull Request**

![image-20250209191852390](/Images/20205-02-09-GitCollaboration//image-20250209191852390.png)

**4. base : develop > Create Pull Request**

![image-20250209192554675](/Images/20205-02-09-GitCollaboration//image-20250209192554675.png)

**5. 제목 및 설명 작성 > reviewer 등록> Create Pull Request**

![image-20250209193204390](/Images/20205-02-09-GitCollaboration//image-20250209193204390.png)

**6. 리뷰 완료후 > merge**

- Before Review

![image-20250209193617116](/Images/20205-02-09-GitCollaboration//image-20250209193617116.png)

- Affter Reivew

  ![image-20250209193848644](/Images/20205-02-09-GitCollaboration//image-20250209193848644.png)

---

**4) Review 등록**

**1. 상단탭 Pull Request > Files Changed**

![image-20250209194414465](/Images/20205-02-09-GitCollaboration//image-20250209194414465.png)

**2. (선택사항)라인 옆 (+) Comment 달기**

![image-20250209194759132](/Images/20205-02-09-GitCollaboration//image-20250209194759132.png)

**3. Review Changes or Finish your reivew**

   ![image-20250209195148122](/Images/20205-02-09-GitCollaboration//image-20250209195148122.png)

- **Comment** : 댓글만 달기 **(승인/거부 없음)**
- **Approve** :  Feedback 제출 및 동의 **(승인)**
- **Request Changes** : 수정 요청 **(거절)**
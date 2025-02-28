---
layout: single
title: "유니티 입문(개인)_트러블슈팅"
categories: [트러블슈팅]
tags: [C#, Troubleshooting]
typora-root-url: ../
---

> 요구하신 트러블슈팅과는 다를 수 있지만, 이번 과제를 진행하면서 부딪쳤던 문제와 해결 과정을 정리하려 합니다.



## 1️⃣ WebGL + Firebase Realtime Database 시도



처음에는 **ZEP과 같은 웹 기반 플랫폼을 목표로 하여, Firebase Realtime Database를 활용한 WebGL 게임을 배포**하려 했습니다.

이를 통해 별도의 서버 없이 Firebase를 활용하여 멀티플레이 데이터를 관리하고자 했습니다.



**❌ 문제점:**

1. **Firebase Unity SDK가 WebGL 환경에서 동작하지 않음**
   - WebGL에서는 Firebase Unity SDK를 직접 사용할 수 없었고,
   - 기존에 작성한 Firebase SDK 기반 코드들을 **REST API 방식으로 변경해야 하는 상황 발생**	
   - 이를 위해 RestClient를 활용한 방식으로 데이터를 처리하도록 코드를 대폭 수정

2. **WebGL 환경에서 OnDestroy() 콜백 함수가 정상적으로 동작하지 않음**
   - 유니티 WebGL에서는 OnDestroy()가 **브라우저에서 강제 종료되거나 탭이 닫힐 때 실행되지 않음**
   - 이를 해결하려면 **JavaScript에서 직접 window.onbeforeunload 등의 이벤트를 처리해야 하지만, 익숙하지 않아 개발이 지연됨**



🔥 **결론:**

WebGL에서 Firebase를 활용한 멀티플레이 게임을 구현하는 것은

**기술적 제약이 많고 개발 속도가 더뎌질 것이라 판단하여, 타겟 플랫폼을 데스크톱으로 변경하기로 결정**

---

#### 📌 **데스크톱 전환 및 아키텍처 구성**

WebGL에서 데스크톱으로 전환하면서 **아키텍처를 새롭게 구성**했습니다.

​	•	**사용자 인증:** Firebase Authentication (익명 로그인)

​	•	**채팅 및 데이터 저장:** Firebase Realtime Database

​	•	**멀티플레이어 기능:** Photon Fusion



이 방식은 Firebase를 활용하여 **유저 인증과 데이터 저장을 처리하고, 멀티플레이어 기능은 Photon Fusion을 활용**하는 형태로 설계되었습니다.

---

## 2️⃣ Photon Fusion 도입 후 발생한 문제



Firebase 기반 **사용자 인증과 각 사용자별 마지막 위치 저장 기능을 구현한 후, Fusion을 활용한 멀티플레이 기능을 개발**하였습니다.



**❌ 문제점:**

- **모든 플레이어가 같은 NetworkObject를 공유하여 캐릭터의 조작권을 잃음**

  Photon Fusion의 NetworkRunner.Spawn()이 올바르게 작동하지 않아,

  **각 플레이어가 개별적인 NetworkObject를 가지지 못하고 하나의 오브젝트를 공유하는 문제 발생**    

  결과적으로 **한 플레이어가 나가면 모든 캐릭터가 삭제되는 현상까지 발생**

- **Fusion에 대한 문서 및 정보 부족**

  Fusion을 처음 다루다 보니 **문제 해결을 위한 공식 문서나 사례가 부족하여 구글링에 어려움**

​	반면 **같은 Photon사의 PUN2는 비교적 많은 정보와 튜토리얼이 존재**(Fusion도 존재하지만 너무 어려웠습니다.)

🔥 **결론:**

Fusion의 문제를 해결하는 데 시간이 오래 걸릴 것으로 판단하고,

**더 많은 자료를 찾을 수 있는 PUN2를 활용하여 과제를 최종 제출**하기로 결정

---

## 3️⃣ 최종 결론 및 요약

최종적으로 **PUN2(Photon Unity Networking 2)를 활용한 데스크톱 게임을 제작 및 제출**하였습니다.



| 단계  |           시도한 기술            |                            결과                             |
| :---: | :------------------------------: | :---------------------------------------------------------: |
| 1단계 |         WebGL + Firebase         | WebGL 환경에서 Firebase SDK 미지원 및 OnDestroy 문제로 실패 |
| 2단계 |        Fusion + Firebase         |     Fusion의 네트워크 오브젝트 생성 문제로 어려움 발생      |
| 3단계 | PUN2 (Photon Unity Networking 2) |   최종적으로 데스크톱 환경의 결과물을 제작하여 과제 제출    |



**📌 트러블슈팅 요약**

1. Firebase 기반 WebGL 멀티플레이를 시도했으나, WebGL 환경에서 Firebase SDK 미지원 및 콜백 함수 문제로 실패
2. Fusion을 도입하여 개선하려 했으나, 모든 플레이어가 같은 오브젝트를 공유하는 문제 발생
3. PUN2로 전환하여 최종적으로 안정적인 멀티플레이 기능을 구현하고 과제 제출
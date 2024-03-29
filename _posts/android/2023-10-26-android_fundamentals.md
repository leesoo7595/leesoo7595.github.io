---
layout: post
title: "[Android] 애플리케이션 기본 항목"
date: 2023-10-26
tags: [Android]
comments: true
---

이 글은 구글 안드로이드 공식문서를 읽고 정리한 글 입니다.

# 애플리케이션 기본항목

Android SDK는 모든 데이터 및 리소스 파일과 함께 코드를 컴파일하여 하나의 APK를 만든다. 안드로이드 패키지는 `.apk`인 아카이브 파일이다. 이 안에는 안드로이드 앱의 모든 콘텐츠가 들어있고, Android로 구동하는 기기가 앱을 설치할 때 이 파일을 사용한다.

각 안드로이드 앱은 다음과 같은 자체적인 보안기능으로 보호된다.

- 안드로이드 운영체제는 멀티유저 리눅스 시스템으로, 각 앱은 다른 각각의 사용자와 같다.
- 시스템은 각 앱에 고유한 리눅스 id를 할당하고 그 id만 앱에 엑세스하도록 한다. 앱에서는 이 리눅스 id를 인식하지 못한다. 시스템은 앱안의 모든 파일에 대해 권한을 설정하여 해당 앱에 할당된 id만 엑세스할 수 있도록 한다.
- 각 프로세스는 vm이 있고, 그래서 한 앱의 코드가 다른 앱과는 격리된 상태로 실행된다.
- 모든 앱은 앱 자체의 리눅스 프로세스에서 실행된다. 안드로이드 시스템은 앱의 구성요소중 어느것이 실행되도 프로세스를 새로 시작하고, 더 이상 필요없는 경우 해당 프로세스를 종료한다.

안드로이드 시스템은 이런 방식으로 **최소 권한의 원리**를 구현한다. 각 앱은 기본적으로 필요한 구성요소에만 엑세스 권한을 가지고 그 이상은 허용되지 않는다. 그럼으로써 각 앱은 완전히 격리된다.

그러나 앱이 다른 앱과 데이터를 공유하고 시스템 서비스에 엑세스하는 방식 또한 여러가지가 있다.

- 두 개의 앱이 같은 리눅스 id를 공유하도록 설정할 수 있다. 그렇게하면 두 앱은 서로 파일에 엑세스할 수 있게 된다. 더해서 같은 리눅스 프로세스에서 실행되고, vm을 공유하도록 설정할 수도 있다. 이러한 앱은 같은 인증서로 서명하여야한다.
- 앱은 여러가지 기기 데이터에 액세스할 권한을 요청할 수 있다. 사용자는 권한을 명시적으로 부여해야한다.

## 앱 구성요소

안드로이드 앱의 필수적인 기본 구성요소는 시스템이나 사용자가 앱에 들어올 수 있는 진입점이다.

- Activity
- Service
- Broadcast Receiver
- Contents Provider

### Activity

액티비티는 사용자와 상호작용하기 위한 entry이다. 액티비티는 사용자 인터페이스를 포함한 화면 하나를 나타낸다. 여러 액티비티가 함께 작동하여 환경 구성을 할 수 있지만, 각자 독립되어 있다. 이걸 활용하여 앱에서 허용하는 경우 다른 앱이 이 앱의 액티비티 중 하나를 시작할 수 있다. 즉, 액티비티는 다음과 같이 시스템과 앱의 주요 상호작용을 돕는다.

- 사용자가 화면에 표시된 것을 추적해서 액티비티를 호스팅하는 프로세스를 시스템에서 계속 실행하도록 한다.
- 이전에 사용한 프로세스에 사용자가 다시 찾을만한 액티비티가 있다는 것을 인지하고, 해당 프로세스를 유지하는데 더 높은 우선순위를 부여한다.
- 앱이 프로세스를 종료하게되는 경우 이전 상태가 복원되는 동시에 사용자가 액티비티로 돌아갈 수 있게한다.
- 앱이 사용자 플로우를 구현하고 시스템이 이러한 플로우를 조정하기 위한 수단을 제공한다.

하나의 액티비티는 `Activity` 클래스의 하위 클래스로 구현한다.

### Service

서비스는 여러이유로 백그라운드에서 앱을 계속 실행하기 위한 다목적 entry이다. 서비스는 백그라운드에서 실행되는 구성요소로 오랫동안 실행되는 작업을 수행하거나, 원격 프로세스를 위한 작업을 수행한다. 사용자 인터페이스는 제공하지 않는다.

예를 들면 사용자가 다른 앱에 있는 동안 백그라운드에서 음악을 재생하거나, 사용자와 액티비티 간의 상호작용을 차단하지 않고 네트워크를 통해 데이터를 가져올 수도 있다. 다른 구성요소(액티비티 같은)가 서비스를 시작한 다음 실행되도록 두거나 자신에게 바인딩하여 상호작용할 수 있도록 한다.

서비스는 작업이 완료될 때까지 해당 서비스를 계속 실행하도록 지시할 수 있다. 백그라운드에서 일부 데이터를 동기화하거나, 앱에서 나간 후에도 음악을 재생하는 것은 두 가지 타입의 다른 시작된 서비스이다.

- 음악 재생의 경우 사용자가 바로 인식할 수 있는 작업이다. 이 시스템은 이 서비스의 프로세스가 계속 실행되도록 하여야한다.
- 정기적인 백그라운드 서비스는 사용자가 실행되고 있다고 인식할 수 없는 작업이므로, 시스템은 자유롭게 프로세스 관리가 가능하다. 좀 더 사용자와 직접적인 작업에 RAM이 필요한 경우 이 서비스를 종료할 수도 있다. (그리고 나중에 다시 시작)

바인딩된 서비스는 다른 앱에서 서비스를 사용하길 원할 때 실행된다. 이는 서비스가 다른 프로세스에 API를 제공하는 것이다. 시스템은 프로세스 사이에 종속성이 있는지 알 수 있다. 예를 들면 프로세스 A가 프로세스 B의 서비스에 종속되어있는 경우, 프로세스 A를 실행시키기 위해 B도 실행해야한다는 것을 인식한다.

서비스는 유연해서 각종 시스템 개념의 유용한 기본 구성요소로 사용되었다. 라이브 배경화면, 알림 리스너, 화면 보호기, 입력 메서드, 접근성 서비스 등 여러가지 기타 서비스 기능이 시스템에서 앱을 실행할 때 바인딩하는 서비스로 빌드 된다.

서비스는 `Service` 클래스의 하위 클래스로 구현된다.

### Broadcast Receiver

시스템이 정기적인 사용자 플로우 밖에서 이벤트를 앱에 전달하도록 지원하는 구성요소이다. 이 아이 또한 앱으로 들어갈 수 있는 또 다른 entry이기 때문에 실행되지 않은 앱에도 시스템이 브로드캐스트를 전달할 수 있다.

예를 들어 예정된 이벤트 알림을 예약알람으로 설정할 때, 그 알람을 앱의 broadcast receiver에 전달하면 알림이 울릴 때까지 앱을 실행하지 않아도 된다. 대부분의 브로드캐스트는 시스템에서 발생한다. 앱도 브로드캐스트를 실행할 수 있다. 주로 다른 구성요소의 gateway인 경우가 많고, 매우 적은 작업만 수행한다.

브로드캐스트 리시버는 `BroadcastReceiver`의 하위 클래스로 구현되고 Intent 객체로 전달된다.

### Contents Provider

콘텐츠 제공자는 웹상이나 앱이 액세스할 수 있는 영구저장 위치에 저장가능한 앱데이터의 공유형 집합을 관리한다. 앱은 콘텐츠 제공자를 통해 해당 데이터를 쿼리하거나, 허용하는 경우 수정할 수도 있다. 예를 들면, 사용자의 연락처 정보를 관리는 컨텐츠 제공자를 안드로이드 시스템에서 제공한다.

콘텐츠 제공자를 데이터베이스의 추상화된 무언가라고 인지하기 쉬운데, 이는 시스템 설계 관점에서 목적이 다르다. 시스템의 경우 콘텐츠 제공자는 URI 구성표로 식별되고 이름이 지정된 데이터 항목을 게시할 목적으로 앱에 진입하기 위한 entry이다. 이 URI를 통해 데이터에 엑세스할 수 있다.

- URI를 소유한 앱이 종료된 후에도 URI를 유지할 수 있다.
- 이 URI를 통해 중요한 보안 모델이 제공된다. 앱은 이미지에 URI를 할당하고 콘텐츠 제공자가 검색하도록 하여 다른 앱이 자유롭게 이미지에 엑세스하지 못하게 막을 수 있다.

콘텐츠 제공자는 앱 전용이어서 공유되지 않는 데이터를 읽고 쓰는데도 유용하다. `ContentProvider` 하위 클래스로 구현된다.

### 구성요소 활성화

구성요소 유형 중 액티비티, 서비스, 브로드캐스트 리시버는 인텐트라는 **비동기식 메시지**로 활성화된다. 인텐트는 런타임에서 각 구성요소를 서로 바인딩한다. 구성요소가 어느 앱에 속하든 관계없이 다른 구성요소로부터 작업을 요청하는 역할을 한다.

인텐트는 `Intent` 객체로 생성되고, 어떤 구성요소 유형을 활성화할지 나타내는 메세지를 정의한다.

액티비티와 서비스의 경우, 인텐트는 수행할 작업을 정의하며 작업을 수행할 데이터의 URI를 지정할 수 있다. 브로드캐스트 리시버의 경우, 인텐트는 단순히 브로드캐스트될 알림을 정의한다.

반면 콘텐츠 제공자는 인텐트로 활성화되지 않는다. `ContentResolver`가 보낸 요청의 대상으로 지정되면 활성화된다. 콘텐츠 확인자는 제공자와의 모든 트랜잭션을 처리하여 ContentResolver 객체에서 메서드를 호출하게 한다. 이런식으로 하면 콘텐츠 제공자와 정보를 요청하는 구성요소 사이에 보안 목적의 추상화 계층이 남아있게 된다.

## [안드로이드 - 애플리케이션 기본 항목](https://developer.android.com/guide/components/fundamentals?hl=ko#Components)

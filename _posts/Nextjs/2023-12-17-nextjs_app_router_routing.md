---
layout: post
title: "[Nextjs] App Router - Routing"
date: 2023-12-17
tags: [nextjs]
comments: true
---

# 라우팅 기초

모든 애플리케이션의 기초는 라우팅이다. 이 페이지에서는 웹 라우팅의 기본 개념과 Nextjs의 라우팅의 처리하는 방법을 소개한다.

## 용어

첫번째로, 문서 전체에서는 이러한 용어가 사용된다:

<img width="531" alt="스크린샷 2023-12-17 오후 11 53 19" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/ad6c173d-29f7-433f-9dd4-fa8a007d82cc">

- Tree : 계층구조를 시각화한 컨벤션. 예를 들면, 부모와 자식 컴포넌트들을 가지고 있는 컴포넌트 트리, 폴더 구조 등을 이야기한다.
- Subtree : 트리의 부분, 새 루트에서 시작하여 마지막 잎사귀에서 끝나는 트리의 일부를 의미한다.
- Root : 트리나 서브트리의 첫번째 노드 (root 레이아웃 같은 것을 의미)
- Leaf : 자식이 없는 서브트리의 노트. (URL 경로의 마지막 세그먼트 등)

<img width="399" alt="스크린샷 2023-12-17 오후 11 53 49" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/0320227c-1650-4af4-8c04-992dd303221b">

- URL Segment: 슬래시로 구분된 Url 경로의 일부
- URL Path : 도메인 뒤에 오는 URL의 일부 (세그먼트로 구성된)

## app Router

Nextjs 13에서는 App Router라는 새로운 개념의 React 서버 컴포넌트를 기반한 것을 소개한다. 이것은 Layout을 공유할 수 있고, 네스팅된 라우팅, 로딩 상태, 에러핸들링 등등을 지원한다.

App Router는 app이라는 새로운 디렉토리 이름으로 작동한다. app 디렉토리는 page 디렉토리와 함께 작동하여 점진적인 적용이 가능하다. 이렇게하면 애플리케이션의 일부 행동을 새 동작으로 선택하면서 다른 행동을 이전 동작의 페이지 디렉터리에 유지할 수 있다. pages 디렉토리를 사용하고 있다면 Pages Router 도큐먼트를 보아라.

App 라우터가 page 라우터보다 우선된다. 디렉토리 간 경로는 동일한 URL 경로로 리졸브할 경우 빌드오류의 원인이 된다.

+++
title = "Flutter 학습 노트"
date = 2026-01-25
description = "Flutter 학습 과정에서 정리한 핵심 개념과 실습 내용"

[extra]
page_style = "note"
+++

## Flutter란?

Google에서 개발한 크로스 플랫폼 UI 프레임워크. 단일 코드베이스로 iOS, Android, Web, Desktop 앱을 개발할 수 있다.

## 핵심 개념

### Widget

Flutter의 모든 것은 Widget이다. UI를 구성하는 기본 빌딩 블록.

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Text('Hello Flutter'),
    );
  }
}
```

### State Management

- **StatelessWidget**: 상태가 없는 정적인 위젯
- **StatefulWidget**: 상태를 가지고 변경 가능한 위젯

### Widget Tree

위젯들의 계층 구조. React의 Component Tree와 유사.

## 실습 프로젝트

### 1. Counter App

기본 템플릿으로 제공되는 카운터 앱. State 관리의 기초를 이해.

**핵심 코드:**
```dart
setState(() {
  _counter++;
});
```

### 2. Todo App

CRUD 기능을 구현하며 다음을 학습:
- ListView를 활용한 리스트 표시
- TextField로 사용자 입력 받기
- State 관리로 동적 UI 업데이트

### 3. 레이아웃 실습

- **Row/Column**: 수평/수직 배치
- **Container**: 패딩, 마진, 장식 적용
- **Expanded/Flexible**: 유연한 레이아웃

## React와의 비교

| Flutter | React |
|---------|-------|
| Widget | Component |
| build() | render() |
| setState() | useState() |
| StatefulWidget | Class Component |
| StatelessWidget | Functional Component |

## 장단점

### 장점
- Hot Reload로 빠른 개발
- 네이티브 성능
- 풍부한 위젯 라이브러리
- 일관된 UI (플랫폼 독립적)

### 단점
- Dart 언어 학습 필요
- 앱 크기가 큼
- 플랫폼별 네이티브 기능 제한

## 유용한 리소스

- [Flutter 공식 문서](https://docs.flutter.dev/)
- [Flutter Layout Cheat Sheet](https://medium.com/flutter-community/flutter-layout-cheat-sheet-5363348d037e)
- [Dart 언어 투어](https://dart.dev/guides/language/language-tour)

## 다음 학습 목표

1. Provider를 활용한 상태 관리
2. HTTP 통신과 JSON 파싱
3. Navigation과 라우팅
4. 애니메이션 구현
5. 플랫폼별 네이티브 기능 연동
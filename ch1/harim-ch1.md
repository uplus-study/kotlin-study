# 1. 코틀린이란 무엇이며 왜 필요한가 

> **코틀린**: 자바 플랫폼에서 돌아가는 프로그래밍 언어


### 주요 특성
- 코틀린 목적: 현재 자바가 사용되고 있는 모든 용도에 적합하면서 간결, 생산적, 안전한 대체 언어
- 정적타입 지정 언어: 구성 요소 타입을 컴파일 시점에 알 수 있고 컴파일러가 타입 검증해줌
  - 장점: 성능, 신뢰성, 유지보수성, 도구 지원
- 타입 추론: 모든 변수 타입 명시할 필요 없이 컴파일러가 자동 유추
  - 널이 될 수 있는 타입 지원 -> 컴파일 시점에 널 확인을 통해 신뢰성 높임
- 함수형 프로그래밍
  - 일급 시민인 함수: 함수를 값처럼 다룸, 인자로 전달
  - 불변성: 내부 상태 바뀌지 않는 불변 객체 사용
  - 부수효과 없음: 입력이 같다면 같은 출력을 내놓음
  - 장점: 간결성, 다중스레드 안전성(불변 데이터 구조 여러 스레드가 변경할 수 없음), 테스트 쉬움

### 응용
- 코틀린 서버 프로그래밍
  - 자바와 상호운용 가능
  - 빌더 패턴 -> 간결
- 코틀린 안드로이드 프로그래밍
  - 다양한 디바이스에 대해 서비스 신뢰성 보장, 빠른 개발 및 배포
  - 안코 라이브러리: 안드로이드 API에 대한 어댑터 제공
  - 코틀린을 통해 신뢰성 높아짐 
    - 주로 애플리케이션에서 처리되지 않는 예외로 인해 프로세스 중단이 되는 경우가 많지만 코틀린에서는 자바의 NPE를 일으키는 코드가 컴파일부터 막힘
  - 성능
    - 코틀린 컴파일러가 생성한 바이트코드는 자바 코드와 똑같이 실행됨
    인라이닝?

### 철학
- 실용성
  - 특정 프로그래밍 스타일이나 패러다임 사용 강제하지 않음
  - 도구 강조 (IDE)
- 간결성 
  - getter, setter, 생성자 파라미터 필드 대입 로직 등 묵시적 제공
  - 다양한 표준 라이브러리를 제공하여 반복되거나 길어지는 코드 대체 가능
- 안전성
  - 일부 유형의 오류를 프로그램 설계가 원칙적으로 방지
  - JVM 에서 실행 -> 메모리 안전성 보장, 버퍼 오버플로 방지 등
  - 컴파일 시점 검사를 통해 오류 더 많이 방지함 (NPE)
- 상호운용성
  - 자바와 상호운용성, 프로젝트에서 섞어 사용 가능

### 도구

**코틀린 코드 컴파일**

코틀린 컴파일러가 코틀린 소스코드 분석해서 .kt -> .class 파일 만듦
-> .jar -> 애플리케이션

코틀린으로 컴파일한 코드는 코틀린 런타임 라이브러리에 의존하는데 애플리케이션 배포 시 라이브러리도 함께 배포 필요
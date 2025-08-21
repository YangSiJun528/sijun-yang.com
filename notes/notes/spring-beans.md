# 스프링 빈과 컨테이너 잡소리

- 스프링 목표: 엔터프라이즈 프레임워크 개발
- 스프링 사상: POJO로 개발하자!

- 스프링 빈: 스프링에서 관리하는 객체.
    - BeanDefinition 으로 관리된다.
- 스프링 컨테이너: 스프링 빈을 관리하는 컨테이너. 빈 간의 의존성을 설정한다.
    - 빈 팩토리가 컨테이너 기능, 실재로 사용 시에는 ~ApplicationContext라는 기능이 더 추가된 걸 쓴다.
- 스프링 컨테이너는 여러 개 존재할 수 있다.
    - 부모, 자식 관계로 생성한다. 자식은 여럿이 될 수 있고, 자식끼리는 의존이 없다.
    - 웹 관련은 서블릿 컨테이너, 나머지는 루트 컨테이너에서 관리하는 식.
    - Spring Initializr는 요청별로 임시 컨텍스트를 생성하여 특정 작업 수행 후 폐기한다.
        - ProjectGenerationInvoker#createProjectGenerator 호출 시 외부에서 Context를 전달하는게 아니면 매 요청 시마다 ProjectGenerationContext를 만든다.
        - ProjectGenerator#defaultContextFactory 참고
        - 각 프로젝트 생성 요청이 서로 영향을 주지 않도록 완전히 격리한다. (설정이나 컨트리뷰터(Gradle, Java, Kotlin에 Build, Code 추가 같은거)가 요청마다 다를 수 있어서)
        - 실제로 이 해당 경우가 얼마나 있을지는 모르겠음. Spring Initializr에서나 봄.
        - 어쨌든 여러 컨테이너 사용 예시긴 하니까
- 스프링 부트 기준 시작 시 발생하는 일
    - 스프링이 여러 어노테이션 붙은걸 보고, 컴포넌트 스캔을 수행하여 빈이나 구성 정보를 읽는다. 
    - 빈으로 등록(스프링 컨테이너에 담는다)하면서 의존성을 적절하게 주입해준다.

---

- 그래서 뭐가 POJO한가?
    - 빈으로 정의된 객체는 스프링 컨테이너가 의존성을 관리해줄 뿐, 
    - 객체 자체를 스프링이 아닌 곳으로 가져다 쓰더라도 문제가 되지 않는다. 
        - (스프링 라이브러리 기능에 의존하지 않는 한 문제 없다. 어노테이션 방식 말고 다른 객체 의존적인 방식을 지양하는 이유기도 함.)
    - 즉, 어떤 프레임워크에 의존해야만 동작하는 코드가 아니므로 여전히 POJO하다.

---

- ClassPathBeanDefinitionScanner 가 하는 일
    - 컴포넌트 스캔을 하는데, .class(바이트코드) 파일을 읽어서 적절한 것만 BeanDefinition에 등록해둔다.
    - 실제로 쓸 때 클래스 로드가 발생하여 효율적으로 동작할 수 있게 해준다.

---

- 내부 코드 설명하는 글
    - [[Spring] ApplicationContext refresh 과정 | Blog](https://pplenty.tistory.com/6)
    - [Instantiating the Spring Container by Using AnnotationConfigApplicationContext| Spring Framework Docs](https://docs.spring.io/spring-framework/reference/core/beans/java/instantiating-container.html#beans-java-instantiating-container-scan)
        - 우리가 쓰는 방식은 이렇게 동작한다.

---

- 그래서 스프링 컨테이너(DI) 장점이 뭐임?
    - 변경/추가가 쉬움. 
        - 외부 구성만 변경하면 기존 코드를 변경하지 않고도 신규 코드를 추가하거나 기존 기능의 의존성을 대체할 수 있음.
    - 테스트가 쉬움.
        - 위 특징으로 인해서 복잡한 기존 의존성이나, 테스트를 위해서 특정 기능이 필요한 경우, 테스트용으로 쉽게 대체할 수 있음.
        - 이건 스프링 내부의 테스트코드를 보면 테스트 객체에서 컨텍스트를 주입받는 경우가 있는데, 쉽게 의존성을 추가/대체할 수 있다.
            - DI의 장점이 보고 싶으면 이게 더 도움이 될 수도? (이걸 DI없이 구현하려면 코드가 어떻게 될지 고민해보자)
            - https://github.com/spring-io/initializr/blob/de8d39cc02f5673fa4935fd0ca60aed764b755ff/initializr-generator-spring/src/test/java/io/spring/initializr/generator/spring/properties/ApplicationPropertiesComplianceTests.java << `applicationPropertiesWithCustomProperties()`의 동작과 `AbstractProjectGenerationTester` 클래스를 보면 어떤 식으로 되는지 알 듯?
    - 런타임 시점에 동작하는거라 setter를 사용해서 빈이 사용하는 의존성을 바꿀 수도 있음.
        - 별로 추천하는 동작은 아닌거 같지만.
        - 실재로 내부적으론 null 일수도 있는거, 외부 구성에서 추가하고 null check해서 특정 동작 수행 같은 식으로 쓰는거 같음.

---

- 스프링 컨테이너가 꼭 필요한가? (런타임 시점 DI)
    - Rust로 만들어진 Axum은 컴파일 DI, Jetbrains에서 만드는 Ktor은 기본적으로 DI가 없고 추가해서 씀.
    - 컴파일 시점에 되면 런타임 시점에 스캔이 필요없고, 불필요한건 지울 수 있으니까 좋을거 같은데
    - 테스트 쉽게 하는것도, 외부 구성의 장점이지 런타임 시점 DI의 장점이 아닌 거 같음.
    - setter도 복잡한 구성이 필요한 내부 코드에서나 쓴다는 걸 보면 컴파일 시점으로 대체 가능하므로 런타임 시점의 장점이 크게 없어보임.

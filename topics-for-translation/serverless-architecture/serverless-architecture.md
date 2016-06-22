# 서버리스 아키텍처 #
serverless-architectures

---
2016년 6월 17일

![](http://martinfowler.com/articles/serverless/mike.jpg)

[마이크 로버츠 Mike Roberts](https://twitter.com/mikebroberts)
마이크는 뉴욕에 사는 엔지니어링 리더이다. 요즘엔 팀 매니지먼트가 주요 업무이긴 하지만 여전히 클로주어 Clojure 쪽에서 코딩도 하고 소프트웨어 아키텍처 쪽에서도 활발한 의견 개진을 하고 있다. 그는 지금 사람들이 서버리스 아키텍처에 대해 주목하는 현상에 대해 꽤 긍정적이다.

아래 태그들을 통해 **비슷한 문서들**을 찾을 수 있다:
[application architecture](http://martinfowler.com/tags/application%20architecture.html)

목차

* [서버리스란 무엇인가?](#what-is-serverless)
  * 몇가지 예제
    * UI 주도 애플리케이션
    * 메시지 주도 애플리케이션
  * `Function as a Service` 뒤집어보기
    * 상태
    * 실행 기간
    * 초기 실행 지연
    * API 게이트웨어
    * 도구들
    * 오픈소스
  * 서버리스가 아닌 것은?
    * PaaS와 비교
    * #NoOps
    * Stored Procedures as a Service

---

`서버리스`는 요즘 소프트웨어 아키텍처 세상에서는 아주 핫한 토픽입니다. [책들도](https://leanpub.com/serverless) [나왔고](https://www.amazon.com/gp/product/1680501496?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=1680501496), [오픈소스 프레임워크도 있고](https://github.com/serverless/serverless), [수많은](https://aws.amazon.com/lambda/) [벤더들이](https://cloud.google.com/functions/docs/) [프레임워크를](https://azure.microsoft.com/en-us/services/functions/) [내놨죠](http://www.ibm.com/cloud-computing/bluemix/openwhisk/). 게다가 아예 `서버리스`만을 주제로 하는 [컨퍼런스](http://serverlessconf.io/)까지 생겼습니다. 그런데, 도대체 `서버리스`가 뭘까요? 그리고 어째서 이 `서버리스`를 고려해야 (혹은 고려하지 말아야) 할까요? 이 [계속 업데이트 되는 문서](http://martinfowler.com/bliki/EvolvingPublication.html)를 통해 저는 당신이 이러한 질문들에 대한 답을 구할 수 있는 빛을 찾기를 바랍니다.


<a name="what-is-serverless"></a>
## 서버리스란 무엇인가? ##

소프트웨어 업계에서 늘상 그렇듯이, `서버리스`에 대한 명확한 관점은 없습니다. 그리고 아래 두 가지의 다르지만 겹치는 부분이 있는 이러한 견해들 역시도요:

1. `서버리스`는 서버단 로직이나 상태 등을 관리하기 위한 써드파티 애플리케이션 혹은 클라우드 서비스에 현저히 또는 온전히 의존하는 애플리케이션들을 설명하기 위해 쓰였습니다. 주로 `리치 클라이언트` 애플리케이션(예를 들자면 단일 페이지 웹 애플리케이션이나 모바일 앱 같은 것들)을 가리키는데, 클라우드에서 접근 가능한 `Parse`나 `Firebase` 같은 데이터베이스라든가, `Auth0`, `AWS Cognito` 같은 인증 서비스들 같은 거대한 생태계를 사용하는 것들입니다. 예전에는 이러한 서비스들을 [(Mobile) Backend as a Service](https://en.wikipedia.org/wiki/Mobile_backend_as_a_service)라고 불렀으니, 여기에서는 이들을 그냥 `BaaS`라고 부르도록 하죠.
2. 또한 `서버리스`는 개발자들이 서버단 로직을 개발자들이 짜긴 하지만, 전통적인 아키텍처와는 달리 상태를 저장하지 않는 Stateless 컴퓨팅 컨테이너에 넣고 돌리는 애플리케이션을 의미하기도 합니다. 이러한 애플리케이션은 보통 이벤트 기반으로 작동하고, 한 번 쓰고 버리고, 써드파티에 의해 관리되죠(ThoughWorks는 최근 [자사 포스트](https://www.thoughtworks.com/radar/techniques/serverless-architecture)에서 이렇게 정의했습니다). 이런 방식으로 생각해 볼 수 있는 한가지 방법은 `[Functions as a Service](https://twitter.com/marak/status/736357543598002176) (or FaaS)`입니다. [AWS 람다](https://aws.amazon.com/lambda/)는 현재 이 FaaS계의 가장 인기있는 구현체지요. 하지만 다른 것들도 더 있습니다. 여기서는 바로 이 `FaaS`를 `서버리스`의 의미로 사용하도록 하겠습니다.


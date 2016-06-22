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
  * [몇가지 예제](#a-couple-of-examples)
    * [UI 주도 애플리케이션](#ui-driven-applications)
    * [메시지 주도 애플리케이션](#message-driven-applications)
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

저는 주로 두 번째 얘기를 할텐데요, 조금 더 새롭기도 하고 우리가 흔히 기술적인 아키텍처에 대해 생각하는 것과 현격한 차이가 있기도 합니다. 게다가 요즘 `서버리스`라는 것에 대한 수많은 얘기들이 오고가기 때문이기도 하구요.

하지만, 이러한 개념들이 사실은 모두 관련이 있고 하나로 모여들고 있습니다. [`Auth0`](https://auth0.com/)가 하나의 좋은 예가 될 수 있겠네요. 처음에 BaaS 형태인 `Authentication as a Service`로 시작했다가 지금은 [Auth0 Webtask](https://webtask.io/)를 통해 FaaS 영역으로 들어왔습니다.

게다가 `BaaS 형태의` 애플리케이션을 개발하는 많은 경우, 특히 모바일 앱과 반대로 `리치` 웹 앱을 개발하는 경우, 어느 정도 서버단의 커스텀 기능들이 여전히 필요합니다. 특히 당신이 사용하고 있는 BaaS 서비스와 어느 정도 통합을 한다면 FaaS 가 이런 경우 좋은 솔루션이 될 수 있습니다. 이러한 기능들의 좋은 예로는 데이터 유효성 검사(악성 클라이언트로부터 보호하기 위한)라든가 많은 계산 용량을 필요로 하는 작업들(이미지나 비디오 프로세싱 같은 것들)이 있겠지요.


<a name="a-couple-of-examples"></a>
### 몇 가지 예제 ###

<a name="ui-driven-applications"></a>
#### UI 주도 애플리케이션 ####

서버단에 로직이 있는 전통적인 쓰리티어 클라이언트 시스템을 봅시다. 좋은 예로는 전자상거래 시스템들이 있겠네요. 온라인 애완동물 용품 사이트 같은거?

전통적으로 이런 아키텍처는 이런 식으로 생겼습니다. 서버단에 자바로 구현했고, 클라이언트단에는 HTML과 자바스크립트로 구현하죠.

![](http://martinfowler.com/articles/serverless/ps.svg)

이런 아키텍처에서 클라이언트는 상대적으로 그닥 똑똑하지 않습니다. 대부분의 로직들 &ndash; 인증, 페이지 네비게이션, 검색, 트랜잭션 등은 서버단에서 구현을 해놨으니까요.

`서버리스` 아키텍처에서는 이렇게 보일 겁니다:

![](http://martinfowler.com/articles/serverless/sps.svg)

엄청나게 간단하게 그린 모델인데요, 그럼에도 불구하고 여전히 수많은 변화들이 일어난 것을 볼 수 있습니다. 여기서 잠깐! 이건 단순히 `서버리스` 개념을 보여주기 위한 도구로서 만든 그림이지 이게 이런 식으로 아키텍처를 이전해야 한다고 추천하는 건 아니라는 것을 기억해 두세요!



<a name="message-driven-applications"></a>
#### 메시지 주도 애플리케이션 ####

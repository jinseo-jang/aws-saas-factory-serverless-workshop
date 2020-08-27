# Lab 2 – Onboarding, Identity, and a Modern Client

Now that we have a fully deployed and functional monolith, it's time to start looking at what it will take to move this monolith to a multi-tenant, modern architecture. The first step in that process is to introduce a way to have tenants onboard to your system. This means moving away from the simple login mechanism we had with our monolith and introducing a way for tenants to follow an automated process that will allow us to sign-up and register as many tenants as we want. As part of this process, we'll also be introducing a mechanism for managing tenant identities. Finally, we'll also create a new client experience that extracts the client from our server-side web/app tier and moves it to S3 as a modern React application.

이제 완전히 배포가능 하고 기능을 갖춘 모놀리스 애플리케이션을 만들었으므로 이제 이 모놀리스를 멀티 테넌트, 현대식 아키텍처로 옮기기 위해 무엇이 필요한지 살펴볼 차례입니다. 첫 번째 단계는 테넌트가 여러분의 시스템에 온보딩 하는 방법을 도입 하는 것입니다. 이것은 우리가 모놀리스 애플리케이션에서 가지고 있던 간단한 로그인 메커니즘에서 벗어나 테넌트가 자동화 된 프로세스를 따라 원하는 방식으로 테넌트들이 가입하고 등록 할 수있는 방법을 도입하는 하는 것입니다. 이 프로세스의 일부로 테넌트 ID를 관리하기 위한 메커니즘도 도입 할 예정입니다. 마지막으로, 서버 측 webapp 계층에서 클라이언트 부분을 분리해 최신 React 클라이언트 애플리케이션 만들어 S3로 옮기는, 새로운 클라이언트 환경도 만들 것 입니다.

It's important to note that this workshop does not dive deep into the specifics of onboarding and identity. These topics could consume an entire workshop and we recommend that you leverage our other content on these topics to fill in the details. The same is true for the React client. Generally, there aren't many multi-tenant migration considerations that change your approach to building and hosting a React application on AWS. There are plenty of examples and resources that cover that topic. Our focus for this workshop is more on the microservices pieces of the migration and the strategies we'll employ with our onboarding automation to allow us to cutover gracefully from single-tenant to multi-tenant microservices.

이 워크샵은 온보딩 및 indenity 세부 사항에 대해 자세히 다루지 않습니다. 관련한 내용은 다른 컨텐츠를 살펴 보시기를 추천 드립니다. React 클라이언트도 마찬가지입니다. 일반적으로 AWS에서 React 애플리케이션을 구축하고 호스팅하는 방식을 변경할 때 멀티 테넌트 관점에서 고려해야 하는 사항들이 많지는 않습니다.React에 관한 다른 다양한 자료를 살펴 보시기를 추천 드립니다. 이 워크샵에서는 마이크로 서비스 마이그레이션 및 온보딩 자동화를 위해 채택할 전략에 중점을 두어 단일 테넌트에서 다중 테넌트 마이크로 서비스로 전환하는 과정을 살펴볼 것 입니다.

Therefore, we will intentionally gloss over some of the details of onboarding, identity, and how we built our new React client and get into how our automated onboarding will orchestrate the creation of tenant resources. Here's a conceptual view of the architecture:

그러므로 저희는 온보딩, indentity 에 관한 세부 사항과 어떻게 새로운 React 클라이언트를 만드는지 살펴 보기 보다 어떻게 자동화된 온보딩 프로세스가 테넌트 별 리소스 생성을 오케스트레이션 하는지 살펴 볼 것 입니다. 아래는 이에 대한 개념적인 아키텍처 입니다:

<p align="center"><img src="../images/lab2/LogicalArchitecture.png" alt="Logical Architecture"/></p>

You'll see that we now support a separate application tier for each tenant in your system, each of which has its own storage. This allows us to leave portions of our code in our monolith but be able to operate in a multi-tenant fashion. In this example, Tenant 1 and Tenant 2 are presumed to have already onboarded. Our new S3-hosted web application interacts with these tenant deployments via the API Gateway.

이제 시스템은 각 테넌트별로 별도의 애플리케이션 티어와 스토리지 티어를 갖게될 것 입니다. 이런 아키텍처 모습은 코드의 일부를 모놀리스 애플리케이션에 남겨 두지만 멀티 테넌트 방식으로 작동될 수 있게 해줍니다. 이 예에서 테넌트 1 및 테넌트 2는 이미 온보드 된 것으로 가정합니다. 새로운 S3 호스팅 웹 애플리케이션은 API Gateway를 통해 이러한 테넌트 앱 계층과 상호 작용합니다.

The challenge is that because every tenant will access the system through a shared web experience, we need to introduce some notion of routing that will direct requests per tenant to the appropriate silo of compute and database resources. This is achieved via Application Load Balancer (ALB) listener rules that inspect the headers of each incoming request, and route the traffic to the proper target group that is deployed separately for each registered tenant.

문제는 모든 테넌트가 공유된 웹 환경을 통해 시스템에 액세스하기 때문에 테넌트 당 요청을 적절하게 사일로된 테넌트별 컴퓨팅 및 데이터베이스 리소스로 보내는 라우팅 개념을 도입해야한다는 것입니다. 이는 각 수신 요청의 헤더를 검사한 후 등록 된 각 테넌트별로 배포 된 적절한 대상 그룹으로 트래픽을 라우팅하는 ALB (Application Load Balancer) 리스너 규칙을 통해 만들수 있습니다.

As new tenants are added to the system we must provision a new instance of the application tier for this tenant (shown as Tenant 3 in the picture above). This automated process will both provision the new tier and configure the rules of the ALB to route traffic for Tenant 3 to this cluster.

새로운 테넌트가 시스템에 추가되면이 이 신규 테넌트용으로 새로운 애플리케이션 계층 인스턴스를 프로비저닝해야합니다 (위 그림에서 Tenant 3으로 표시). 이 자동화 된 프로세스는 새 계층을 프로비저닝하고 Tenant 3에 대한 트래픽을 이 클러스터로 라우팅하도록 ALB 규칙을 구성합니다.

Our goals for this lab are to enable this new onboarding automation and register some tenants to verify that the new resources are being allocated as we need. Again, we are only going to highlight the identity and underlying orchestration bits of this process, but we will dive deep on the request routing mechanics that makes this phase of our migration to multi-tenant microservices possible.

이 실습의 목표는 이 새로운 온보딩 자동화를 만들고 일부 테넌트를 등록하여 필요한 만큼 새 리소스가 할당되는지 확인하는 것입니다. 또한, 우리는 indentity 와 온 보딩 프로세스뿐 아니라, 멀티 테넌트 마이크로 서비스로의 마이그레이션 단계를 가능하게하는 Request 라우팅 메커니즘에 대해서도 자세히 살펴볼 것 입니다.

## What You'll Be Building

Before we can start to dig into decomposing our system into microservices, we must introduce the notion of tenancy into our environment. While it's tempting to focus on building new Lambda functions, we have to start by setting up the mechanisms that will be core to creating a new tenant, authenticating them, and connecting their tenant context to each authenticated user for this experience.

시스템을 마이크로 서비스로 분해하기 시작하기 전에, 저희는 먼저 테넌시 개념을 저희 환경에 도입해야합니다. 새로운 Lambda 함수를 만드는 데 중점을두고 있지만, 새로운 테넌트 생성, 인증 및 테넌트 컨텍스트를 인증 된 각 사용자에게 연결하는 핵심적인 메커니즘을 설정하는 것을 우선 시작해야 합니다.

A key part of this lab is deploying an onboarding mechanism that will establish the foundation we'll need to support the incremental migration of our system to a multi-tenant model. This means introducing automation that will orchestrate the creation of each new tenant silo and putting all the wiring in place to successfully route tenants, ideally without changing too much of our monolith to support this environment.

이 실습의 핵심은 시스템의 멀티 테넌트 모델로의 순차적 마이그레이션을 지원하는 데 필요한 기반을 설정하는 온 보딩 메커니즘을 배포하는 것입니다. 이는 각각의 새로운 테넌트를 위한 사일로 형태의 자원 생성을 오케스트레이션 하고 테넌트를 라우팅하기 위해 필요한 복잡한 요소를 제 위치에 배치 하는것을 의미 합니다. 물론 모놀리스 애플리케이션에 대한 너무 많은 변경 사항 없이 말이죠.

As preparation for decomposing our monolithic application tier into microservices, we will also move from an MVC page controller architecture to a REST API based architecture. By exposing our business logic services through a REST API, we start to see what our microservices will look like and we enable moving to a modern client-rendered web front end. We'll extract the UI from the server and replace it with a React application running in the browser. With this context as our backdrop, here are the core elements of this exercise:

모놀리식 애플리케이션 계층을 마이크로 서비스로 분해하기 위한 준비로 MVC 페이지 컨트롤러 아키텍처에서 REST API 기반 아키텍처로 이동할 것입니다. REST API를 통해 비즈니스 로직 서비스를 공개함으로써 우리이 마이크로 서비스가 어떤 모습일지와 modern client-rendered 웹 프론트엔드로 이동되는지 함께 살펴 볼것 입니다. 서버에서 UI를 분리하여 브라우저에서 실행되는 React 애플리케이션으로 대체할 것 입니다. 이를 바탕으로 실습의 주요 부분을 살펴 보겠습니다:

- Prepare the new infrastructure that's needed to enable the system to support separate silos for each tenant in the system. This will involve putting in new mechanisms that will leverage the tenant context we're injecting and route each tenant to their respective infrastructure stack.
- Introduce onboarding and identity that will allow tenants to register, create a user identity in Cognito, and trigger the provisioning of a new separate stack for each tenant. This orchestration is at the core of enabling your first major step toward multi-tenancy, enabling you to introduce tenant context (as part of identity) and automation that configures and provisions the infrastructure to enable the system to run as a true siloed SaaS solution.
- Refactor the application code of our solution to move away from the MVC model we had in Lab 1 and shift to a completely REST-based API for our services. This means converting the controllers we had in Lab 1 into an API and connecting that API to the API Gateway setting the stage for our service decomposition efforts.
- Migrate from a server-side rendered UI to a modern React UI, moving the static JavaScript, CSS and HTML code to an S3 bucket and enabling us to align with best practices for fully isolating the UI from the server. This is a key part of our migration story since it narrows the scope of what is built, deployed, and served from the application tier.
- Dig into the weeds of how the UI connects to the application services via the API Gateway. We'll find some broken code in our UI and reconnect it to the API Gateway to expose you to the new elements of our service integration.
- Use our new UI to onboard new tenants and exercise the onboarding process. The goal here is to illustrate how we will provision new users and tenants. A key piece of this will involve the provisioning of a new application tier (still a monolith) for each tenant that onboards. This will allow us to have a SaaS system where each tenant is running in its own silo on largely "legacy" code while appearing to be a multi-tenant system to your end customers -- your tenants.

Once we complete these fundamental steps, we will have all the moving parts in place to look forward to Lab 3 where we start migrating our monolithic application tier to a series of serverless functions.

- 시스템이 각 테넌트에 대해 별도의 사일로 환경을 갖는데 필요한 새로운 인프라 준비. 여기에는 테넌트 컨텍스트를 활용하고 각 테넌트를 해당 인프라 스택으로 라우팅하는 새로운 메커니즘을 도입하는 것도 포함.
- 테넌트가 등록 되면 Cognito에서 user idendity 생성하며 각 테넌트에 대해 새 별도 스택을 프로비저닝 할 수있는 온보딩 및 Indenity 도입. 이 오케스트레이션은 멀티 테넌시를 향한 첫 번째 핵심사항 이며, 동시에 시스템을 진정한 Siloed SaaS 솔루션으로 실행할 수 있도록 인프라 프로비저닝을 자동화하고 테넌트 컨텍스트 개념을 도입 할 수 있게 함.
- 솔루션의 애플리케이션 코드를 리팩터링하여 Lab 1에서 만든 MVC 모델에서 벗어나 Rest API 형태로 전환. 즉, Lab 1에서 사용했던 컨트롤러를 API로 변환하고 해당 API를 API Gateway에 연결.
- 서버 측 렌더링 UI에서 최신 React UI로 마이그레이션하여 정적 JavaScript, CSS 및 HTML 코드를 S3 버킷으로 이동하고 서버에서 UI를 완전히 격리하기위한 모범 사례 도입. 이는 애플리케이션 계층으로 부터 무엇이 만들어지고, 배포되고, 서비스로 제공되어져야 하는지 범위를 좁히기 때문에 마이그레이션 스토리의 핵심 부분.
- API가 API Gateway를 통해 UI가 애플리케이션 서비스에 연결되는 방식에 대해 자세히 살펴봄.
- 새로운 UI를 사용하여 새 테넌트를 온 보딩하고 온 보딩 프로세스를 연습. 이때 목표는 새로운 사용자 및 테넌트를 프로비저닝하는 방법을 이해하는 것. 이것의 핵심 부분은 온보드 과정에 있는 각 테넌트에 대해 새로운 애플리케이션 티어 (여전히 모놀리스)를 프로비저닝하는 것을 말함. 이를 통해 각 테넌트 서비스가 본래의 "레거시"코드를 바탕으로 사일로 형태의 멀티 테넌트 아키텍처위에서 구동됨.

이런 기본 단계들을 완료하면 여러분은 Lab3에서 모놀리식 애플리케이션 계층을 서버리스 형태로 마이그레이션 하기 위한 준비를 마칠 수 있습니다.

## Step-By-Step Guide

Starting Lab 2 you should have a good sense of the core elements of our monolith. It's time to start the modernization process. The steps below will guide you through the migration to a REST based API for our services which will support a new, modern UI. We will introduce multi-tenant onboarding, support for authentication with tenant context, and the automated provisioning of tenant application tiers.

Lab 2를 시작과 동시에 여러분은 실습 대상이 되는 모놀리스의 핵심 요소에 대해 잘 파악해야 합니다. 왜나하면 이제 이를 모더나이즈 할 시간이기 때문입니다. 아래 단계는 여러분들에게 새롭고 최신 UI 서비스를 위한 REST API 기반으로 기존 애플리케이션을 마이그레이션하는 과정을 안내할 것입니다. 앞으로 멀티 테넌트 온보딩, 테넌트 컨텍스트를 통한 인증 지원 및 테넌트 애플리케이션 계층의 자동화 된 프로비저닝 과정을 소개 할 것 입니다.

<b>Step 1</b> – The first step in our migration to multi-tenancy is to introduce the core infrastructure that will enable us to have separate silos for each of our tenant environments. Our initial single-tenant system simply directed traffic through an ALB that routed _all_ traffic to a single stack for our one customer. However, to support multi-tenancy, we'll now have multiple instances of our stack and will need to put new routing infrastructure in front of these stacks to support this advanced architecture.

To introduce these new constructs, we'll need to first execute a script that will use CloudFormation to configure the elements of this lab. Running the script will require you to navigate to the Cloud9 service in the AWS console and open the IDE for this workshop. Once the IDE is open, go to the terminal window in the lower window pane and run the following commands:

<b>Step 1</b> – 멀티 테넌시로 마이그레이션하는 첫 번째 단계는 각 테넌트 환경에 대해 별도의 사일로 환경을 가질 수 있도록 핵심 인프라를 도입하는 것입니다. 초기 단일 테넌트 시스템은 단순히 _all_ 트래픽을 단일 고객의 단일 스택으로 라우팅하는 ALB를 통해 트래픽을 전달했습니다. 그러나 멀티 테넌시를 지원하기 위해 이제 애플리케이션 스택의 여러 인스턴스들을 갖게 되며 우리는 이제 이 고급 아키텍처를 지원하기 위해 이러한 여러 애플리케이션 스택 앞에 새로운 라우팅 인프라를 배치해야 합니다.

이러한 새로운 구성을 도입하려면 먼저 CloudFormation을 사용하여 이 실습의 환경을 구성하는 스크립트를 실행해야합니다. 스크립트를 실행하려면 AWS 콘솔에서 Cloud9 서비스로 이동하여이 워크샵을 위한 IDE를 열어야합니다. IDE가 열리면 하단에 위치한 터미널 창으로 이동하여 다음 명령을 실행합니다:

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh lab2.sh
```

<b>Step 2</b> – Let's have a look at the status of the infrastructure deployment that we just kicked off. Navigate to the CloudFormation service within the AWS console. Locate the stack in the list that is contains <b>lab2</b> in its name. This stack is responsible for creating the infrastructure to support multi-tenancy including our new routing infrastructure. The screen should be similar to the following:

<b>Step 2</b> – 방금 실행한 인프라 배포의 상태를 살펴 보겠습니다. AWS 콘솔 내에서 CloudFormation 서비스로 이동합니다. 이름에 <b>lab2</b>가 포함 된 목록에서 스택을 찾습니다. 이 스택은 새로운 라우팅 인프라를 포함하여 멀티 테넌시를 지원하기위한 인프라를 생성합니다. 화면은 다음과 유사해야합니다.

<p align="center"><img src="../images/lab2/CloudFormation.png" alt="Cloud Formation"/></p>

We must wait until the stack has a status of <b>CREATE_COMPLETE</b>, indicating that all the elements of the stack have been created. If it is not complete, continue to select the refresh button (just to left of the Delete button at the top of the page) to get updated status. <b>You must wait for this process to finish before moving onto the next step</b>. The stack should take less than 5 minutes to complete.

스택의 상태가 <b>CREATE_COMPLETE</b>가 될 때까지 기다려야합니다. 이는 스택의 모든 요소가 생성되었음을 나타냅니다.아직 완료되지 않은 경우 계속해서 새로 고침 버튼 (페이지 상단의 삭제 버튼 바로 왼쪽)을 눌러 업데이트 된 상태를 가져옵니다. <b>다음 단계로 이동하기 전에 이 프로세스가 완료 될 때까지 기다려야합니다</b>. 스택을 완료하는 데 5 분 정도 소요될 것 입니다.

<b>Step 3</b> – As part of moving to a multi-tenant environment, we've also opted to migrate our monolithic web UI (where all the HTML was rendered and served from the app server) to a modern UI framework that is served from Amazon S3 and executes in the user's browser. While this could be viewed as an optional step for many who are migrating to SaaS, we felt it was a compelling strategy and wanted to illustrate how making a move of this nature would influence the look of your final environment. The details of the React application that we'll deploy are mostly out of scope for this effort, but we encourage you to examine the code more carefully to understand how it interacts with the services of our environment.

For now, our goal is to simply get this new UI deployed and working so we can begin to interact with our new multi-tenant model. To simplify things, we've created a shell script to build the React application and copy it to an S3 bucket to make it accessible. <b>You must ensure that the lab2 CloudFormation stack has completed successfully before continuing</b>. To run this script, navigate to your Cloud9 environment and enter the following commands to execute the web client deployment script:

<b>Step 3</b> – 멀티 테넌트 환경으로 이동하는 과정에서 모놀리식 웹 UI (모든 HTML이 앱 서버에서 렌더링되고 제공됨)를 Amazon S3에서 호스팅 되고 사용자의 브라우저에서 실행되는 최신 UI 프레임워크로 마이그레이션 하도록 결정했습니다. 이 결정이 사실은 SaaS로 마이그레이션하는 분들에 따라 다르게 나타날 수 있지만, 우리는 이 선택이 매력적인 전략이라고 느꼈고 이러한 마이그레이션이 최종 적인 아키텍처 모습에 어떤 영향을 미치는지 설명하고 싶었습니다. 우리가 배포 할 React 애플리케이션의 세부 사항은 본 실습의 범위를 벗어 나지만 이 React 코드가 우리 아키텍처의 다른 서비스와 어떻게 상호 작용하는 이해하려면 코드를 주의 깊게 살펴 보는 것도 좋습니다.

현재 우리의 목표는 단순하게 이 새로운 UI를 배포하고 작동시켜 새로운 멀티 테넌트 모델과 상호 작용을 시작할 수 있도록 만드는데 있습니다. 이 작업을 단순화 하기 위해 React 애플리케이션을 빌드하고 S3 버킷에 복사하여 액세스 할 수 있도록 셸 스크립트를 만들었습니다. <b>계속하기 전에 lab2 CloudFormation 스택이 성공적으로 완료되었는지 확인해야합니다 </b>. 이 스크립트를 실행하려면 Cloud9 환경으로 이동하고 터미널 창에서 다음 명령을 입력하여 웹 클라이언트 배포 스크립트를 실행합니다:

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh website-lab2.sh
```

<b>Step 4</b> – Now that our stack has been created, let's go look at the infrastructure we've introduce. In the new multi-tenant model we're building in this lab, each request that comes in from our React client will include the tenant context in a JWT token (we'll look deeper into how that works later in this lab). The API Gateway exposes our REST resources to client requests and manipulates the HTTP headers so the Application Load Balancer can route the requests to the appropriate tenant stack (silo). The first step in making this routing work is to construct the API Gateway and associate a custom authorizer with it that will extract our tenant context from the JWT token and expose it as part of the request context to the downstream resource the API Gateway proxies.

A custom authorizer is simply a Lambda function that is invoked with each request that is processed by the API Gateway. Within this Lambda function, we can inspect the incoming authorization token and inject context for downstream processing. To view the custom authorizer that was provisioned, navigate to the API Gateway in the AWS console. And select <b>saas-factory-srvls-wrkshp-lab2</b> from the list of APIs. Then, with this API selected, choose <b>Authorizers</b> from the menu of options displayed on the left. After you select this option, you'll see a page similar to the following:

<b>Step 4</b> – 이제 스택이 생성 되었으므로 앞서 소개한 인프라를 살펴 보겠습니다. 이 실습에서 구축중인 새로운 다중 테넌트 모델에서 React 클라이언트에서 들어오는 각 요청에는 JWT 토큰에 테넌트 컨텍스트가 포함됩니다 (이 실습의 뒷부분에서 어떻게 작동하는지 자세히 살펴 보겠습니다). API Gateway는 REST 리소스를 클라이언트에 노출하고 HTTP 헤더를 조작하여 Application Load Balancer가 요청을 적절한 테넌트 스택(사일로)으로 라우팅 할 수 있도록합니다. 이 라우팅 작업을 수행하는 첫 번째 단계는 API Gateway를 구성하고 JWT 토큰에서 테넌트 컨텍스트를 추출하고 API Gateway가 프록시 하는 다운스트림 리소스에 대한 요청 컨텍스트의 일부로 전달(노출) 할 사용자 지정 권한 부여자(Custom Authorizer)를 연결하는 것입니다.

사용자 지정 권한 부여자는 API Gateway에서 처리하는 각 요청과 함께 호출되는 단순한 Lambda 함수입니다. 우리는 이 Lambda 함수 내에서 수신된 권한 부여 토큰(Authorization token)을 검사하고 다음 이어질 다운 스트림 처리를 위해 컨텍스트를 삽입 할 수 있습니다. 프로비저닝 된 사용자 지정 권한 부여자를 보려면 AWS 콘솔에서 API Gateway로 이동합니다. 그리고 API 목록에서 <b>saas-factory-srvls-wrkshp-lab2</b>를 선택합니다. 그런 다음 이 API를 선택한 상태에서 왼쪽에 표시된 옵션 메뉴에서 <b>Authorizers</b>를 선택합니다. 이 옵션을 선택하면 다음과 유사한 페이지가 표시됩니다:

<p align="center"><img src="../images/lab2/LambdaAuthorizer.png" alt="Lambda Authorizer"/></p>

You'll see that our authorizer expects an event payload of type <b>Token</b> and is associated with a specific Lambda function. In this case, our function is <b>saas-factory-srvls-wrkshp-lambda-authorizer-[REGION]</b>. Also note the <b>Token Source</b> is set to Authorization (more on this below).

권한 부여자가 <b>Token</b> 유형의 이벤트 페이로드(Lambda Event Payload)를 기다리면서 특정 Lambda 함수와 연결되어 있음을 알 수 있습니다. 이 경우 함수는 <b>saas-factory-srvls-wrkshp-lambda-authorizer-[REGION]</b>입니다. 또한 <b>Token Source</ b>가 Authorization 으로 설정되어 있습니다 (자세한 내용은 아래 참조).

<b>Step 5</b> – Time to dive deep on Lambda Authorizers! Let's open code for this Lambda function. Navigate back to Cloud9 and open the <b>Authorizer.java</b> file located in <b>resources/lambda-authorizer/src/main/java/com/amazon/aws/partners/saasfactory</b>

<b>Step 5</b> – Lambda Authorizer에 대해 자세히 알아볼 시간입니다! 이 Lambda 함수에 대한 코드를 열어 보겠습니다. Cloud9로 다시 이동하여 <b>resources/lambda-authorizer/src/main/java/com/amazon/aws/partners/saasfactory</b>에있는 <b>Authorizer.java</b> 파일을 엽니 다.

<p align="center"><img src="../images/lab2/LambdaAuthorizerCode.png" alt="Lambda Function"/></p>

Here's a snippet of code from that file:

여기 코드의 일부가 있습니다.

<p align="center"><img src="../images/lab2/LambdaCode.png" alt="Lambda Function"/></p>

Notice this is a normal Lambda request handler method. First, we parse the <b>authorizationToken</b> from the incoming <b>event</b>. The value of the authorizationToken is defined by the <b>Token Source</b> when you setup the authorizer for the API Gateway. We chose the <b>Authorization</b> header from the HTTP request which contains our signed JWT token. Once we have extracted the tenant identifier from the token, we add it to the <b>context</b> of the response object.

이는 일반적인 Lambda 요청 핸들러 메서드 라는 점을 말씀 드립니다. 먼저 수신 <b>event</b>에서 <b>authorizationToken</b>을 구문 분석합니다. authorizationToken의 값은 API Gateway에 대한 권한 부여자를 설정할 때 <b>Token Source</b>에 의해 정의됩니다. 저희는 서명 된 JWT 토큰이 포함 된 HTTP 요청에서 <b>Authorization</b> header를 선택했습니다. 토큰에서 테넌트 식별자를 추출한 후 response object의 <b>context</b>에 추가합니다.

<b>Step 6</b> - Now that the Lambda Authorizer has extracted the tenant identifier from the signed JWT token in the HTTP Authorization header and passed it along to the API Gateway as part of the request context, we can map that value to a resource method's <b>Integration Request</b>. Let's take a look at how we do that. Go back to the Amazon API Gateway console, select the <b>saas-factory-srvls-wrkshp-lab2</b> API and then select <b>Resources</b> from the left-hand menu. Now click on the <b>GET</b> method listed under the <b>/products</b> resource. Your screen should look similar to this:

<b>Step 6</b>-이제 Lambda Authorizer가 HTTP Authorization 헤더의 서명 된 JWT 토큰에서 테넌트 식별자를 추출하고 이를 요청 컨텍스트의 일부로 API Gateway에 전달 했으므로 해당 값을 resource method의 <b>Integration Request</b>에 매핑할 수 있습니다. 이를 어떻게 하는지 살펴 보겠습니다. Amazon API Gateway 콘솔로 돌아가 <b>saas-factory-srvls-wrkshp-lab2</b> API를 선택한 다음 왼쪽 메뉴에서 <b>Resources</b>를 선택합니다. 이제 <b>/products</b> 리소스 아래에 나열된 <b>GET</b> 메소드를 클릭합니다. 화면은 다음과 유사해야합니다:

<p align="center"><img src="../images/lab2/APIProductsGet.png" alt="Products GET Method"/></p>

Now, click on the blue <b>Integration Request</b> in the upper right of the 4 phases of the API method execution settings. Expand the caret/triangle next to <b>HTTP Headers</b> and you'll see that we've added a custom header named <b>X-Tenant-ID</b> and set its value as mapped from <b>context.authorizer.TenantId</b>. This is the TenantId property we set on the AuthorizerResponse object in our Lambda function.

이제 API 메서드 실행 설정의 4 단계 오른쪽 상단에 있는 파란색 <b>Integration Request</b>을 클릭합니다. <b>HTTP Headers</b> 옆에 있는 삼각형 모양을 확장하면 <b>X-Tenant-ID</b>라는 사용자 지정 헤더가 추가되고 해당 값이 <b>context.authorizer.TenantId</b>에서 매핑 되도록 설정 되었음을 알 수 있습니다. 이것은 Lambda 함수에서 AuthorizerResponse object에 설정 한 TenantId 속성입니다.

 <p align="center"><img src="../images/lab2/APIIntegrationRequest.png" alt="Integration Request"/></p>

<b>Step 7</b> – Now that we have our API Gateway injecting the header, the next piece to look at is the ALB which will be using this header to route traffic to each of our stacks. To view the ALB that was provisioned for our environment, navigate to the EC2 service in the AWS console and select the <b>Load Balancers</b> item from the menu on the left of the page (you may have to scroll down to find this menu item). The next step is to locate the ALB in the list of ALBs. Select the box next to the <b>saas-wrkshp-lab2-[REGION]</b>. The page should appear similar to the following:

<b>Step 7</b> – 이제 API Gateway가 header를 주입(추가) 했으므로 다음 으로 살펴볼 부분은 이 header를 사용하여 트래픽을 각 스택으로 라우팅 할 ALB 입니다. 여러분의 환경에 프로비저닝 된 ALB를 보려면 AWS 콘솔에서 EC2 서비스로 이동하고 페이지 왼쪽의 메뉴에서 <b>Load Balancer</b> 항목을 선택합니다 (아래로 이 메뉴 항목 찾기까지 스크롤 다운 해야 합니다). 다음 단계는 ALB 목록에서 ALB를 찾는 것입니다. <b>saas-wrkshp-lab2-[REGION]</b> 옆의 상자를 선택합니다. 페이지는 다음과 유사하게 나타납니다.

<p align="center"><img src="../images/lab2/Lab2ALB.png" alt="ALB"/></p>

With this item selected you can select the <b>Listeners</b> tab in the lower section of the page to view the listeners associated with this ALB. There is a single listener defined for our load balancer. It is listening for incoming HTTP requests on port 80. Click on the <b>View/edit rules</b> link in the right-most column of the listeners table. Now, the expectation here is that we would have seen routing rules in this listener list that would apply the <b>X-Tenant-ID</b> header we injected in the API Gateway to route traffic to the appropriate target group. However, there is only a default rule returning an HTTP 401 Unauthorized status for any request to this ALB. Why is that? It's because we haven't actually registered any tenants yet. An ALB must have at least one listener and a listener must have, at a minimum, a default rule. We have added an unauthorized rule to protect our system. The custom routing rules for our multi-tenant architecture only get added during the provisioning of each new tenant silo. We'll circle back here after we provision some tenants to see how that changes the configuration of our ALB.

이 항목을 선택하면 페이지 하단의 <b>Listeners</b> 탭을 선택하여 이 ALB와 관련된 리스너를 볼 수 있습니다. 로드 밸런서에 대해 정의 된 단일 리스너가 확인될 것입니다. 현재 포트 80에서 들어오는 HTTP 요청을 수신하고 있습니다. 리스너 테이블의 맨 오른쪽 열에 있는 <b>View/edit rules</b> 링크를 클릭하십시오. 이제 여기서 여러분의 기대는 리스너 목록에서 트래픽을 적절한 대상 그룹으로 라우팅하기 위해 API Gateway에서 삽입 한 <b>X-Tenant-ID</b> 헤더를 적용하는 라우팅 규칙을 보는것 일 겁니다. 그러나 이 ALB에 대한 모든 요청에 ​​대해 HTTP 401 Unauthorized 상태를 반환하는 기본 규칙 만 있습니다. 왜 그럴까요? 아직 테넌트가 등록 되지 않았기 때문입니다. ALB에는 최소한 하나의 리스너가 있어야 하며 리스너에는 최소한 기본 rule이 있어야합니다. 그래서 우선 시스템을 보호하기 위해 unauthorized 에 대한 rule만을 추가했습니다. 멀티 테넌트 아키텍처에 대한 사용자 지정 라우팅 규칙은 각각의 새 테넌트들이 사일로 스택으로 프로비저닝하는 동안에 만 추가가 됩니다. 따라서 저희는 일부 테넌트를 프로비저닝 한 후 여기로 돌아와 ALB 구성이 어떻게 변경되는지 확인할 예정 입니다.

<b>Step 8</b> – Before we can use our new React UI, we'll need a new URL to use for accessing our application (since it is now hosted on S3 and not served from the application server). To find the URL of the application, you'll need to navigate to the CloudFront service in the AWS console. This page will show a list of distributions. You should see a distribution listed with the origin value of <b>[StackID]-lab1-[RANDOM]-websitebucket-[RANDOM].s3-website-[REGION].amazonaws.com</b>. Copy the <b>Domain Name</b> value. You'll want to make note of this value, since it will be used throughout the remainder of this workshop to access the client application.

<b>Step 8</b> – 새 React UI를 사용하려면 먼저 애플리케이션에 액세스하는 데 사용할 새 URL이 필요합니다. (이제 S3에서 호스팅되고 애플리케이션 서버에서 제공되지 않기 때문에). 애플리케이션의 URL을 찾으려면 AWS 콘솔에서 CloudFront 서비스로 이동해야합니다. 이 페이지에는 배포 목록이 표시됩니다. 원본 값이 <b>[StackID]-lab1-[RANDOM]-websitebucket-[RANDOM].s3-website-[REGION].amazonaws.com</b> 인 배포가 확인 되어야 합니다. <b>Domain Name</b> 값을 복사합니다. 이 값은 이 워크숍의 나머지 부분에서 클라이언트 응용 프로그램에 액세스하는 데 사용되므로 기록해 두는 것이 좋습니다.

<p align="center"><img src="../images/lab2/CloudFrontDistributions.png" alt="CloudFront"/></p>

<b>Step 9</b> – Now that we have the URL, we can access the application and verify that it works. Enter the URL we captured from the prior step and open the application. Our new React client is up and running and being served from S3 and cached at global edge locations by CloudFront. When the application opens it will appear as follows:

<b>Step 9</b> – 이제 URL이 확인되었기 때문에 애플리케이션에 액세스하여 작동 여부를 확인할 수 있습니다. 이전 단계에서 캡처 한 URL을 입력하고 애플리케이션을 열어봅니다. 새로운 React 클라이언트가 가동 및 실행 중이며 S3 에서 호스팅되고 CloudFront에 의해 글로벌 엣지 로케이션에 캐시됩니다. 애플리케이션이 열리면 다음과 같은 화면이 나타날 것 입니다.

<p align="center"><img src="../images/lab2/Homepage.png" alt="Homepage"/></p>

This page looks remarkably like the application from the monolith solution that was used in Lab 1. While they look similar, in the real-world scenario, you'd likely redesign aspects of your UI during the rewrite with a modern UI framework.

보이는 화면(페이지)의 모습은 Lab 1에서 사용된 모놀리식 애플리케이션과 매우 유사합니다. 유사해 보이지만 Real-World 상황에서는 아마도 여러분들은 최신 UI 프레임워크로 새롭게 만드는 동안 전체적인 UI 디자인 역시 변경 하실겁니다.😎

<b>Step 10</b> – With this new multi-tenant environment, we can no longer simply sign-in to the system. As a SaaS system, we now onboard our tenants by having them complete a registration process. This is an important step in thinking about your migration. With this registration process, we are essentially presenting our system to end users as a fully SaaS system. This represents a key milestone in your migration approach, enabling your old monolith to run largely unchanged while providing the foundation for migrating the underlying implementation without users being aware of the shift to a multi-tenant serverless implementation.

Let's create our first tenant by selecting the "Sign Up" button at the top right of our application. Upon selecting this option, you'll be presented with a form similar to the following:

<b>Step 10</b> – 이 새로운 멀티 테넌트 환경에서는 더 이상 단순한 시스템 로그인을 사용할 수 없습니다. SaaS 시스템으로서 우리는 이제 테넌트가 등록 프로세스를 완료 하도록 하여 서비스에 온보딩할 수있도록 합니다. 이것은 마이그레이션에 대해 생각할 때 중요한 단계입니다. 이 등록 프로세스를 통해 우리는 기본적으로 최종 사용자에게 완전한 하나의 시스템으로 SaaS 서비스를 제공합니다. 이는 마이그레이션 접근 방식에 있어 중요한 밑바탕이 됩니다. 왜냐하면 이를 통해 사용자는 멀티 테넌트 서버리스 아키텍처로 전환되었다는 사실을 인식하지 못하게 하면서 기본 구현을 마이그레이션 할 수있는 기반을 마련할 수 있기 때문입니다.

애플리케이션의 오른쪽 상단에있는 "Sign up" 버튼을 선택하여 첫 번째 테넌트를 생성 해 보겠습니다. 이 옵션을 선택하면 다음과 유사한 양식이 표시됩니다.

<p align="center"><img src="../images/lab2/Signup.png" alt="Signup"/></p>

Enter the values for your tenant and your tenant's first user (you do _not_ have to use a real email address for this workshop). This page is meant to be a bit of a simplified SaaS registration page, collecting common attributes that might be collected as new tenants onboard. If your organization doesn't support direct registration, you should still have internal automated tooling that would be collected and used to trigger onboarding. Please make sure to take a note of the email address and password you are providing.

테넌트와 테넌트의 첫 번째 사용자에 대한 값을 입력합니다 (이 워크숍에 실제 이메일 주소를 사용할 필요는 _없습니다_). 이 페이지는 새로운 테넌트 온보드시 수집 될 수있는 공통 속성을 수집하는 단순한 SaaS 서비스 가입 페이지입니다. 만약 여러분의 조직이 이런 직접 가입(등록)을 지원하지 않을 경우 온 보딩을 트리거 하면서 관련 정보도 수집할 수 있는 자동화 도구가 여전히 있어야 합니다. 다음 확인을 위해 가입시 제공한 이메일 주소와 비밀번호를 기록해 두십시오.

<b>Step 11</b> - The registration form triggers a series of steps orchestrated by a Registration Service. A CloudFormation stack has been launched to onboard the new tenant. Verify the status of the stack by navigating to CloudFormation in the console and see the stack being created. This stack will take a few minutes to provision all of the siloed infrastructure for your new tenant. <b>You must wait for this stack to complete before proceeding</b>.

<b>Step 11</b>-등록 양식은 Registration Service에 의해 조정(orchestrated)되는 일련의 단계를 트리거 합니다. 즉 새로운 테넌트를 온보딩하기 위해 CloudFormation 스택이 시작 됩니다. 콘솔에서 CloudFormation으로 이동하여 스택 상태를 확인하고 생성중인 스택을 확인합니다. 이 스택은 새 테넌트에 대한 모든 사일로 인프라를 프로비저닝 하며 완료 까지 몇 분 정도 걸립니다. <b>계속하기 전에이 스택이 완료 될 때까지 기다려야합니다</b>.

<p align="center"><kbd><img src="../images/lab2/TenantRegistrationStack.png" alt="Tenant Registration"/></kbd></p>

<b>Step 12</b> - Just as in Lab 1, we need to trigger the CI/CD pipeline to deploy our monolith to the silo of infrastructure created for your new tenant. Navigate to CodePipeline and click on the <b>saas-factory-srvls-wrkshp-pipeline-lab2</b> pipeline. This pipeline will be in failed state as of now. Clicking on the pipeline will take us to details page as below. Click on the orange <b>Release Change</b> button to launch the pipeline. <b>You must wait for all 3 phases of the pipeline to finish successfully before continuing</b>.

<b>Step 12</b>- Lab 1과 마찬가지로 여러분은 이제 CI/CD 파이프 라인을 트리거하여 모놀리스를 새 테넌트 용으로 생성된 시일로 인프라에 배포해야합니다. CodePipeline으로 이동하여 <b>saas-factory-srvls-wrkshp-pipeline-lab2</b> 파이프 라인을 클릭합니다. 현재 이 파이프 라인은 실패 상태 일겁니다. 파이프 라인을 클릭하면 아래와 같은 세부 정보 페이지로 이동합니다. 주황색 <b>Release Change</b> 버튼을 클릭하여 파이프 라인을 시작합니다. <b>계속하기 전에 파이프 라인의 3 단계가 모두 완료 될 때까지 기다려야합니다</b>.

<p align="center"><kbd><img src="../images/lab2/ReleaseChange.png" alt="Release Change"/></kbd></p>

<b>Step 13</b> - Once the pipeline has completed successfully, our new tenant is fully onboarded into their own stack! Let's go back to our client web application hosted at the CloudFront domain name you captured above and sign in using the email and password which you used during the registration process. Click on <b>Sign In</b> button on top right corner and enter your login details. Click on <b>Sign In</b> to authenticate into the application.

<b>Step 13</b>- 파이프 라인이 성공적으로 완료되면 새 테넌트가 태넌트 만을 위해 마련된 스택에 완전히 온보딩 됩니다! 위에서 캡처 한 CloudFront 도메인 이름으로 호스팅 된 클라이언트 웹 애플리케이션으로 돌아가서 등록 프로세스 과정에서 입력한 이메일과 암호를 사용하여 로그인하겠습니다. 오른쪽 상단의 <b>Sign In</b> 버튼을 클릭하고 로그인 세부 정보를 입력하십시오. <b>Sign In</b>을 클릭하여 애플리케이션에 인증을 받으십시오.

<p align="center"><kbd><img src="../images/lab2/LoginPage.png" alt="Login Page"/></kbd></p>

<b> Step 14</b> – Once you're in the application, you will land on the dashboard page (just like the monolith experience in Lab 1) that is a placeholder for providing analytics about your ecommerce business. The page also has a navigation bar at the top to access the various capabilities of the application. The page will appear as follows:

<b>Step 14</b> – 우선 애플리케이션에 로그인하면 전자 상거래 비즈니스에 대한 분석을 제공하는 대시 보드 페이지 (Lab 1의 모놀리식 애플리케이션으로 확인한것 과 같이)로 이동할 것 입니다. 페이지 상단에는 애플리케이션의 다양한 기능에 액세스 할 수있는 메뉴 모음도 있을 것입니다. 즉 페이지는 다음과 같이 나타납니다.

<p align="center"><img src="../images/lab2/Dashboard.png" alt="Dashboard"/></p>

<b>Step 15</b> – Now, let's access the product page by selecting the <b>Products</b> item from the navigation at the top of the page. The page will be empty because this tenant has just registered and hasn't added any products to their catalog. The screen will appear as follows:

<b>Step 15</b> – 이제 페이지 상단의 탐색에서 <b>Products</b> 항목을 선택하여 제품 페이지에 액세스 해보겠습니다. 그런데 이 테넌트가 방금 등록했고 카탈로그에 제품을 추가하지 않았기 때문에 페이지가 비어 있습니다. 화면은 다음과 같이 나타납니다.

<p align="center"><kbd><img src="../images/lab2/EmptyProducts.png" alt="Empty Products"/></kbd></p>

<b>Step 16</b> – Click the <b>Add Product</b> button to create a new product. Upon selecting this option, you will be presented with a form to enter the product information that appears as follows:

<p align="center"><img src="../images/lab2/AddProduct.png" alt="Add Product"/></p>

Just as you did in Lab 1, enter some product information and select <b>Add Product</b> to save your product information. This will return you to the list of products where you will be able to see that your new product was added. Make sure to add at least two products, because we will need them in Lab 3.

<p align="center"><img src="../images/lab2/Products.png" alt="Products"/></p>

<b>Step 17</b> – Now that we've successfully added a tenant and some products to our system, we can take a closer look at what the system provisioned for each tenant and how the routing was configured to direct individual tenants to their respective silos. We'll start this process by looking at how our tenant was provisioned into Amazon Cognito (which authenticates users and provides the essential JWT token that controls the flow of tenants through the system).

Navigate to the Cognito service within the AWS console. In this example, we're provisioning a separate user pool for each tenant. These pools let us group and configure policies separately for each tenant. Select <b>Manage User Pools</b> from the landing page and you'll be presented with a list of user pools similar to the following:

<p align="center"><img src="../images/lab2/UserPools.png" alt="User Pools"/></p>

Each time you add a new tenant to the system, a new Cognito User Pool will be created. At this point, you should have only one pool since we've only added one tenant. Select that pool from the user pool page. This will provide you with a summary of the pool configuration. Now, select <b>Users and groups</b> from the left-hand side of the page to view users that currently reside in this pool. The page will appear as follows:

<p align="center"><img src="../images/lab2/Users.png" alt="Users"/></p>

Listed here will be the user that you registered when you created your tenant. Select the link for your user name to view the attributes of the user you created. A page similar to the following will appear:

<p align="center"><img src="../images/lab2/UserAttributes.png" alt="User Attributes"/></p>

When we provisioned the tenant user pool, we configured specific attributes that allow us to track this user's relationship to a specific tenant. This is shown as the <b>custom:tenant_id</b> custom attribute for the user. This tenant identifier will now be included in the JWT token that is returned from your authentication experience and will be passed through as part of all our interactions with downstream services.

<b>Step 18</b> – So, we have a tenant identifier embedded in our JWT token and we've seen how the API Gateway custom authorizer will inject tenant context. However, if you recall, when we looked at the ALB it did not have a routing rule for our tenant because we hadn't onboarded any yet. Now we do have a tenant and we can return to see how the ALB was configured to support the routing for this new tenant. To view this new information, navigate to the EC2 service in the AWS console and select <b>Load Balancers</b> from the left-hand side of the page (you may have to scroll down some to find it). This will provide you with a list of load balancer similar to the following:

<p align="center"><img src="../images/lab2/Lab2ALB.png" alt="Lab2 ALB"/></p>

Select the <b>saas-wrkshp-lab2-[REGION]</b> load balancer from the list. Now, scroll down the page and select the <b>Listeners</b> tab for the ALB. Click on the <b>View/edit rules</b> link and you'll now see a rule has been added specifically for our tenant to control routing. Note that this forwarding rule is set to a higher priority than the default 401 unauthorized rule. The screen will appear similar to the following:

<p align="center"><img src="../images/lab2/ALBListeners.png" alt="ALB Listeners"/></p>

This rule examines the value of the X-Tenant-ID header we inserted via our custom authorizer and forwards traffic to the target group for that tenant's stack of infrastructure. As each new tenant is added, a new rule will be introduced in this list.

<b>Step 19</b> – In addition to configuring the routing, the onboarding process also provisioned a separate, siloed set of compute resources for each tenant. This cluster of auto-scaled instances continue to run the application services portion of our system. If we look closely at the EC2 instances and databases, you'll find there are separate instances and a separate database provisioned for each tenant. We won't dig into this too deeply. However, it's a critical element of this model that enables our future incremental move to microservices.

Let's take a quick look at the EC2 resources that we currently have to get a better sense of what was created. Navigate to the EC2 service in the console and select the <b>Instances</b> option from the menu on the left-hand side of the page. In this list of instances, you'll see instances with the name <b>saas-factory-srvls-wrkshp-lab2-[TENANT_ID]</b> that represent the instances that were provisioned for your new tenant. If you onboard another tenant, you'd see more instances added here to support that tenant.

<p align="center"><img src="../images/lab2/EC2Instances.png" alt="EC2 Instances"/></p>

<b>Step 20</b> – Now that you see how the infrastructure and onboarding have changed to support our multi-tenant model, let's look at how the new React client, the API Gateway, and the proxied application services in the tenant silos all connect together. Let's go back to the web client using the CloudFront URL you captured above and sign in with the email address and password you used to register your new tenant.

Once you've logged in, you'll land on the dashboard home page. Click on the <b>Products</b> link in the navigation header to go to the product catalog listing page. Your list of products may be empty. Add a couple of products to your catalog.

Now, delete a product by selecting the red <b>Del</b> icon on the right hand side of the row of the product you'd like to remove. You will see a confirmation dialog asking if you really want to delete your product. Click on the <b>Delete Product</b> button. Nothing happens! Why? Time to debug.

<b>Step 21</b> - Let's start our trace from the client to the server by looking at some code in the React application. To get to this code, you'll need to open Cloud9 in the AWS console again and open the IDE environment for this workshop. Open the <b>lab2/client/src/components/products/actions</b> folder in the navigation pane on the left. Double-click on the <b>index.js</b> file to view its contents. In this file you'll find a <b>deleteProduct</b> function toward the bottom that appears as follows:

```javascript
export const deleteProduct = (product) => {
  return function (dispatch) {
    const url = `/products/${product.id}`;
    const instance = createAxiosInstance();

    dispatch(closeModal());

    /*
        instance.delete(url, { data: product })
            .then(response => {
                const deletedProduct = response.data;
                if (deletedProduct && deletedProduct.id) {
                    dispatch(deleteProductFinished(deletedProduct));
                } else {
                    dispatch(errorModal("The product was not deleted."));
                }
            }, error => console.error(error))
            .then(() => {
            }, error => console.error(error));
        */
  };
};
```

This function is called whenever the client indicates that they want to invoke the DELETE REST method of the Product service. You'll see here that the actual invocation of this method has been disabled. We'll need to un-comment the block of code that makes the call as the first step in repairing this broken path in our application. Remove the comment markers and save the file with the Ctrl-S keyboard shortcut, or by choosing <b>Save</b> from the <b>File</b> menu in Cloud9.

<b>Step 22</b> – Now that we've repaired our client code, we'll need to re-deploy our changes to S3 to have them applied. With the Cloud9 IDE, navigate to the terminal window and executed the following command to re-deploy our client:

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh website-lab2.sh
```

<b>Step 23</b> – Now, let's navigate back to the application and attempt to delete the product again (using the URL and credentials we used above). Be sure to <b><i>refresh your web browser</i></b> to force it to pull down a fresh copy of the JavaScript you just fixed. Our updates now have the React client submitting our delete action. However, despite our changes, delete is still not working.

<p align="center"><img src="../images/lab2/ProductDeleteError.png" alt="Product Delete Error"/></p>

This is because the API Gateway is still not wired to connect to our application tier services. To resolve this, we'll need to open the API Gateway service in the console and select the <b>saas-factory-srvls-wrkshp-lab2</b> API from the list. This will display the various resources that define our system's REST API. In the list of resources, find the <b>/products</b> resource as follows:

<p align="center"><img src="../images/lab2/ProductsService.png" alt="Products Service"/></p>

To resolve our issue, we need to repair the DELETE method for this resource. Select DELETE to configure this method.

<b>Step 24</b> – Once you've selected DELETE, you see the details of the method configuration in a screen that appears as follows:

<p align="center"><img src="../images/lab2/DeleteMethod.png" alt="Delete Method"/></p>

<b>Step 25</b> – From here, click <b>Integration Request</b> at the top of the box that appears at the right. The integration request configures the mapping for our DELETE method. When you choose this option, you will see a screen similar to following:

<p align="center"><img src="../images/lab2/DeleteIntegrationRequests.png" alt="Delete Integration Request"/></p>

You may notice a warning icon next to the <b>Endpoint URL</b>. It turns out, our endpoint URL is missing a key qualifier. Select the pencil icon next to the Endpoint URL to edit the value. Append the <b>/{id}</b> path variable to your endpoint and <b>select the small checkmark icon</b> to save the value.

<p align="center"><img src="../images/lab2/DeleteMethodEndpointURL.png" alt="Integration Request Endpoint URL"/></p>

Now that our Endpoint URL has a placeholder for the path variable (the product id), we need to tell API Gateway how to map that to the request so it gets properly passed along to our backend application tier. Expand the <b>URL Path Parameters</b> section by clicking on the caret/triangle. Click on the <b>Add path</b> link and enter <b>id</b> for the <b>Name</b> of your parameter and <b>method.request.path.id</b> for the <b>Mapped from</b> value. Be sure to <b>select the small checkmark icon</b> to save your changes. This should have repaired the last broken piece of the puzzle.

<p align="center"><img src="../images/lab2/DeleteMethodPathParameters.png" alt="Integration Request Path Parameters"/></p>

<b>Step 26</b> - Before we can see our fix in action, we must redeploy our API. At the top of the screen, above the list of resources, select <b>Deploy API</b> from the <b>Actions</b> drop down menu.

<p align="center"><img src="../images/lab2/APIGatewayDeploy.png" alt="API Gateway Deploy API"/></p>

Select <b>v1</b> for the <b>Deployment stage</b> and click the <b>Deploy</b> button.

<p align="center"><img src="../images/lab2/APIGatewayDeployStage.png" alt="API Gateway Deploy Stage"/></p>

<b>Step 27</b> – To validate that our change worked, return to the serverless client application (using the URL from above) and sign in with the provided credentials. Now, attempt to delete the product and you should find that our change has resolved the issue.

<b>Step 28</b> – Finally, we should also note how this move to the API Gateway and a new UI model influenced the implementation of our application service. While our goal is to minimize changes to the monolith code, the interaction between the client and the monolith did change from an MVC model to a REST-based API. That meant that the controller portion of our MVC needed to be refactored to expose the REST API that is invoked through the API Gateway.

To view these changes, navigate to the Cloud9 services in the AWS console and open the IDE for this workshop. In the left-hand pane, open the <b>lab2/server/src/main/java</b> folder. This will open a series of folders that correspond to the Java package naming. Under the <b>saasfactory</b> folder you'll see an <b>api</b> folder that holds the new REST API for our services. Double-click on the <b>Products.java</b> file to open the API in the editor. The following is a snippet of code from this file:

```java
@CrossOrigin
@GetMapping(path = "/products/{id}")
public Product getProduct(@PathVariable Integer id) throws Exception {
    logger.info("Products::getProduct id = " + id);
    return productService.getProduct(id);
}

@CrossOrigin
@GetMapping(path = "/products")
public List<Product> getProducts() throws Exception {
    logger.info("Products::getProducts");
    return productService.getProducts();
}
```

While this code does have similarities to the code from our controller, you'll notice here that we've introduced annotations that declare our REST API entry points. Just like our page controllers, this code delegates to the business logic in our service classes, but now instead of integrating with an HTML templating framework, these methods return JSON strings to the client which now assumes responsibility for rendering the HTML. You'll also notice that we no longer need the controller classes or the HTML templates from the resources folder.

## Review

While it may not feel as though we've done much to enable migration to our serverless SaaS microservices, the work done in Lab 2 is one of the most fundamental bits of plumbing that has to get put into place to enable you to start thinking about carving out individual microservices.

With Lab 2, we put all the pieces of multi-tenancy in place without making huge changes to our monolithic application. We introduced onboarding and identity to enable tenant context to get introduced into our environment. We also created all the automated provisioning to create siloed instances of our monolith for each tenant with separate computing and storage for each tenant.

At the core of all this was a routing experience that enabled our system to route calls from our client to each tenant stack. Finally, we also made the move to a modern UI model, introducing a React application that is served up from S3. By extracting and committing to this new client model, we streamline and focus our efforts as we look to start creating serverless microservices. This prevents our services from being cluttered with worrying about server-rendered HTML.

You have now completed Lab 2.

[Continue to Lab 3](../lab3/README.md)

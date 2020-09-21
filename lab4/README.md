# Lab 4 – Extracting the Remaining Service - Goodbye Monolith!

At this stage, our migration to serverless SaaS is well underway. We have introduced onboarding, identity, and a model that supports side-by-side support for our new multi-tenant microservices and the existing monolith. For many organizations, this represents a key stage in their migration. Having created a microservice and demonstrated that this two-pronged approach can work, you can now focus your energy on carving out more and more services until the monolith is no longer needed.

이 단계 까지 서버리스 SaaS 로의 마이그레이션이 잘 진행되고 있습니다. 저희는 지금까지 새로운 멀티 테넌트 마이크로 서비스와 기존 모놀리스에 대한 병렬 연결을 지원하는 온보딩, Identity 모델을 도입했습니다. 이는 많은 조직에서 거치게 되는 핵심 단계를 들입니다. 여러분은 마이크로 서비스를 만들고 이 두 가지 접근 방식이 작동 할 수 있음을 확인 했으므로 이제 모놀리스가 더 이상 필요하지 않을 때까지 점점 더 많은 서비스를 구성하는 데 남은 세션을 사용할 수 있습니다.

In a real-world scenario, this transition would be a somewhat involved process where you would need to invest significant time to slowly migrate your business logic and data to a microservices model. However, we've limited the functionality of our sample monolith so that we can demonstrate what it would look like to migrate the remaining bits of functionality and completely eliminate the need for the monolith application tier.

실제 현장의 시나리오에서 이런 전환은 비즈니스 로직과 데이터를 마이크로 서비스 모델로 천천히 마이그레이션하는 데 상당한 시간을 투자해야하는 다소 복잡한 프로세스 입니다. 그러나 저희는 모놀리식에서 마이크로서비스 모델로의 기능적 전환 과정을 충실히 보여드리기 위해서 샘플 애플리케이션의 기능과 범위를 의도적으로 복잡하게 만들지 않았습니다.

To extract the final bits of functionality out of our monolith, we'll need to move the product service out to a standalone microservice. This will mimic much of what we discussed above. However, now that we have multiple microservices, we will also need to think about what it will mean to have code and concepts that are common to both of our microservices. We'll also look at another data partitioning model for this service, having our data represented in a relational database that can store data for all tenants in a shared construct (using what we call a <b>pool</b> model).

모놀리식 애플리케이션에서 기능의 마지막 부분을 추출하려면 제품 서비스를 마저 독립형 마이크로 서비스로 이동해야합니다. 그런데 이미 여러 마이크로 서비스가 있으므로 두 마이크로 서비스에 공통적 인 코드와 개념이 있다는 것이 의미하는 바도 고려해야합니다. 또한 이 서비스에 대한 또 다른 데이터 파티셔닝 모델을 살펴볼 것입니다. 이 데이터 파티셔닝 모델은 공유 구조안에 모든 테넌트의 데이터를 저장하는 관계형 데이터베이스를 바탕으로 하고 있습니다.(이를 <b>pool</b> model 이라고 함).

The architecture at this stage is much simpler. Gone is the need to provision full monolith stacks each time a tenant onboards. Instead, our tenants now all share the serverless microservices that can scale on their own and do not require separate provisioning for each tenant. The diagram below provides a high-level view of the new architecture:

이 단계의 아키텍처는 훨씬 간단합니다. 테넌트가 온 보딩 할 때마다 전체 모놀리스 스택을 프로비저닝 할 필요가 없습니다. 대신, 이제 테넌트는 모두 자체적으로 확장 할 수 있고 각 테넌트에 대해 별도의 프로비저닝이 필요하지 않은 서버리스 마이크로 서비스를 공유합니다. 아래 다이어그램은 새 아키텍처에 대한 개념적인 모습을 보여주고 있습니다:

<p align="center"><img src="../images/lab4/LogicalArchitecture.png" alt="Logical Architecture"/></p>

This diagram represents the completion of our migration process, highlighting that the functionality of our system is now supported by an API Gateway and two logical microservices (which are a composite of multiple Lambda functions).

이 다이어그램은 마이그레이션 프로세스의 완성된 모습을 나타내며, 시스템의 기능이 이제 API Gateway와 두 개의 논리적 마이크로 서비스 (여러 Lambda 함수의 복합)에서 지원되고 있음을 강조하고 있습니다.

## What You'll Be Building

The lab is all about getting your solution moved away from the monolith. The basic elements of making this final migration are as follows:

이번 실습은 솔루션을 최종적으로 모놀리스로 부터 완전히 떠나는 것을 목표로 합니다. 이 최종 마이그레이션을 수행하는 기본 항목들은 다음과 같습니다.

- We'll extract our last service from the monolith, the product service. This service will mostly move in the same fashion as the order service did in Lab 3. We will start with deploying the new Lambda functions for the serverless version of the product service.

- 모놀리스에서 마지막 서비스인 제품 서비스를 추출합니다. 이 서비스는 대부분 Lab 3의 주문 서비스와 동일한 방식으로 이동합니다. 제품 서비스의 서버리스 버전을 위한 새로운 Lambda 함수 배포부터 시작할것 입니다.

- Now that we have a second service, we can see that we need some way to extract and share core concepts that span our services. In the universe of serverless, we can achieve this through the use of Lambda layers. Layers provide a mechanism for Lambda functions to access shared code that can be deployed and versioned separately from the function itself. We'll move our existing token management code from the order service to a layer so we can share it between both the order and product services and we'll create new multi-tenant aware logging and metrics libraries and add those to our layer.

- 이제 마이크러 서비스 스택에 두 번째 서비스를 갖게 되었으므로 서비스 전반에 걸친 핵심 개념을 추출하고 이를 서비스 전반에 걸쳐 공유 할 방법이 필요하다는 것을 알 수 있습니다. 서버리스 세계에서는 Lambda layer을 사용하여 이를 달성 할 수 있습니다. layer는 Lambda 함수가 함수 자체와 별도로 배포하고 버전을 지정할 수있는 공유 코드에 액세스 할 수있는 메커니즘을 제공합니다(\*library 개념). 기존 토큰 관리 코드를 주문 서비스에서 layer으로 이동하여 주문 서비스와 제품 서비스 모두 함께 사용 할 수 있도록하고 새로운 멀티 테넌트 인식 로깅 및 메트릭 라이브러리를 만들어 layer에 추가 할 것입니다.

- Once we have these common concepts in our layers, we'll want to go back to our product and order services and add the instrumentation to use the code in these layers. We'll update our logging and and add some metrics instrumentation to illustrate how these new mechanisms remove all awareness of tenant context from developers.

- layer에 이러한 공통 코드를 갖게 되었으므로 제품 및 주문 서비스로 돌아가서 이러한 layer의 코드를 사용한 장치 하나 추가 하려고 합니다. 로깅을 업데이트하고 이러한 새로운 메커니즘이 개발자들로 하여금 테넌트 컨텍스트를 의식하지 않아도 되는 방법을 설명하기 위해 몇 가지 메트릭 장치를 추가 할 것입니다.

- As part of moving our service over, we also have to consider how we want to represent the product data. In this case, we'll use a <b>pool</b> model for the data where the tables in our relational database will hold the data for all tenants in a shared construct. A column in the table will be used to discriminate the data by tenant identifier.

- 서비스 마이그레이션의 일환으로 제품 데이터를 표현하는 방법도 고려해야합니다. 이 경우 관계형 데이터베이스의 테이블이 공유 구조의 모든 테넌트에 대한 데이터를 보유하는 <b>pool</b> model을 사용합니다. 테이블의 컬럼 하나는 테넌트 식별자에 의해 데이터를 구별하는 데 사용됩니다.

Once these steps are completed, we'll have the core elements of our migration completed. We'll also have layers in place that will simplify the introduction of new services as we move forward.

이 단계들이 완료되면 마이그레이션의 핵심 요소가 완료됩니다. 우리는 또한 앞으로 나아갈 때 새로운 서비스의 도입을 단순화 할 계층을 마련 할 것입니다.

## Step-By-Step Guide

The following is a breakdown of the step-by-step process for building out our new shared layers and moving the Product service from our monolith to a serverless microservice:

다음은 새로운 공유 계층을 구축하고 제품 서비스를 모놀리스에서 서버리스 마이크로 서비스로 이동하기위한 단계별 프로세스를 나눈 것입니다.

<b>Step 1</b> - Let's deploy our new product microservice. As before, we have a script to execute which will build and upload all the pieces required for our Lambda functions and then kick off a CloudFormation stack to provision everything. Run the following commands in the terminal windows of your Cloud9 IDE to launch this script:

<b>Step 1</b> - 새로운 제품(Product) 마이크로 서비스를 배포하겠습니다. 이전과 마찬가지로 Lambda 함수에 필요한 모든 부분을 빌드 및 업로드 한 다음 CloudFormation 스택을 시작하여 모든 것을 프로비저닝하는 스크립트를 실행할 수 있습니다. 이 스크립트를 시작하려면 Cloud9 IDE의 터미널 창에서 다음 명령을 실행하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh lab4.sh
```

This will trigger a cloud formation stack creation. <b>Before proceeding make sure that lab4 stack has been created successfully as follows</b>:

이 명령은 cloudformation 스택 생성을 시작시킬것 입니다. <b>진행하기전에, 아래와 같이 lab4의 스택이 성공적으로 생성되었는지 먼저 확인하세요</b>:

<p align="center"><img src="../images/lab4/CloudFormation.png" alt="CloudFormation"/></p>

<b>Step 2</b> - <b>You must confirm that the lab4 CloudFormation stack has completed successfully before continuing</b>. We will now update our React client to use the new endpoints created as part of above CloudFormation stack. Update your website by running following commands:

<b>Step 2</b> - <b>계속하기 전에 lab4 CloudFormation 스택이 성공적으로 완료되었는지 확인해야합니다</b>. 이제 위에 CloudFormation 스택의 일부로 생성 된 새 엔드 포인트를 사용하도록 React 클라이언트를 업데이트합니다. 다음 명령을 실행하여 웹 사이트를 업데이트하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh website-lab4.sh
```

<b>Step 3</b> – Now that we have a second microservice that will be moving into our architecture, we need to start thinking about how we can move common mechanisms that have tenant-context to a reusable mechanism. We could certainly use libraries and frameworks to do this. The question is, how do we make them reusable across our services? This is where Lambda layers come in. With layers we can separately deploy and version shared code that will support our need to hide away the details of multi-tenancy. We will start with some code we've already been using in our order service from Lab 3. We can now move the token manager class out of the order service and into a shared layer.

<b>Step 3</b> – 이제 아키텍처로 이동할 두 번째 마이크로 서비스가 준비되었므로 테넌트 컨텍스트를 가진 공통 메커니즘을 재사용 가능한 메커니즘으로 이동하는 방법에 대해 생각해야합니다. 이를 위해 우리는 확실히 라이브러리와 프레임 워크를 사용할 수 있습니다. 이때 문제는 어떻게 우리 서비스 전반에 걸쳐 재사용 할 수 있는가? 입니다. 이 부분에서 Lambda layer가 역할을 담당 합니다. layer을 사용하면 멀티 테넌시의 세부 사항을 숨길 수 있도록 지원하는 공유 코드를 별도로 배포하고 버전을 지정할 수 있습니다. lab 3의 주문 서비스에서 이미 사용하고있는 일부 코드로 시작해 보겠습니다. 이제 토큰 관리자 클래스를 주문 서비스에서 공유 layer로 이동할 수 있습니다.

The role of the token manager is to simplify acquiring the tenant identifier from the supplied JWT token. Layers allow our separate Lambda functions to follow software engineering best practices. This class is imported by the other classes in the layer to expand their functionality while minimizing code duplication. In your Cloud9 environment navigate to <b>lab4/layers/serverless-saas-layer/src/main/java/</b> and, inside the series of folders defining the Java package name, open <b>TokenManager.java</b> by double-clicking it. A portion of the code for this class is shown below.

토큰 관리자의 역할은 제공된 JWT 토큰에서 테넌트 식별자 획득 과정을 단순화하는 것입니다. layer을 통해 여러분은 Lambda 함수들을 분리함으로써 소프트웨어 엔지니어링 모범 사례를 따를 수 있습니다. 이 클래스는 코드 중복을 최소화하면서 기능을 확장하기 위해 layer의 다른 클래스에서 가져옵니다. Cloud9 환경에서 <b>lab4/layers/serverless-saas-layer/src/main/java /</b>로 이동하고 Java 패키지 이름을 정의하는 일련의 폴더 내에서 <b>TokenManager.java</b> 두 번 클릭합니다. 이 클래스에 대한 코드의 일부가 아래와 같습니다.

```java
private static final String TENANT_CLAIM = "custom:tenant_id";

public String getTenantId(Map<String, Object> event) {
    String bearerToken = ((Map<String, String>) event.get("headers")).get("Authorization");
    String jwtToken = bearerToken.substring(bearerToken.indexOf(" ") + 1);
    Claims verifiedClaims = Jwts.parser()
            .setSigningKeyResolver(keyResolver())
            .parseClaimsJws(jwtToken)
            .getBody();

    String tenantId = verifiedClaims.get(TENANT_CLAIM, String.class);

    if (tenantId == null) {
        throw new RuntimeException("No tenant id in token");
    }

    return tenantId;
}
```

The token manager extracts the bearer token from the Authorization header of the request event. It then decrypts the signed token and unpacks all of the claims (name/value pairs) embedded in the JWT. Once the token is unpacked, we get the tenant id from its claims by using the custom claim defined by our identity provider (Cognito in this case) and return this value.

토큰 관리자는 요청 이벤트의 Authorization 헤더에서 Bearer 토큰을 추출합니다. 그런 다음 서명 된 토큰을 해독하고 JWT에 포함 된 모든 클레임 (이름/값 쌍)의 압축을 풉니다. 토큰의 압축이 풀리면 ID 공급자 (이 경우 Cognito)가 정의한 사용자 지정 클레임을 사용하여 클레임에서 테넌트 ID를 가져오고 이 값을 반환합니다.

<b>Step 4</b> - With the token manger in place, we can start to introduce the other layer code that will reference the token manager. The first of these will be the logging manager. By moving our logging code to a layer, we can simply make logging calls and have the logging manager inject the tenant context. Not complicated, but good at hiding away tenant details from developers. Open the <b>LoggingManager.java</b> file in Cloud9 by double-clicking it. A portion of the code for this class is shown below.

<b>Step 4</b> - 토큰 관리자가 준비되면 토큰 관리자를 참조 할 다른 layer 코드를 도입 할 수 있습니다. 이들 중 첫 번째는 로깅 관리자입니다. 로깅 코드를 layer 로 이동하면 로깅 호출을 수행하고 로깅 관리자가 테넌트 컨텍스트를 주입하도록 할 수 있습니다. 복잡하지는 않지만 개발자로 부터 테넌트 세부 정보를 숨기는데 좋은 방법입니다. Cloud9에서 <b>LoggingManager.java</b> 파일을 두 번 클릭하여 엽니다. 이 클래스에 대한 코드의 일부가 아래에 나와 있습니다.

```java
public static void info(Map<String, Object> event, String message) {
    LOGGER.info(decorateMessage(event, message));
}

private static String decorateMessage(Map<String, Object> event, String message) {
    return "Tenant ID [" + new TokenManager().getTenantId(event) + "] " + message;
}
```

Notice that our LoggingManager depends on the TokenManager to extract the tenant id from the incoming request. Our new LoggingManager will ensure that all of our log messages always include tenant context which will help with monitoring and debugging of our multi-tenant system.

LoggingManager는 TokenManager에 의존하여 수신 요청(incoming request)에서 테넌트 ID를 추출합니다. 새로운 LoggingManager는 모든 로그 메시지에 항상 멀티 테넌트 시스템의 모니터링 및 디버깅에 도움이되는 테넌트 컨텍스트가 포함되도록합니다.

<b>Step 5</b> – The last item we're going to add to our layer is a metrics manager. This helper is used to publish metrics data with tenant context. Recording and publishing metrics is essential to SaaS organizations. Although what we have here is rather simple and lightweight, we wanted to show the importance of adding tenant-aware metrics to your SaaS solutions. Open the <b>MetricsManager.java</b> file to see the simple metrics manager we've created for this workshop. A portion of the code for this class is shown below.

<b>Step 5</b> – 레이어에 추가 할 마지막 항목은 메트릭 관리자입니다. 이 도우미는 테넌트 컨텍스트로 메트릭 데이터를 게시하는 데 사용됩니다. 지표 기록 및 게시는 SaaS 조직에 필수적입니다. 여기에있는 것은 다소 간단하고 가볍지 만 SaaS 솔루션에 테넌트 인식 메트릭을 추가하는 것의 중요성을 보여주고 싶었습니다. <b> MetricsManager.java </b> 파일을 열어 이 워크숍을 위해 만든 간단한 측정 항목 관리자를 확인하세요. 이 클래스에 대한 코드의 일부가 아래에 나와 있습니다.

```java
public static void recordMetric(Map<String, Object> event, String source, String action, Long duration) {
    String tenantId = new TokenManager().getTenantId(event);

    Map<String, Object> metric = new HashMap<>();
    metric.put("tenantId", tenantId);
    metric.put("source", source);
    metric.put("action", action);
    metric.put("duration", duration);
    metric.put("utc", Instant.now().truncatedTo(ChronoUnit.MILLIS).toString().substring(0, 23).replace('T', ' '));

    String json;
    try {
        json = MAPPER.writerWithDefaultPrettyPrinter().writeValueAsString(metric);
    } catch (JsonProcessingException e) {
        LoggingManager.error(event, getFullStackTrace(e));
        json = e.getMessage();
    }

    LoggingManager.log(event, "MetricsManager::recordMetric\n" + json);
}
```

This code also utilizes the TokenManager to help build the data for each metric. Our metric is a simple execution time for a given source and action. We will send this metric data to a log file for this workshop, but you can imagine how you'd integrate this with a data collection pipeline using tools such as Amazon SQS or Amazon Kinesis Data Firehose.

이 코드는 또한 TokenManager를 사용하여 각 메트릭을 위한 데이터를 만듭니다. 여기서 사용하는 우리의 메트릭은 단순히 주어진 소스와 액션에 대한 실행 시간입니다. 이 지표 데이터를 이 워크숍을 위해 단순하게 로그 파일로 보내 겠지만, 실 세계에서는 Amazon SQS 또는 Amazon Kinesis Data Firehose와 같은 도구를 사용하여 이 메트릭을 데이터 수집 파이프 라인과 통합하는 방법을 생각해볼 수 도 있습니다.

<b>Step 6</b> – You may be wondering why we're putting all 3 of our helpers into a single layer. Why not separate them into their own layers? While it is tempting to keep functionality separate, there is a relatively low limit to how many layers you can have per Lambda function. It is best practice to package your layer code as needed to stay under the limit.

<b>Step 6</b> – 여러분은 아마 우리가 왜 세 개의 helper 를 모두 단일 layer에 배치하는지 그 이유가 궁금 할 수 있습니다. 왜 그들 자신의 layer로 분리하지 않을까요? 기능을 별도로 유지하고 싶지만 Lambda 함수 당 보유 할 수있는 layer 수에는 상대적으로 낮은 제한(limit)이 있습니다. 제한을 유지하기 위해 필요에 따라 layer 코드를 패키징하는 것이 가장 좋습니다.

The deployment script you ran at the beginning of this lab deployed our layer and wired it up to the product service Lambda functions. The order service functions, however, were deployed as a step in the previous lab. Now that we've reviewed our multi-tenant layer code, we need to update our order service to take advantage of it. Enter the following commands in the terminal window of the Cloud9 IDE to configure our order service functions to use the new layer:

이 실습을 시작할 때 실행 한 배포 스크립트는 layer를 배포하고 이를 제품 서비스 Lambda 함수에 연결 합니다. 그러나 주문 서비스 기능은 이전 lab의 단계로 이미 배포되었습니다. 이제 멀티 테넌트 layer 코드를 검토 했으므로 이를 활용하기 위해 주문 서비스를 업데이트해야 합니다. Cloud9 IDE의 터미널 창에 다음 명령을 입력하여 새 layer를 사용하도록 주문 서비스 기능을 구성합니다:

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/lab4/order-service
sh update-layers.sh
```

<b>Step 7</b> – To verify that our layers were successfully deployed, let's open the Lambda service in the AWS console. Once the Lambda page is open, select the <b>Layers</b> option from the left-hand side of the page. This will display a list of layers that should include our multi-tenant token, logging and metrics code. The view will appear as follows:

<b>Step 7</b> – layer가 성공적으로 배포되었는지 확인하기 위해 AWS 콘솔에서 Lambda 서비스를 열어 보겠습니다. Lambda 페이지가 열리면 페이지 왼쪽에서 <b>Layers</b> 옵션을 선택합니다. 그러면 멀티 테넌트 토큰, 로깅 및 메트릭 코드를 포함해야하는 레이어 목록이 표시됩니다. 보기는 다음과 같이 나타납니다.

<p align="center"><img src="../images/lab4/Layers.png" alt="Layers"/></p>

<b>Step 8</b> - We should confirm that the layer has been integrated with our functions. Click on the <b>Functions</b> link in the left hand menu of the Lambda service console. Enter <b>saas-factory-srvls-wrkshp-orders</b> into the filter box above the list of functions to narrow this list. Select any of the order service functions to open the detail page for the function. In the <b>Designer</b> section, click on the <b>Layers</b> icon under the function name and you'll see that our layer has been merged with the function.

<b>Step 8</b> - layer가 함수들과 통합되었는지 확인해야합니다. Lambda 서비스 콘솔의 왼쪽 메뉴에서 <b>Functions</b> 링크를 클릭합니다. 이 목록의 범위를 좁히려면 함수 목록 위의 필터 상자에 <b>saas-factory-srvls-wrkshp-orders</b>를 입력합니다. 주문 서비스 기능을 선택하여 기능에 대한 세부 사항 페이지를여십시오. <b>Designer</b> 섹션에서 함수 이름 아래의 <b>Layers</b> 아이콘을 클릭하면 layer가 함수와 병합 되었음을 알 수 있습니다.

<p align="center"><img src="../images/lab4/LambdaLayer.png" alt="Layers"/></p>

<b>Step 9</b> – It's time to look at our new serverless version of our product service. Migrating the product service was the last part of breaking up the monolith. This required both creating a multi-tenant version of the service as well as the database. Let's start with the service code. In Cloud9, open <b>lab4/product-service/src/main/java</b> and, inside the series of folders defining the Java package name, open <b>ProductService.java</b> by double-clicking it. You'll see that this class underwent a similar transition from the monolith REST API controller class from Lab 2 as the order service did. This class contains the various methods that implement the GET, PUT, POST, and DELETE methods for our product service - each of which will be deployed as a separate Lambda function to make up our logical product service. Here's a snippet of the code.

<b>Step 9</b> – 이제 우리의 제품 서비스의 새로운 서버리스 버전을 살펴볼 때입니다. 제품 서비스 마이그레이션은 모놀리스 해체의 마지막 부분이었습니다. 이를 위해서는 서비스의 멀티 테넌트 버전과 데이터베이스를 모두 만들어야했습니다. 서비스 코드부터 시작하겠습니다. Cloud9에서 <b>lab4/product-service/src/main/java</b>를 열고 Java 패키지 이름을 정의하는 일련의 폴더 내에서 두 번 클릭하여 <b>ProductService.java</b>를 엽니다. 이 클래스는 주문 서비스와 마찬가지로 Lab 2의 모놀리식 REST API 컨트롤러 클래스와 유사한 전환을 거쳤음을 알 수 있습니다. 이 클래스에는 제품 서비스에 대한 GET, PUT, POST 및 DELETE 메서드를 구현하는 다양한 메서드가 포함되어 있습니다. 각 메서드는 논리적 제품 서비스를 구성하기 위해 별도의 Lambda 함수로 배포됩니다. 다음은 코드 일부 입니다.

```java
public APIGatewayProxyResponseEvent getProducts(Map<String, Object> event, Context context) {
    LoggingManager.log(event, "ProductService::getProducts");
    List<Product> products = DAL.getProducts(event);
    APIGatewayProxyResponseEvent response = new APIGatewayProxyResponseEvent()
            .withStatusCode(200)
            .withHeaders(CORS)
            .withBody(toJson(products));
    return response;
}
```

The code in this service and the order service have both been updated to take advantage of our new shared Lambda layer. The main difference visible is the change to our logging calls. Where our prior version called a static class logger with a message, our new product and order services now call our <b>LoggingManager</b> from the layer, supplying the <b>event</b> from the incoming request that contains the JWT token.

이 서비스의 코드와 주문 서비스는 모두 새로운 공유 Lambda layer를 활용하도록 업데이트 되었습니다. 눈에 보이는 주요 차이점은 로깅 호출의 변경입니다. 이전 버전이 메시지와 함께 static class logger 라고 했던 곳에서, 새로운 제품 및 주문 서비스는 이제 layer에서 JWT 토큰이 포함된 수신 요청(incoming request)로 부터 <b>이벤트</b>를 전달하며<b>LoggingManager</b>를 호출합니다.

<b>Step 10</b> – We also migrated the database for the product service from the single-tenant monolith representation to our microservice model. As we did with the Order service, we must consider how the Product service will want to represent its data in a multi-tenant model. This means thinking about performance, isolation, and so on. We decided that, for products, we'd demonstrate another flavor of data partitioning, using a <b>pooled</b> model where the data for tenants would co-exist in the same tables of a relational database (isolated by a column with a tenant identifier). This means that our service will use the same database for all tenants and won't require us to provision new tables or other constructs as new tenants are introduced.

<b>Step 10</b> – 또한 단일 테넌트 모놀리식에서 마이크로 서비스 모델로 제품 서비스 용 데이터베이스를 마이그레이션 했습니다. 주문 서비스와 마찬가지로 제품 서비스가 멀티 테넌트 모델에서 데이터를 표현하는 방법을 고민해야만 합니다. 이것은 성능, 테넌트 격리 등에 대해 생각하는 것을 의미합니다. 우리는 제품의 경우 테넌트 데이터가 관계형 데이터베이스의 동일한 테이블 (컬럼으로 분리됨)에 테넌트 데이터가 공존하는 <b>pooled</b> 모델을 사용하여 데이터 파티셔닝의 또 다른 특징을 보여 주고자 했습니다(테넌트 구분자가 포함된 컬럼으로 격리하는). 즉, 이제 우리 서비스는 모든 테넌트에 대해 동일한 데이터베이스를 사용하고 새 테넌트가 도입 될 때 새 테이블이나 기타 구성을 프로비저닝 할 필요가 없습니다.

In many respects, the code for interacting with the multi-tenant database does not change that much from the silo version (since they were both working against a relational database and the schema is mostly unchanged). Let's look at some SQL code that illustrates the minor tweaks that are needed to move from our monolith database to our pooled, multi-tenant representation. Open <b>ProductServiceDAL.java</b> in Cloud9 by double-clicking on the filename. Here are a couple of lines from that file.

여러 측면에서 멀티 테넌트 데이터베이스와 상호 작용하는 코드는 사일로 버전에서 크게 변경되지 않았습니다 (둘 다 관계형 데이터베이스에 대해 작업하고 스키마가 대부분 변경되지 않았기 때문에). 모놀리식 데이터베이스에서 풀링 된 멀티 테넌트 표현으로 이동하는 데 필요한 사소한 조정을 보여주는 몇 가지 SQL 코드를 살펴 보겠습니다. 파일 이름을 두 번 클릭하여 Cloud9에서 <b>ProductServiceDAL.java</b>를 엽니 다. 다음은 해당 파일의 일부 코드 입니다.

```java
private final static String INSERT_PRODUCT_SQL = "INSERT INTO product (tenant_id, sku, product, price) VALUES (?, ?, ?, ?)";
private final static String SELECT_CATEGORY_SQL = "SELECT category_id, category FROM category WHERE tenant_id = ?";
```

These statements show how we have to add reference to a <b>tenant_id</b> column when we query the shared database to partition the data between our multiple tenants.

이 쿼리문은 여러 테넌트간에 데이터를 분할하기 위해 공유 데이터베이스를 쿼리 할 때 <b>tenant_id</b> 컬럼에 대한 참조를 추가하는 방법을 보여줍니다.

<b>Step 11</b> – To confirm that our service is actually running in a multi-tenant fashion, let's verify that the data partitioning and the logging messages that we've focused on are landing where we expect. Re-open the application (using the same CloudFront URL we captured above). Be sure to <b><i>refresh your browser</i></b> to force it to pull down a new copy of our React app.

<b>Step 11</b> – 서비스가 실제로 멀티 테넌트 방식으로 실행되고 있는지 확인하기 위해 데이터 파티셔닝 및 로깅 메시지가 예상한 위치에 있는지 확인하겠습니다. 위에서 캡처 한 것과 동일한 CloudFront URL을 사용하여 애플리케이션을 다시 엽니다. <b><i>브라우저를 새로 고침</i></b>하여 React 앱의 새 사본을 강제로 가져 오도록합니다.

Sign-in to the application and select the <b>Products</b> link from the navigation at the top of the page. Now select the <b>Add Product</b> button and enter new product information into the Add Product form. When you finish, select the <b>Add Product</b> button at the bottom of the form to save the new product information.

애플리케이션에 로그인하고 페이지 상단의 탐색에서 <b>Products</b> 링크를 선택합니다. 이제 <b>Add Product</b> 버튼을 선택하고 제품 추가 양식에 새 제품 정보를 입력하십시오. 완료되면 양식 하단의 <b>Add Product</b> 버튼을 선택하여 새 제품 정보를 저장하십시오.

<b>Step 12</b> – Open the CloudWatch service in the AWS console and select <b>Log groups</b> from the menu on the left. Enter <b>/aws/lambda/saas-factory-srvls-wrkshp-products-insert</b> in the <b>Filter</b> box at the top of the screen and select the most recent Log Stream. If you look closely, you'll see log entries from our LoggingManager prefixed with the tenant id. You will also see log entries that did not use our tenant-aware logger and they will lack the tenant id prefix.

<b>Step 12</b> – AWS 콘솔에서 CloudWatch 서비스를 열고 왼쪽 메뉴에서 <b>Log groups</b>을 선택합니다. 화면 상단의 <b>Filter</b> 상자에 <b>/aws/lambda/saas-factory-srvls-wrkshp-products-insert</b>를 입력하고 최신 로그 스트림을 선택합니다. 자세히 살펴보면 Tenant ID가 접두사로 붙은 LoggingManager의 로그 항목을 볼 수 있습니다. 테넌트 인식 로거를 사용하지 않은 로그 항목도 표시되며 Tenant ID 접두사가 없습니다.

<p align="center"><img src="../images/lab4/LoggingManager.png" alt="LoggingManager"/></p>

<b>Step 13</b> - As a final activity for this workshop, let's exercise our tenant-aware MetricsManager. Back in Cloud9, open <b>ProductService.java</b> and navigate to the <b>insertProduct</b> method. We'll wrap the lines of the method with some simplistic timing code and then submit that data to our MetricsManager. Your insertProduct method should look like the following.

<b>Step 13</b> - 이 워크숍의 마지막 활동으로 테넌트 인식 MetricsManager를 살펴 보겠습니다. Cloud9로 돌아가 <b>ProductService.java</b>를 열고 <b>insertProduct</b> 메소드로 이동합니다. 간단한 타이밍 코드로 메서드 줄을 래핑 한 다음 해당 데이터를 MetricsManager에 제출합니다. insertProduct 메소드는 다음과 같아야합니다.

```java
public APIGatewayProxyResponseEvent insertProduct(Map<String, Object> event, Context context) {
    long startTimeMillis = System.currentTimeMillis();

    LoggingManager.log(event, "ProductService::insertProduct");
    APIGatewayProxyResponseEvent response = null;
    Product product = productFromJson((String) event.get("body"));
    if (product == null) {
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(400);
    } else {
        product = DAL.insertProduct(event, product);
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(200)
                .withHeaders(CORS)
                .withBody(toJson(product));
    }

    long totalTimeMillis = System.currentTimeMillis() - startTimeMillis;
    MetricsManager.recordMetric(event, "ProductService", "insertProduct", totalTimeMillis);

    return response;
}
```

Be sure to save your changes using the Ctrl-S keyboard shortcut or selecting <b>Save</b> from the <b>File</b> menu in Cloud9.

Ctrl-S 키보드 단축키를 사용하거나 Cloud9의 <b>File</b> 메뉴에서 <b>Save</b>을 선택하여 변경 사항을 저장해야합니다.

<b>Step 14</b> - Now we need to redeploy our product service. Enter the following commands to rebuild and deploy our Lambda functions.

<b>Step 14</b> - 이제 제품 서비스를 다시 배포해야합니다. 다음 명령을 입력하여 Lambda 함수를 다시 빌드하고 배포합니다.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/lab4/product-service
sh update-service.sh
```

<b>Step 15</b> - Return to the application using the same URL as before and add a new product.

<b>Step 15</b> - 이전과 동일한 URL을 사용하여 애플리케이션으로 돌아가서 새 제품을 추가합니다.

<b>Step 16</b> - The addition of the new product should have triggered our MetricsManager layer feature. Let's go to CloudWatch in the AWS console and see if we can find it. Again, select <b>Log groups</b> from the menu on the left and enter <b>/aws/lambda/saas-factory-srvls-wrkshp-products-insert</b> in the <b>Filter</b> box at the top of the screen and select the most recent Log Stream. You should find a log entry with a metric data packet (formatted in JSON) similar to the following.

<b>Step 16</b> - 새 제품을 추가하면 MetricsManager layer 기능이 실행되었을 것입니다. AWS 콘솔의 CloudWatch로 이동하여 찾을 수 있는지 확인하겠습니다. 다시 왼쪽 메뉴에서 <b>Log groups</b>을 선택하고 <b>Filter</b>에서 <b>/aws/lambda/saas-factory-srvls-wrkshp-products-insert</b>를 입력합니다. 그리고 가장 최근의 로그 스트림을 선택합니다. 다음과 유사한 지표 데이터 패킷 (JSON 형식)이있는 로그 항목이 확인되어야 합니다.

<p align="center"><img src="../images/lab4/MetricsManager.png" alt="MetricsManager"/></p>

## Review

This lab was the final step in our move away from the monolith. We moved the last bit of the monolith, the Product service, to a serverless microservice. This eliminated our need to route between the monolith and the microservices and left the system in a fully multi-tenant model. As part of this move, we also took measures to standardize our approach to implementing multi-tenant microservices, introducing layers that would be used to centralize the logging, metrics, and token management that is used by all microservices in our system.

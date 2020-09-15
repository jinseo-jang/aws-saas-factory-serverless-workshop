# Lab 3 – Carving Out Our First Multi-Tenant Serverless Microservice

Now that we have an automated onboarding process and multi-tenant identity in place, we can start to focus more of our energy on decomposing our monolith into multi-tenant microservices. Our goal is to carve out individual services from our monolith, slowly extracting functionality from the monolith and to come up with mechanisms that enable some of our application to remain in the monolith while other bits of it are running as true multi-tenant serverless microservices.

이제 자동화 된 온보딩 프로세스와 멀티 테넌트 indentity가 준비되었으므로 모놀리스를 다중 테넌트 마이크로 서비스로 분해하는 데 보다 더 집중할 수 있습니다. 우리의 목표는 모놀리스에서 개별 서비스를 분리하고, 모놀리스에서 천천히 기능을 추출하고, 일부 애플리케이션이 모놀리스에 남아있는 동안 다른 부분이 진정한 멀티 테넌트 서버리스 마이크로 서비스로 실행될 수 있도록하는 메커니즘을 만드는 것입니다.

A key part of this strategy is to support a routing scheme that enables side-by-side execution of the microservice and monolith services. The idea here is that we will have a single unified API for our services that will then route traffic to the appropriate target service (a Lambda function or an entry point in our monolith). The following diagram provides a conceptual view of the approach we'll be taking to carving out this first service.

이 전략의 핵심 부분은 마이크로 서비스 및 모놀리식 서비스의 병렬 실행을 가능하게 하는 라우팅 체계를 지원하는 것입니다. 여기서 아이디어는 서비스에 대한 단일 통합 API를 갖게되고 트래픽을 적절한 대상 서비스 (Lambda 함수 또는 모놀리스의 진입 점)로 라우팅한다는 것입니다. 다음 다이어그램은 이 첫 번째 서비스를 구성하기 위해 선택한 접근 방식에 대한 개념적 모습입니다.

<p align="center"><img src="../images/lab3/LogicalArchitecture.png" alt="Logical Architecture"/></p>

In this diagram, you'll notice now that we have two separate paths for our tenants. As tenants enter the API Gateway, they can either be routed to our new Order service, or they can be routed to the monolith application tier. It's important to note that the flows will be different if you are consuming the new microservices (Lambda functions) or the application tier. Since our Order service is a pooled multi-tenant service, it will process requests for all tenants. Meanwhile, the monolith will still require a separate deployment for each tenant. This means our siloed monoliths will also require the routing rules to direct traffic to the appropriate application tier (using Application Load Balancer routing rules).

이 다이어그램에서 테넌트에 대한 두 개의 별도 경로가 있음을 알 수 있습니다. 테넌트가 API Gateway에 들어가면 새로운 주문 서비스로 라우팅되거나 모놀리식 애플리케이션 계층으로 라우팅 될 수 있습니다. 새로운 마이크로서비스(Lambda 함수) 또는 애플리케이션 계층을 사용하는 경우 흐름이 달라진다는 점에 유의해야합니다. 주문 서비스는 Pooled multi-tenant 서비스이므로 모든 테넌트에 대한 요청을 함께 처리합니다. 한편, 모놀리스에는 각 테넌트에 대해 별도의 배포가 여전히 필요합니다. 즉, 사일로화된 모놀리스에는 트래픽을 적절한 애플리케이션 계층으로 전달하기위한 라우팅 규칙도 필요합니다 (Application Load Balancer 라우팅 규칙 사용).

In this lab we will enable the conceptual view you see above. We'll introduce and deploy our new multi-tenant Order service in a serverless model. We've stayed with Java as the language for the individual functions of our Order service. However, you could imagine that a migration to a serverless model might also involve a switch in languages. There are a wide range of factors that could influence your choice. For this exercise though, it seemed to make sense to stick with one language to make it easier to follow the transition from monolith to serverless.

이 실습에서는 상기 개념적 아키텍처를 구현할 것 입니다. 이를 위해 서버리스 모델에서 새로운 멀티 테넌트 주문 서비스를 도입하고 배포 할 것입니다. 우리는 주문 서비스의 개별 기능을 위한 언어로 Java를 그대로 사용했습니다. 그러나 서버리스 모델로의 마이그레이션에는 때에 따라 언어의 변경도 포함될 수도 있습니다(\*선택에 영향을 줄 수있는 다양한 요인이 있습니다). 하지만 이 연습 에서는 실습의 목적상 그대로 Java를 사용 합니다.

Generally, as we've broken services out of our monolith, we're trying to create a class file that represents a logical microservice. The idea here is that, just like our "service" layer code from the monolith, we'll still have a class with the different methods of our solution. We will then deploy the different methods of our class as individual Lambda functions. This model makes it easy for us to move our code over from the monolith without major changes. However, this is only the case because we already had a clear notion of services in our monolith. In many cases, the monolith will not break apart so cleanly. We also have to begin to leverage the tenant context that flows into this service, using it to partition data, add logging context, and so on.

일반적으로 모놀리스에서 서비스를 분리 했으므로 논리적인 하나의 마이크로 서비스를 나타내는 클래스 파일을 만들려고합니다. 여기서 아이디어는 모놀리스의 "서비스" 레이어 코드와 마찬가지로 솔루션의 다른 메서드를 가진 클래스를 계속 보유한다는 것입니다. 그런 다음 클래스의 다양한 메서드를 개별 Lambda 함수로 배포합니다. 이 모델을 사용하면 큰 변경 없이 모놀리스에서 코드를 쉽게 이동할 수 있습니다. 그러나 이것은 우리가 이미 모놀리스에서 서비스에 대한 명확한 개념을 가지고 만들었기 때문에 가능하지 대부분의 경우 모놀리스는 그렇게 깨끗하게 분리되지 않습니다. 또한 이 서비스로 유입되는 테넌트 컨텍스트를 활용하여 데이터를 분할하고 로깅 컨텍스트를 추가하는 등의 작업을 시작해야합니다.

We should note that this migration will not dig into the details of the data migration aspects of this problem. While these are an important part of the broader migration story, they are considered out of scope for this effort. So, as we carve out our new services and introduce new multi-tenant storage constructs, we'll be starting with a blank slate (leaving whatever data exists behind in the monolith database).

이 마이그레이션은 데이터 마이그레이션 측면의 세부 사항은 깊이 살펴보지 않을 것입니다. 물론 데이터 마이그레이션 역시 중요한 부분이지만 실습의 목적상 깊이 다루지 않습니다. 따라서 새로운 서비스를 쪼개고 새로운 다중 테넌트 스토리지 구성을 도입 할 때 빈 슬레이트 (모놀리스 데이터베이스에 존재하는 모든 데이터를 남겨 둡니다)로 시작합니다.

## What You'll Be Building

This lab is all about getting our first service carved out of our application tier. We'll focus on looking at what it means to build a serverless version of this service. This also means looking into how the service is deployed and configured within our infrastructure to support the model described above. The following is a breakdown of the key elements of the lab:

이 실습은 애플리케이션 계층에서 첫 번째 서비스를 만들어 내는 것입니다. 이 서비스의 서버리스 버전을 구축하는 것이 무엇을 의미하는지 살펴 보겠습니다. 이는 또한 위에서 설명한 모델을 지원하기 위해 서비스가 인프라 내에서 배포되고 구성되는 방법을 살펴 본다는 것을 의미합니다. 다음은 실습의 핵심 요소를 요약한 것입니다.

- Our first step will be to actually create the Lambda functions that makeup our Order service. We'll review how our Java gets converted to a function-based approach and describe the basic exercise of getting these new functions deployed.

- 첫 번째 단계는 주문 서비스를 구성하는 Lambda 함수를 실제로 생성하는 것입니다. Java가 함수 기반 접근 방식으로 어떻게 변환되는지 검토하고 이러한 새로운 함수를 배포하는 기본 연습을 합니다.

- The API Gateway will also require changes to support our new functions (along with the monolith tier). We'll look at how the API Gateway will route specific Order service calls to the Lambda functions in our service and connect the dots between the development experience and the run-time environment.

- API Gateway는 모놀리스 계층을 함께 지원하기 위해 변경해야합니다. API Gatewat가 특정 주문 서비스 호출을 서비스의 Lambda 함수로 라우팅하고 Lambda 개발 경험과 런타임 환경 등 을 살펴 보겠습니다.

- As part of extracting the order service, we must also think about how our data will be represented in a multi-tenant model. More specifically, we'll need to pick a storage technology that best fits the isolation, performance, and scale needs of the Order service and introduce a data partitioning scheme. Upon completion of these steps, you should be well on your way to a full serverless SaaS model. With one service migrated, you can now begin to think about how you would carve out the remaining services from the application tier. We look forward to that exercise in Lab 4.

- 주문 서비스 추출의 일환으로 데이터가 멀티 테넌트 모델에서 어떻게 표현 될 것인지도 고려해야 합니다. 더 구체적으로 말하자면 주문 서비스의 격리, 성능 및 확장 요구 사항에 가장 적합한 스토리지 기술을 선택하고 데이터 파티셔닝 체계를 도입해야합니다. 이러한 단계를 완료하면 완전한 서버리스 SaaS 모델로의 전환이 가능합니다. 하나의 서비스가 마이그레이션되면 이제 애플리케이션 계층에서 나머지 서비스를 분할하는 방법에 대해 생각할 수 있습니다. 이 부분에 대해서는 Lab 4에서 더 다룹니다.

## Step-By-Step Guide

The following is a breakdown of the step-by-step process for getting our new multi-tenant Order service deployed in a serverless model.

다음은 서버리스 모델에 배포 된 새로운 멀티 테넌트 주문 서비스를 만들기 위한 단계별 프로세스 입니다.

<b>Step 1</b> – Let's start by finding the code that will start with for our Order service. Open the Cloud9 IDE in the AWS console and select the <b>Serverless SaaS Workshop IDE</b>. With the IDE, we can now examine the code that currently exists for our Order service. To get there, open the <b>lab3</b> folder in the left-hand pane of the page. Under the <b>order-service/src/main/java</b> you will find the various Java files that makeup our Order service. Double click <b>OrderService.java</b> to open it in the editor window. Here's a snippet of that file:

<b>Step 1</b> – 주문 서비스에서 시작할 코드 부터 찾는 것 으로 시작하겠습니다. AWS 콘솔에서 Cloud9 IDE를 열고 <b>Serverless SaaS Workshop IDE</b>를 선택합니다. IDE를 사용하여 현재 주문 서비스에 대해 존재하는 코드를 검사 할 수 있습니다. 이동하려면 페이지 왼쪽 창에서 <b>lab3</b> 폴더를 엽니 다. <b>order-service/src/main/java</b>에서 주문 서비스를 구성하는 다양한 Java 파일을 찾을 수 있습니다. <b> OrderService.java </b>를 두 번 클릭하여 편집기 창에서 엽니다. 다음은 해당 파일의 일부입니다.

<p align="center"><img src="../images/lab3/OrderServiceCode.png" alt="Order Service Code"/></p>

This class contains a series of Lambda function event handlers which corresponde to the CRUD operations of our Order Service.

이 클래스에는 주문 서비스의 CRUD 작업에 해당하는 일련의 Lambda 함수 이벤트 핸들러가 포함되어 있습니다.

<b>Step 2</b> - Let's deploy this new microservice and see it in action. We've introduced a script that will be responsible for uploading and publishing changes to your microservices (or, in this case, publishing it for the first time). Run the following commands in the terminal windows of your Cloud9 IDE to launch this script:

<b>Step 2</b>-이제 새로운 마이크로 서비스를 배포하고 실제로 작동하는지 살펴 보겠습니다. 이를 위해 저희는 마이크로 서비스에 대한 변경 사항을 업로드 및 게시 (또는이 경우 처음 게시)하는 스크립트를 도입했습니다. 이 스크립트를 시작하려면 Cloud9 IDE의 터미널 창에서 다음 명령을 실행하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh lab3.sh
```

This will trigger a cloud formation stack creation. <b>Before proceeding make sure that lab3 stack has been created successfully as follows</b>:

이 명령문은 cloud formation 스택 생성을 트리거 합니다. <b>계속 진행 하기전에 아래와 같이 lab3 stack이 성송적으로 생성되었는지 확인하십시오.</b>

<p align="center"><img src="../images/lab3/CloudFormation.png" alt="Cloud Formation"/></p>

<b>Step 3</b> - <b>You must confirm that the lab3 CloudFormation stack has completed successfully before continuing</b>. We will now update our React client to use the new endpoints created as part of above CloudFormation stack. Update your website by running following commands:

<b>Step 3</b> - <b>계속하기 전에 lab3 CloudFormation 스택이 성공적으로 완료되었는지 확인해야합니다</b>. 이제 위에서 CloudFormation 스택의 일부로 생성 된 새로운 엔드 포인트를 사용하도록 React 클라이언트를 업데이트합니다. 다음 명령을 실행하여 웹 사이트를 업데이트하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/resources
sh website-lab3.sh
```

<b>Step 4</b> – Now we can go verify that our new Order service has been deployed. Open the Lambda service within the AWS console. You'll be presented with a list of all of your functions. Enter <b>saas-factory-srvls-wrkshp-orders</b> into the filter box above the list of functions to narrow this list to those that we've deployed. Here you'll notice that there are separate functions for each of the operations of our Order service. Each one corresponds to a REST operation (GET, PUT, DELETE, etc.). This confirms that our service has been deployed.

<b>Step 4</b> – 이제 새로운 주문 서비스가 배포되었는지 확인할 수 있습니다. AWS 콘솔에서 Lambda 서비스를 엽니다. 모든 함수 목록이 표시됩니다. 함수 목록 위의 필터 상자에 <b>saas-factory-srvls-wrkshp-orders</b>를 입력하여 목록을 좁힙니다. 여기서 우리는 주문 서비스의 각 작업 별로 별도 함수가 있음을 알 수 있습니다. 각각은 REST 작업 (GET, PUT, DELETE 등)에 해당합니다. 이것으로 우리 서비스가 배포되었음을 확인했습니다.

<p align="center"><img src="../images/lab3/LambdaFunctions.png" alt="Lambda Functions"/></p>

<b>Step 5</b> – The functions are in place. Now we need to verify that the API Gateway has mapped an entry to these functions to route traffic to this serverless microservice (instead of the monolith we were using before). Navigate to the API Gateway service in the AWS console. Select the <b>saas-factory-srvls-wrkshp-lab3</b> API from the list. This should display a list of resources that are configured for this API that appears as follows:

<b>Step 5</b> – 관련 lambda 함수가 위치해 있습니다. 이제 API Gateway가 이전에 사용했던 모놀리스 대신이 서버리스 마이크로 서비스로 트래픽을 라우팅하기 위해 이러한 함수에 entry point를 매핑했는지 확인해야합니다. AWS 콘솔에서 API Gateway 서비스로 이동합니다. 목록에서 <b>saas-factory-srvls-wrkshp-lab3</b> API를 선택합니다. 그러면 다음과 같이 해당 API에 대해 구성된 리소스 목록이 표시됩니다.

<p align="center"><img src="../images/lab3/APIGatewayOrderService.png" alt="Order Service"/></p>

Here you'll see the basic CRUD operations that are enabled as resources in our API Gateway. There are GET and POST resources which don't require parameters. There is also a /{id} route that adds an identifier to the resource to enable GET, DELETE, and PUT operations on individual orders.

여기에는 API Gateway에서 리소스로 활성화 된 기본 CRUD 작업이 표시됩니다. 매개 변수가 필요하지 않은 GET 및 POST 리소스가 있습니다. 또한 리소스에 식별자를 추가하여 개별 주문에 대한 GET, DELETE 및 PUT 작업을 활성화하는 /{id} 경로가 있습니다.

<b>Step 6</b> – Now we can verify that these REST resource methods are mapped entry to these functions to route traffic to this serverless microservice (instead of the monolith we were using before). Select the <b>GET</b> method under the <b>/orders</b> resource to access the configuration for the GET orders operation. You will see that this method shows <b>LAMBDA_PROXY</b> for the Integration Request type. This will display a view similar to the following:

<b>Step 6</b> – 이제 이러한 REST 리소스 메서드가 이전에 사용했던 모놀리스 대신 이 서버리스 마이크로 서비스로 트래픽을 라우팅하기 위해 이러한 함수에 대한 항목을 매핑했는지 확인할 수 있습니다. GET 주문 작업에 대한 구성에 액세스하려면 <b>/orders</b> 리소스 아래에서 <b>GET</b> 메서드를 선택합니다. 이 메서드는 Integration Reqeust type에 대해 <b>LAMBDA_PROXY</b>를 표시하는 것을 볼 수 있습니다. 다음과 유사하게 표시될겁니다.

<p align="center"><img src="../images/lab3/OrderService.png" alt="Order Service"/></p>

<b>Step 7</b> – Finally, we can verify that the correct Lambda function is the one being proxied. Select the <b>Integration Request</b> title from the top of the box at the right. This will display a view similar to the following:

<b>Step 7</b> – 마지막으로 올바른 Lambda 함수가 프록시되는 함수인지 확인할 수 있습니다. 오른쪽 상자 상단에서 <b>Integration Request</b> 제목을 선택합니다. 다음과 유사한 화면이 나타날겁니다.

<p align="center"><img src="../images/lab3/OrderServiceIntegrationRequest.png" alt="Order Service Integration Request"/></p>

The key piece of information here is the Lambda Function. The value for this attribute should map directly to our getOrders() Lambda function that we deployed earlier. The function will be named <b>saas-factory-srvls-wrkshp-orders-get-all-[REGION]</b>.

여기서 핵심 정보는 Lambda 함수입니다. 이 속성의 값은 이전에 배포 한 getOrders() Lambda 함수에 직접 매핑되어야 합니다. 함수의 이름은 <b>saas-factory-srvls-wrkshp-orders-get-all-[REGION]</b>입니다.

<b>Step 8</b> – We've confirmed that our API Gateway and Lambda functions appear to have landed and been configured correctly. Now let's go perform an operation in the application to verify that these new functions are working. We intentionally didn't carry forward any data for the Order Service from our monolith database so we're starting with an empty database.

<b>Step 8</b> – API Gateway 및 Lambda 함수가 제대로 설정되고 구성되었는지 확인했습니다. 이제 애플리케이션에서 작업을 수행하여 이러한 새로운 기능이 작동하는지 확인하겠습니다. 모놀리식 데이터베이스에서 주문 서비스에 대한 데이터를 의도적으로 전달하지(마이그레이션) 않았으므로 빈 데이터베이스로 시작합니다.

Open the application using the URL you've previously saved for your modern application and sign-in with one of the tenants you're previously created. Before we can add an order to our system, we must have at least one product in our catalog. Click on the <b>Products</b> link in the navigation header to confirm that you have a product. If you do not have any products, add one now.

최신 애플리케이션 용도로 이전에 저장 한 URL을 사용하여 애플리케이션을 열고 이전에 생성 한 테넌트 중 하나로 로그인합니다. 시스템에 주문을 추가하기 전에 카탈로그에 하나 이상의 제품이 있어야합니다. 탐색 헤더에서 <b>Products</b> 링크를 클릭하여 제품이 있는지 확인하십시오. 제품이 없는 경우 지금 추가하십시오.

<b>Step 9</b> - Now, navigate to the <b>Orders</b> menu item in the navigation header and select <b>Add Order</b> from the page that is displayed. Fill out the form that is displayed with some mock order information and save the order by selecting the <b>Add Order</b> button. Oops! What happened? Let's go take a look at our Order service to see what went wrong.

<b>Step 9</b> - 이제 탐색 헤더의 <b>Orders</b> 메뉴 항목으로 이동하고 표시되는 페이지에서 <b>Add Order</b>를 선택합니다. 모의 주문 정보로 표시된 양식을 작성하고 <b>Add Order</b> 버튼을 선택하여 주문을 저장합니다. 아마도 에러가 발생할 겁니다. 무엇이 잘못되었는지 확인하기 위해 주문 서비스를 살펴 보겠습니다.

<p align="center"><img src="../images/lab3/404Error.png" alt="404 Error"/></p>

<b>Step 10</b> – Let's start by examining the code of our Order service. Open the Cloud9 IDE in the AWS console and select the <b>Serverless SaaS Workshop IDE</b>. Using the file tree in the left-hand window pane, open the <b>lab3/order-service/src/main/java</b> path to see the different classes that make up the Order service. Double-click on the <b>OrderService.java</b> file to view its contents and locate the <b>insertOrder</b> menthod within the Java class. The function will appear as follows:

<b>Step 10</b> – 먼저 주문 서비스의 코드를 살펴 보겠습니다. AWS 콘솔에서 Cloud9 IDE를 열고 <b>Serverless SaaS Workshop IDE</b>를 선택합니다. 왼쪽 창에있는 파일 트리를 사용하여 <b>lab3/order-service/src/main/java</b> 경로를 열어 주문 서비스를 구성하는 여러 클래스를 확인합니다. <b>OrderService.java</b> 파일을 두 번 클릭하여 해당 컨텐츠를 보고 Java 클래스 내에서 <b>insertOrder</b> 메소드를 찾으십시오. 메소드는 다음과 같이 나타납니다.

```java
public APIGatewayProxyResponseEvent insertOrder(Map<String, Object> event, Context context) {
    LOGGER.info("OrderService::insertOrder");

    APIGatewayProxyResponseEvent response = null;
    Order order = fromJson((String) event.get("body"));
    if (order == null) {
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(400);
    } else {
        // TODO Add code to call DAL.insertOrder here and return a 200
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(404)
                .withHeaders(CORS);
    }
    return response;
}
```

In looking at this function more closely, you'll discover that the function isn't actually finished. There's a to-do note saying we need to call the data access layer to actually save the order object. Currently, the method is returning an HTTP 404 error.

이 함수를 좀 더 자세히 살펴보면 함수가 실제로 완료되지 않은 것을 알 수 있습니다. 실제로 주문 객체를 저장하기 위해 데이터 액세스 레이어를 호출해야한다는 메모가 주석으로 있습니다. 현재이 메서드는 HTTP 404 오류를 반환합니다.

<b>Step 11</b> – To get our insertOrder() method working, we'll need to add the code that inserts an order into the database. Copy and paste the following code so your insertOrder method matches.

<b>Step 11</b> – insertOrder() 메서드가 작동하도록 하려면 주문을 데이터베이스에 삽입하는 코드를 추가해야합니다. insertOrder 메소드가 다음과 일치 하도록 다음 코드를 복사하여 붙여 넣으십시오.

```java
public APIGatewayProxyResponseEvent insertOrder(Map<String, Object> event, Context context) {
    LOGGER.info("OrderService::insertOrder");

    APIGatewayProxyResponseEvent response = null;
    Order order = fromJson((String) event.get("body"));
    if (order == null) {
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(400);
    } else {
        order = DAL.insertOrder(event, order);
        response = new APIGatewayProxyResponseEvent()
                .withStatusCode(200)
                .withHeaders(CORS)
                .withBody(toJson(order));
    }
    return response;
}
```

This code extracts the order data from our incoming request. If the order is empty, we return with an Invalid Request status code of 400. Otherwise, we call our data access layer (DAL) to insert the order into the database and return a status code of 200.

이 코드는 수신 요청에서 주문 데이터를 추출합니다. 주문이 비어 있으면 유효하지 않은 요청 상태 코드 400으로 반환됩니다. 그렇지 않으면 데이터 액세스 계층 (DAL)을 호출하여 주문을 데이터베이스에 삽입하고 상태 코드 200을 반환합니다.

<b>Step 12</b> – Be sure to save your changes with the Ctrl-S keyboard shortcut or by selecting <b>Save</b> from the <b>File</b> menu in Cloud9. With our new code introduced, we now need to deploy this updated function to the Lambda service. Run the following commands to invoke this update:

<b>Step 12</b> – Ctrl-S 키보드 단축키를 사용하거나 Cloud9의 <b>File</b> 메뉴에서 <b>Save</b>을 선택하여 변경 사항을 저장해야합니다. 새로운 코드가 도입 되었으므로 이제이 업데이트 된 함수를 Lambda 서비스에 배포해야합니다. 이 업데이트를 호출하려면 다음 명령을 실행하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/lab3/order-service/
sh update-service.sh
```

<b>Step 13</b> – The updated version of our insertOrder() function is now in place. Let's go open the application again (using the URL you retrieved earlier), login using the tenant you had used earlier, and access the <b>Orders</b> item in the menu. Select the <b>Add Order</b> button from the page and enter an order into the form that is displayed. When you are done entering the order select the <b>Add Order</b> button from the form. You should now see that your order appears in the list of orders on the page, confirming that our fix appears to have worked.

<b>Step 13</b> – 이제 insertOrder() 함수의 업데이트 된 버전이 준비되었습니다. 이전에 검색 한 URL을 사용하여 애플리케이션을 다시 열고 이전에 사용한 테넌트를 사용하여 로그인 한 다음 메뉴에서 <b>Orders</b> 항목에 액세스 하겠습니다. 페이지에서 <b>Add Order</b> 버튼을 선택하고 표시되는 양식에 주문을 입력합니다. 주문 입력이 완료되면 양식에서 <b>Add Order</b> 버튼을 선택하십시오. 이제 페이지의 주문 목록에 주문이 표시되어 수정 사항이 제대로 작동하는지 확인해야 합니다.

<p align="center"><img src="../images/lab3/AddOrderSuccess.png" alt="Add Order"/></p>

<b>Step 14</b> – As part of moving to this new microservice, we also had to remove our dependency on the database monolith where orders had previously been shared in one large database used by all services. Extracting this data from the monolith is essential to our microservices story. Each of our microservices must own the data that it manages to limit coupling and enable autonomy. When we move this data out of the monolith, it also gives us the opportunity to determine what service and multi-tenant storage strategy will best fit the multi-tenant requirements of our microservice.

<b>Step 14</b> – 이 새로운 마이크로 서비스로 이동하는 과정에서 이전에 모든 서비스에서 사용하는 하나의 대규모 데이터베이스에서 주문이 공유되었던 데이터베이스 모놀리스에 대한 종속성도 제거해야했습니다. 모놀리스에서 이 데이터를 추출하는 것은 마이크로 서비스 스토리에 필수적입니다. 각 마이크로 서비스는 결합을(coupling) 제한하고 데이터 자율성(autonomy)을 확보하기 위해 관리하는 데이터를 직접 소유하는게 좋습니다. 이 데이터를 모놀리스 밖으로 이동하면 마이크로 서비스 기반의 멀티 테넌트 요구 사항에 가장 적합한 서비스 및 멀티 테넌트 스토리지 전략을 결정할 수있는 기회가 제공될 것 입니다.

In this case, we're looking at how we want to represent our order data that will be managed by our order management microservice. Should we silo the data for each tenant? Should it be pooled (share a common table/database)? What are its isolation requirements? These are all questions we need to answer. For this solution, we've decided to move the order data to DynamoDB and use a NoSQL representation. However, for isolation reasons, we've opted to put the data in separate tables for each tenant. Below is a conceptual model of the data representation for the order service:

이 경우 주문 관리 마이크로 서비스에서 관리 할 주문 데이터를 나타내는 방법을 살펴 봅니다. 각 테넌트에 대한 데이터를 사일로 (silo)해야 합니까? 풀링(공통 테이블/데이터베이스 공유) 해야 합니까? 격리 요구 사항은 무엇입니까? 이것들은 우리가 답해야 할 모든 질문입니다. 이 솔루션의 경우 주문 데이터를 DynamoDB로 이동하고 NoSQL 표현을 사용하기로 결정했습니다. 그러나 격리상의 이유로 각 테넌트에 대해 별도의 테이블에 데이터를 저장하도록 선택했습니다. 다음은 주문 서비스에 대한 데이터 표현의 개념적 모델입니다.

<p align="center"><img src="../images/lab3/ConceptualModel.png" alt="Conceptual Model"/></p>

To see this in action, let's now go look at the data that was added via our new order microservice. Navigate to the DynamoDB service in the AWS console and select the <b>"Tables"</b> item from the navigation pane on the left. This will display a list of DynamoDB tables, including any tables for any tenant that have orders. Below is a sample that includes a few tenants table that were created:

이것이 실제로 작동하는지 확인 하기 위해 이제 새로운 주문 마이크로 서비스를 통해 추가 된 데이터를 살펴 보겠습니다. AWS 콘솔에서 DynamoDB 서비스로 이동하고 왼쪽 탐색 창에서 <b>"Tables"</b> 항목을 선택합니다. 그러면 주문이있는 테넌트의 테이블을 포함하여 DynamoDB 테이블 목록이 표시됩니다. 다음은 생성 된 몇 가지 테넌트 테이블이 포함 된 샘플 입니다.

<p align="center"><img src="../images/lab3/DynamoDBTables.png" alt="DynamoDB Tables"/></p>

You'll see that there are two tables here, one for each tenant in our system. Your list of tables will vary based on the tenants you've introduced.

여기에는 시스템의 각 테넌트 당 하나 씩 두 개의 테이블이 있습니다. 테이블 목록은 가입한 테넌트에 따라 다릅니다.

<b>Step 15</b> – Now we need to verify that the order we created landed in a DynamoDB table. Select the table named <b>order*fulfillment*[TENANT_ID]</b>. Once you select your table, you'll get a view with a list of tabs with information about your table. Select the <b>Items</b> tab to view the list of the items in that table. The view will be similar to the following:

<b>Step 15</b> – 이제 생성 한 주문이 DynamoDB 테이블에 들어 갔는지 확인해야합니다. <b>order_fulfillment\_[TENANT_ID]</b>라는 테이블을 선택합니다. 테이블을 선택하면 테이블에 대한 정보가 포함 된 탭 목록이있는 보기가 표시됩니다. 해당 테이블의 항목 목록을 보려면 <b>Items</b> 탭을 선택하십시오. 보기는 다음과 유사합니다.

<p align="center"><img src="../images/lab3/ItemsTable.png" alt="Items Table"/></p>

In this example, our table had one order item. You can drill into any item in this list to get more detail on that item.

이 예에서 테이블에는 하나의 주문 항목이 있습니다. 이 목록의 항목을 드릴 하면 해당 항목에 대한 자세한 정보를 얻을 수 있습니다.

<b>Step 16</b> – This tenant-per-table partitioning model uses the context of the current tenant identifier (passed in the JWT token) to generate our table name. Let's look at how this is resolved in the code of our application service. Go back to the files for the lab 3 order service in Cloud8. Open the Cloud9 IDE in the AWS console and select <b>Serverless SaaS Workshop IDE</b>. Using the file tree in the left-hand window pane, open the <b>lab3/order-service/src/main/java</b> path to see the different classes that make up the Order service. Double-click on the <b>OrderServiceDAL.java</b> file and look at the <b>insertOrder()</b> method. The code will appear as follows:

<b>Step 16</b> – 이 테이블 당 테넌트 파티셔닝 모델은 현재 테넌트 식별자 (JWT 토큰으로 전달됨)의 컨텍스트를 사용하여 테이블 이름을 생성합니다. 애플리케이션 서비스 코드에서 이것이 어떻게 해결되는지 살펴 보겠습니다. Cloud9의 Lab 3 주문 서비스 파일로 돌아갑니다. AWS 콘솔에서 Cloud9 IDE를 열고 <b>Serverless SaaS Workshop IDE</b>를 선택합니다. 왼쪽 창에있는 파일 트리를 사용하여 <b>lab3/order-service/src/main/java</b> 경로를 열어 주문 서비스를 구성하는 여러 클래스를 확인합니다. <b>OrderServiceDAL.java</b> 파일을 두 번 클릭하고 <b>insertOrder()</b> 메소드를보십시오. 코드는 다음과 같이 나타납니다.

```java
public Order insertOrder(Map<String, Object> event, Order order) {
    UUID orderId = UUID.randomUUID();
    LOGGER.info("OrderServiceDAL::insertOrder " + orderId);
    order.setId(orderId);

    try {
        Map<String, AttributeValue> item = DynamoDbHelper.toAttributeValueMap(order);
        String tableName = tableName(event);

        PutItemResponse response = ddb.putItem(request -> request.tableName(tableName).item(item));
    } catch (DynamoDbException e) {
        LOGGER.error("OrderServiceDAL::insertOrder " + getFullStackTrace(e));
        throw new RuntimeException(e);
    }

    return order;
}
```

Within this method, you'll see a call to a helper method called <b>tableName</b>, supplying the context of the request in the <b>event</b> parameter. This event parameter contains our JWT token with our tenant context.

이 메서드 내에서 <b>event</b> 매개 변수안의 요청 컨텍스트를 제공하는 <b>tableName</b>이라는 helper 메서드에 대한 호출을 볼 수 있습니다. 이 이벤트 매개 변수에는 테넌트 컨텍스트가 있는 JWT 토큰이 포함됩니다.

Let's look at this implementation of tableName. Scroll down in the OrderServiceDAL.java file until you find the implementation for the tableName method. The code is as follows:

이 tableName 메소드 구현을 살펴 보겠습니다. tableName 메소드에 대한 구현을 찾을 때까지 OrderServiceDAL.java 파일에서 아래로 스크롤하십시오. 코드는 다음과 같습니다.

```java
private String tableName(Map<String, Object> event) {
    String tenantId = new TokenManager().getTenantId(event);
    String tableName = "order_fulfillment_" + tenantId;

    if (!tenantTableCache.containsKey(tenantId) || !tenantTableCache.get(tenantId).equals(tableName)) {
        boolean exits = false;
        ListTablesResponse response = ddb.listTables();
        for (String table : response.tableNames()) {
            if (table.equals(tableName)) {
                exits = true;
                break;
            }
        }
        if (!exits) {
            CreateTableResponse createTable = ddb.createTable(request -> request
                    .tableName(tableName)
                    .attributeDefinitions(AttributeDefinition.builder().attributeName("id").attributeType(ScalarAttributeType.S).build())
                    .keySchema(KeySchemaElement.builder().attributeName("id").keyType(KeyType.HASH).build())
                    .provisionedThroughput(ProvisionedThroughput.builder().readCapacityUnits(5L).writeCapacityUnits(5L).build())
            );
            waitForActive(tableName);
        }
        tenantTableCache.put(tenantId, tableName);
    }

    return tenantTableCache.get(tenantId);
}
```

You'll notice that the very first thing this code does is make a call to the TokenManager to get our current tenant identifier from the supplied JWT token. It then creates and table name that is the concatenation of <b>order*fulfillment*</b> and the tenant id we retrieved. This ensures that each table name is unique for each tenant. If a DynamoDB table with that name doesn't yet exist, one is created on-the-fly.

이 코드가 수행하는 첫 번째 작업은 제공된 JWT 토큰에서 현재 테넌트 식별자를 가져 오기 위해 TokenManager를 호출하는 것입니다. 그런 다음 <b>order_fulfillment\_</b>와 검색 한 테넌트 ID를 연결 한 테이블 이름을 생성합니다. 이렇게 하면 각 테이블 이름이 각 테넌트에 대해 고유해 집니다.. 해당 이름의 DynamoDB 테이블이 아직 존재하지 않으면 즉시 생성됩니다.

<b>Step 17</b> – In order to simplify our ability (for this lab) to identify activity for tenant, we're going to add a bit of logging detail to our order service to include this information. Scroll back up to the insertOrder() method in our OrderServiceDAL.java file. Let's add a logging statement after retrieving the table name for the current tenant. Add this line to your method right after calling the tableName method and right before getting the PutItemResponse from DynamoDB.

<b>Step 17</b> – 이 실습에서 테넌트의 활동을 식별하는 기능을 단순화하기 위해 이 정보를 포함하도록 주문 서비스에 약간의 로깅 세부 정보를 추가 할 것입니다. OrderServiceDAL.java 파일에서 insertOrder() 메소드를 다시 스크롤하십시오. 현재 테넌트의 테이블 이름을 검색 한 후 로깅 문을 추가해 보겠습니다. tableName 메서드를 호출 한 직후와 DynamoDB에서 PutItemResponse를 가져 오기 직전에이 줄을 메서드에 추가합니다.

```java
LOGGER.info("OrderServiceDAL::insertOrder TableName = " + tableName);
```

Your insertOrder method should look similar to this:
추가하면 다음과 같은 메소드 코드가 되어야 합니다:

<p align="center"><img src="../images/lab3/AddLoggingToInsertOrder.png" alt="Logging Statement"/></p>

<b>Step 18</b> – Be sure to save your changes with the Ctrl-S keyboard shortcut or by selecting <b>Save</b> from the <b>File</b> menu in Cloud9. With our new code introduced, we now need to deploy this updated function to the Lambda service. Run the following commands to invoke this update:

<b>Step 18</b> – Ctrl-S 키보드 단축키를 사용하거나 Cloud9의 <b>File</b> 메뉴에서 <b>Save</b>을 선택하여 변경 사항을 저장해야 합니다. 새로운 코드가 도입 되었으므로 이제 이 업데이트 된 함수를 Lambda 서비스에 배포해야합니다. 이 업데이트를 호출하려면 다음 명령을 실행하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/lab3/order-service/
sh update-service.sh
```

<b>Step 19</b> – The last step in validating our change is to run the actual application and verify that our new logging call is recording the tenant table name. Open the application (using same CloudFront URL as before), sign-in with your credentials, and access the Orders link at the top of the page. Select the <b>Add Order</b> button from the orders page and enter a new order into the form. Now select <b>Add Order</b> on the new order form to save the order.

<b>Step 19</b> – 변경 사항을 확인하는 마지막 단계는 실제 애플리케이션을 실행하고 새 로깅 호출이 테넌트 테이블 이름을 기록하고 있는지 확인하는 것입니다. 이전과 동일한 CloudFront URL을 사용하여 애플리케이션을 열고 자격 증명으로 로그인 한 다음 페이지 상단의 주문 링크에 액세스합니다. 주문 페이지에서 <b>Add Order</b> 버튼을 선택하고 양식에 새 주문을 입력합니다. 이제 새 주문 양식에서 <b>Add Order</b>를 선택하여 주문을 저장하십시오.

<b>Step 20</b> – Finally, to see the impact of our change, we'll need to view the log files for our function. Open the CloudWatch service in the AWS console and select <b>Log groups</b> from the navigation pane on the left of the page. This will display a list of multiple log groups. To narrow the list, enter <b>/aws/lambda/saas-factory-srvls-wrkshp-orders-insert</b> into the filters box at the top of the function list. Now, select the function name shown in the list. This will display a list of log streams for the selected function that will be similar to the following:

<b>Step 20</b> – 마지막으로 변경의 결과를 확인하려면 함수에 대한 로그 파일을 확인해야합니다. AWS 콘솔에서 CloudWatch 서비스를 열고 페이지 왼쪽의 탐색 창에서 <b>Log Group</b>을 선택합니다. 그러면 여러 로그 그룹 목록이 표시됩니다. 목록의 범위를 좁히려면 함수 목록 상단의 필터 상자에 <b>/aws/lambda/saas-factory-srvls-wrkshp-orders-insert</b>를 입력합니다. 이제 목록에 표시된 함수 이름을 선택하십시오. 그러면 다음과 유사한 선택한 기능에 대한 로그 스트림 목록이 표시됩니다.

<p align="center"><img src="../images/lab3/CloudWatchLogs.png" alt="CloudWatch Logs"/></p>

<b>Step 21</b> – Click on the top log stream to access the log file contents. Once you're in the log, you'll need to search for your newly inserted log file. Ultimately, you will be able to locate the DynamoDB table name that was associated with the order creation that you performed.

<b>Step 21</b> – 로그 파일 내용에 액세스하려면 상단 로그 스트림을 클릭합니다. 로그에 들어가면 새로 삽입 한 로그 파일을 검색해야합니다. 궁극적으로 수행 한 주문 생성과 관련된 DynamoDB 테이블 이름을 찾을 수 있습니다.

<p align="center"><img src="../images/lab3/CloudWatchLogStream.png" alt="CloudWatch Log Stream"/></p>

## Review

This lab represented a key next step in our migration process. We now started the gradual move to decompose our monolith into microservices. The key element of this model is our ability to run these new microservices side-by-side with code that remains in our monolith. In this scenario, we separated out the Order service, moving it to a serverless microservice. We also moved the data it manages over to the new environment. Our API Gateway was then setup to direct traffic selective to the microservice or the monolith (depending on which functionality was being accessed).

이 실습은 마이그레이션 프로세스의 주요 단계를 담았습니다. 이제 모놀리스를 마이크로 서비스로 분해하는 점진적인 마이그레이션을 시작했습니다. 이 모델의 핵심 요소는 모놀리스에 남아있는 코드와 함께 이러한 새로운 마이크로 서비스를 나란히 실행할 수있는 체계를 만드는 것이었습니다. 이 시나리오에서는 Order 서비스를 분리하여 서버리스 마이크로 서비스로 이동했습니다. 또한 관리하는 데이터를 새로운 환경으로 옮겼습니다. 그런 다음 API Gateway를 설정하여 트래픽을 선택적으로 마이크로 서비스 또는 모놀리스로 전달했습니다 (액세스중인 기능에 따라 다름).

You have now completed Lab 3.

여기까지 여러분은 Lab 3을 마쳤습니다.

[Continue to Lab 4](../lab4/README.md)

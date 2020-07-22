# Lab 1 – Deploying, Exploring, and Exercising the Single-Tenant Monolith

Our transformation process must begin with our baseline monolithic application. We need a point of reference that employs common monolithic patterns that represent our starting point for migrating to microservices and multi-tenant SaaS. For our solution, we've picked a fairly common Java-based technology stack that should be similar to many of the monolithic solutions that have been commonly employed by different organizations. Although this example uses Java, there's little about this stack that is unique to Java. A .NET monolith, for example, would likely have a footprint very similar to what we have created.

저희가 진행할 전환 프로세스는 저희가 만든 기본 모노리식 애플리케이션과 함께 시작 해야합니다. 따라서 마이크로서비스 및 멀티테넌트 SaaS로 마이그레이션 하기 위한 시작점으로 사용될 일반적인 모놀리식 패턴을 사용하는 아키텍처가 필요합니다. 저희는 이를 위해 다른 조직에서도 일반적으로 많이 사용하는 일반적인 Java 기반 기술 스택을 선택했습니다. 이 예제는 Java를 사용하지만 Java에 고유 한이 스택은 거의 없습니다. 예를 들어 .NET 모놀리식 애플리케이션 역시 우리가 만든 것과 매우 유사한 형태가될 것 입니다.

The architecture of our monolith is based on a traditional EC2-based model where the web and application tiers of our system are hosted on a collection of load-balanced, scalable instances. While these web and application layers are logically separated within the code, they are deployed collectively as one monolithic unit. Any change to the web experience or the business logic would require a complete redeployment of the code. This is a classic monolithic challenge. Occasionally, we see the web tier separate from the application tier to allow separate scale and deployment, however, with server-side HTML rendering, this is not common. The data for each customer is stored in a single database that houses all of the system's data. The conceptual footprint of this environment is shown in the image below:

저희가 만든 모놀리스 아키텍처는 시스템의 웹 및 애플리케이션 계층이 로드 밸런싱되고 확장 가능한 인스턴스들로 호스팅되는 전통적인 EC2 모델을 기반으로 합니다. 이러한 웹 및 애플리케이션 계층은 코드 내에서 논리적으로 분리되어 있지만 하나의 모놀리식 단위로 배포됩니다. 웹 경험이나 비즈니스 로직을 변경하려면 하나의 모노리식 단위의 코드를 완전히 재 배포 해야만 합니다. 이것은 고질적으로 모놀리식 아키텍처가 갖는 문제 입니다. 때때로 웹 계층이 애플리케이션 계층과 분리되어 별도의 확장 및 배포가 가능하지만 서버 측 HTML 렌더링에서는 이것이 일반적이지 않습니다. 각 고객의 데이터는 모든 시스템 데이터를 저장하는 단일 데이터베이스에 저장됩니다. 이상의 구조에 대한 영역을 개념적으로 나타낸 이미지가 아래에 있습니다.:

<p align="center"><img src="../images/lab1/LogicalArchitecture.png" alt="Architecture Overview"/></p>

You'll notice that we've shown two copies of this environment. This is to convey the idea that every customer of this monolithic system will be deployed into a complete standalone copy of the infrastructure. This is a common strategy for most single-tenant independent software vendors (ISVs). In these solutions, each customer has their own unique installation. This usually means separate support, potentially different versions, and different management and operations teams for each customer.

여러분들은 똑같은 환경이 두 개 존재한다는것을 눈치챘을 겁니다. 이는 모놀리식 시스템이 모든 고객들이 완전히 독립된 인프라 복사본에 배포 될 것이라는 아이디어를 전달하기위한 것입니다. 이는 대부분의 단일 테넌트 ISV 에게 공통적으로 나타나는 전략입니다. 이런 솔루션들에서는 각 고객마다 설치 형태를 갖습니다. 이는 일반적으로 고객마다 별도의 지원, 잠재적으로 다른 버전 및 다른 관리 및 운영 팀이 필요하다는 것을 의미 합니다.

Lab 1 starts with all the moving parts of this monolith provisioned. We'll look briefly at some of the key elements of the deployed architecture to give you a glimpse of the underlying components. We'll then exercise the monolith as a user might to demonstrate the working application and experience we'll be migrating to serverless microservices.

실습 1은 프로비전된 모놀리스의 모든 구성들과 함께 시작됩니다. 배포 된 아키텍처의 핵심 요소 중 일부를 간단히 살펴보고 기본 구성 요소를 간략하게 살펴 보겠습니다. 그런 다음 사용자가 실제로 애플리케이션을 사용하고 서버리스 마이크로 서비스로 전환된 환경을 경험하도록 모놀리스를 바탕으로 살펴볼 것 입니다.

## What You'll Be Building

This lab is a mix of familiarizing yourself with both the underlying architecture as well as the simplified application of the monolith that we'll be migrating to a serverless, multi-tenant model. The following is a high-level breakdown of the steps that you will be performing in this lab:

- Provision and explore the monolithic environment – at the outset of the lab, you'll have fully provisioned infrastructure to support the single-tenant starting point. You will set up your development environment, including an IDE, and trigger the CI/CD pipeline to deploy the monolithic application into the infrastructure. You will then be taken through some basic steps in the console to examine what has been provisioned.
- Exercise the monolithic application – it's important to have some exposure to the application that is being transformed. We'll bring up the application and exercise a few aspects of it and look at the logs and data that are generated.
- Examine the code of the monolith - it is also important to expose you to enough of the implementation details of the monolith that you'll have a good reference as we begin to carve it up into microservices.

At the end of this lab, you'll still have a single-tenant, monolith experience. However, this first step sets the stage for our transformation of the application to a modern architecture.

이 실습의 목적은 서버리스 멀티테넌트 모델로 마이그레이션 할 모놀리스 애플리케이션 뿐만 아니라 기본 아키텍처를 살펴보는데 있습니다. 다음은이 실습에서 여러분이 수행 할 단계들을 개략적으로 나타낸 것입니다.

- <b>단일 테넌트 환경을 프로비저닝</b> - 여러분은 이번 전체 실습의 시작점이 될 단일 테넌트 인프라를 완전히 프로비저닝 할 것입니다. IDE를 포함한 개발 환경을 설정하고 CI / CD 파이프 라인을 트리거하여 모놀리식 애플리케이션을 인프라에 배포할 것입니다. 그런 다음 콘솔에서 몇 가지 기본 단계를 수행하여 프로비저닝 된 내용을 검사할 것 입니다.
- <b>모놀리식 애플리케이션 연습</b> – 전환될 애플리케이션을 먼저 살펴보는 것이 중요 합니다. 애플리케이션을 불러 와서 몇 가지 측면을 살펴보고 생성 된 로그와 데이터를 살펴 보겠습니다.
- <b>모놀리식 애플리케이션 코드 평가</b> - 또한 모놀리식 애플리케이션의 세부 구현 사항을 충분히 살펴보는것 또한 중요합니다. 왜냐하면 마이크로 서비스로 전환 적용 하기 시작할 때 참조 할만 한 정보들을 제공 하기 때문입니다.

이 실습이 끝난 후에도 여전히 단일 테넌트, 모놀리식 경험이 제공될 것 입니다. 그러나 이 첫 번째 단계는 애플리케이션을 모던 아키텍처로 변환하는 기본 바탕을 만드는 단계 입니다.

## Step-By-Step Guide

The setup of this workshop provisioned all the supporting infrastructure for your Java-based, single-tenant monolith application. This includes the VPC and other networking and security components as well as the EC2 instances and the RDS database instances. To get started we need to get our Java application deployed to this infrastructure so we can begin to exercise its features. The following steps will guide your through this process.

이 워크샵 셋업은 Java 기반의 단일 테넌트 모놀리스 애플리케이션을 위한 모든 지원 인프라를 프로비저닝 했습니다. 여기에는 VPC 및 기타 네트워킹 및 보안 구성 요소와 EC2 인스턴스 및 RDS 데이터베이스 인스턴스가 포함됩니다. 시작하려면 Java 애플리케이션을 이 인프라에 배포해야만 애플리케이션 기능을 사용할 수 있습니다. 다음 단계는 그 과정을 안내합니다.

<b>Step 1</b> – We want to simulate the developer experience as best we can, so we've started this lab with a baseline of infrastructure that has no application deployed. To get things moving, open the IDE that will be used throughout this workshop. We will use Amazon Cloud9 as our IDE for this workshop. To open Cloud9 search for the service in the AWS console, or find it listed under the Developer Tools category.

<b> Step 1 </b> – 저희는 최대한 개발자 경험을 시뮬레이션 하고자 합니다. 따라서 애플리케이션이 배포되지 않은 기본 인프라스트럭처로 이 실습을 시작했습니다. 작업을 진행하려면이 워크샵 전체에서 사용될 IDE를 엽니다. 이 워크숍에서는 Amazon Cloud9를 IDE로 사용합니다. Cloud 콘솔을 열려면 AWS 콘솔에서 서비스를 검색하거나 개발자 도구 범주 아래에있는 서비스를 찾으십시오.

Once you've opened the Cloud9 service, you'll see all the environments available to this account. An environment has been pre-provisioned for use throughout this workshop. On the AWS Cloud9 page you'll see <b>"Serverless SaaS Workshop IDE"</b> listed as one of the environments. Open this environment by selecting the <b>"Open IDE"</b> button.

Cloud9 서비스를 열면 계정에서 사용 가능한 모든 환경이 표시됩니다. 이 워크샵 전체에서 사용할 환경이 사전 프로비저닝되어 있습니다. AWS Cloud9 페이지에서 <b>"Serverless SaaS Workshop IDE"</b>가 환경 중 하나로 표시됩니다. <b>"Open IDE"</b> 버튼을 선택하여 이 환경을 여십시오.

<p align="center"><img src="../images/lab1/Cloud9Console.png" alt="Cloud 9 Console"/></p>

The Cloud9 IDE has all the traditional elements you would expect to find in a modern IDE. Similar in layout to Eclipse, Visual Studio, VS Code and other popular IDEs, Cloud9 defaults to a file tree in a left-hand pane, a main editor pane in the center (which currently displays a welcome page), and an output pane at the bottom. This bottom window pane is an active Linux command-line shell which we will use throughout the workshop. The page will appear as follows:

Cloud9 IDE에는 최신 IDE에서 기대할 수있는 모든 모든 요소들이 있습니다. Eclipse, Visual Studio, VS Code 및 기타 널리 사용되는 IDE와 유사한 레이아웃으로 Cloud9는 기본적으로 왼쪽 창의 파일 트리, 중앙의 기본 편집기 창 (현재 환영 페이지가 표시됨) 및 출력 창의 아래 창은 작업장 전체에서 사용할 활성 Linux 명령 줄 셸입니다. 페이지가 다음과 같이 나타날것 입니다:

<p align="center"><img src="../images/lab1/Cloud9IDE.png" alt="Cloud 9 IDE"/></p>

<b>Step 2</b> – Now that Cloud9 is open, we can fetch our source code and build and deploy our applications. First, we need to bootstrap our Cloud9 instance by installing some dependencies and getting the code base set it up for you to use. We have configured a CI/CD pipeline using the AWS developer tools CodeCommit, CodeBuild, CodeDeploy and CodePipeline. This script will download a copy of the workshop source code and populate a local CodeCommit repository that we will be using as we apply changes during this workshop. Making changes to this repository will trigger the pipeline to build and deploy the application. Let's execute the bootstrap script with a cURL command. Place your cursor at the command line prompt in the lower window pane of Cloud9 and run the following command:

<b>Step 2</b> – 이제 Cloud9이 열려 있으므로 소스 코드를 가져 와서 애플리케이션을 빌드 및 배포 할 수 있습니다. 먼저 일부 라이브러리를 설치하고 사용할 수 있도록 코드베이스를 설정하여 Cloud9 인스턴스를 세팅 해야합니다. 저희는 AWS 개발자 도구 인 CodeCommit, CodeBuild, CodeDeploy 및 CodePipeline을 사용하여 CI / CD 파이프 라인을 구성했습니다. 아래 스크립트는 워크샵 소스 코드 사본을 다운로드하고 이 워크샵 중에 변경 사항을 적용 할 때 사용할 로컬 CodeCommit 저장소를 만들겁니다. 이 저장소에 변경이 일어나면 빌드 파이프라인이 애플리케이션을 빌드하고 배치하도록 트리거 합니다. cURL 명령으로 부트스트랩 스크립트를 실행해 봅시다. Cloud9의 하단 창에있는 명령 행 프롬프트에 커서를 놓고 다음 명령을 실행하십시오:

```
curl -s https://raw.githubusercontent.com/aws-samples/aws-saas-factory-serverless-workshop/master/resources/bootstrap.sh | bash
```

When this script is done executing, you'll have all the necessary pieces in place to begin exercising our application.

이 스크립트 실행이 완료되면, 여러분들은 저희가 미리 준비한 애플리케이션을 사용하는데 필요한 모든 것들을 준비하게 됩니다.

<b>Step 3</b> – The script committed a revision to the CodeCommit source control system, triggering a deployment. To see the status of this pipeline, navigate to the CodePipeline service within the AWS console. You'll see the <b>saas-factory-srvls-wrkshp-pipeline-lab1</b> pipeline listed amongst the pipelines in your account. The screen should appear as follows:

<b>Step3</b> – 스크립트가 CodeCommit 소스 제어 시스템의 개정판을 커밋하며 배포 과정을 트리거했습니다. 이 파이프라인의 상태를 보려면 AWS 콘솔에서 CodePipeline 서비스로 이동하십시오. 계정의 파이프라인들 가운데 <b>saas-factory-srvls-wrkshp-pipeline-lab1 /b> 파이프라인을 확인할 수 있을겁니다. 화면은 다음과 같이 나타납니다:

<p align="center"><img src="../images/lab1/Codepipeline.png" alt="Code Pipeline"/></p>

Once this pipeline succeeds it will have deployed our monolith application to the infrastructure that was provisioned at the outset of this workshop. If you select the pipeline from the list, you'll be able to view the execution status of the pipeline.

이 파이프라인이 성공하면 이 워크샵의 미리 프로비저닝된 인프라에 모놀리식 애플리케이션을을 배포 하게 됩니다. 목록에서 파이프 라인을 선택하면 파이프 라인의 실행 상태를 볼 수 있습니다.

<b>Step 4</b> – While our code is being deployed, let's take a look at some of the infrastructure that will be hosting our application. The architecture that has been pre-provisioned for our monolith intentionally resembles something that could be on AWS now or on-prem. In this particular monolith solution, we have one layer of compute talking to one database. We have deployed this on an EC2 cluster in a multi-AZ configuration in an auto-scaling group. To see the current instances, navigate to the EC2 service within the AWS console. Now, select <b>Auto Scaling Groups</b> from the navigation pane on the left (you may need to scroll to locate the menu item). This will display a list of provisioned auto-scaling groups including one for Lab 1. Select the checkbox for this auto-scaling group. In the details at the bottom of the page, select the <b>Instances</b> tab to view the EC2 instances in the auto-scaling group. The screen should appear as follows:

<b>Step 4</b> – 코드가 배포되는 동안 애플리케이션을 호스팅 할 인프라를 살펴 보겠습니다. 모놀리식 애플리케이션을 위해 사전 프로비저닝 된 아키텍처는 의도적으로 AWS 또는 온 프레미스에있을 수있는 것과 유사하게 만들었습니다. 이 모놀리식 솔루션에서는 하나의 데이터베이스와 통신하는 하나의 컴퓨팅 계층이 있습니다. Auto Scaling Group의 다중 AZ로 구성된 EC2 클러스터에 이를 배포했습니다. 현재 인스턴스를 확인 하려면 AWS 콘솔에서 EC2 서비스로 이동하십시오. 이제 왼쪽의 탐색 창에서 <b>Auto Scaling Group</b>을 선택하십시오 (메뉴 항목을 찾으려면 스크롤해야 할 수도 있음). 그러면 Lab 1 용 그룹을 포함하여 프로비저닝 된 Auto Scaling Group 목록이 표시됩니다. 이 Auto Scaling Group의 확인 란 을 선택하십시오. 페이지 하단의 세부 정보에서 <b>Instances</b> 탭을 선택하여 오토 스켈일링 그룹의 EC2 인스턴스를 살펴 봅니다. 화면은 다음과 같이 나타납니다.:

<p align="center"><img src="../images/lab1/AutoScalingGroup.png" alt="Code Pipeline"/></p>

This view shows that we have 2 running instances in our auto-scaling group. In a production environment, we'd likely provision a larger minimum footprint. These instances sit behind an Application Load Balancer (ALB) that directs traffic to each of the instances shown here.

이 화면이 Auto Scaling Group에 2 개의 실행중인 인스턴스가 있음을 보여줍니다. 프로덕션 환경에서는 더 큰 최소 공간을 프로비저닝 해야 할 것 입니다. 이러한 인스턴스는 여기에 표시된 각 인스턴스로 트래픽을 보내는 Application Load Balancer (ALB) 뒤에 있습니다.

<b>Step 5</b> – The infrastructure provisioned also includes an Aurora RDS cluster for our database. Navigate to RDS in the console and select <b>Databases</b> from the navigation pane on the left-hand side of the console. You'll see the <b>saas-factory-srvls-wrkshp-lab1-cluster</b> cluster and its instances. You will see other RDS clusters that we will utilize later on in the workshop. You can ignore those for now. The screen will appear as follows:

<b>Step 5</b> – 프로비저닝 된 인프라에는 데이터베이스를위한 Aurora RDS 클러스터도 포함됩니다. 콘솔에서 RDS로 이동하고 콘솔 왼쪽의 탐색 창에서 <b>Databases</b>를 선택하십시오. <b>saas-factory-srvls-wrkshp-lab1-cluster</b> 클러스터와 해당 인스턴스가 표시됩니다. 워크샵에서 나중에 활용할 다른 RDS 클러스터도 볼 수 있습니다. 지금은 무시 해도됩니다. 화면은 다음과 같이 나타납니다 :

<p align="center"><img src="../images/lab1/RDS.png" alt="LAB1 RDS"/></p>

This RDS cluster is deployed with separate reader and writer instances. As our monolith database it will hold all of the data for our entire single-tenant environment. As is normal for monolithic systems, all of the code in our application services has access to all of the tables and data in database. There is no separation of which business logic can access which segments of data.

이 RDS 클러스터는 별도의 reader 및 writer 인스턴스와 함께 배포됩니다. 단일 데이터베이스로서 전체 단일 테넌트 환경에 대한 모든 데이터를 보유합니다. 단일 시스템의 경우와 마찬가지로 애플리케이션 서비스의 모든 코드는 데이터베이스의 모든 테이블과 데이터에 액세스 할 수 있습니다. 어떤 비즈니스 로직이 어떤 데이터 세그먼트에 액세스 할 수 있는지 분리 할 수 없는 상태 입니다.

<b>Step 6</b> – Now that we've see the compute and data layers of our supporting infrastructure, let's turn our attention to the running application. Before we can access the application, we'll first need to confirm that the deployment process has completed. Once again, navigate to CodePipeline in the console. Confirm <b>saas-factory-srvls-wrkshp-pipeline-lab1</b> shows Succeeded.

<b>Step 6</b> – 이제 인프라의 컴퓨팅 및 데이터 계층을 확인 했으므로 실행중인 애플리케이션에 주의를 기울여 보겠습니다. 애플리케이션에 액세스 하려면 먼저 배포 프로세스가 완료되었는지 확인해야합니다. 다시 한 번 콘솔에서 CodePipeline으로 이동하십시오. <b>saas-factory-srvls-wrkshp-pipeline-lab1</b>에 성공 함이 표시되어 있는지 확인하십시오.

<b>Step 7</b> – We need to locate the URL for our application. This is the domain name associated with the ALB which is routing requests to the EC2 instances in our target group. Navigate to the EC2 service in the AWS console. Select <b>Load Balancers</b> from the navigation pane on the left-hand side of the page. This will display a list of load balancers. Select <b>saas-wrkshp-lab1-[REGION]</b> from the list and your screen should appear as follows:

<b>Step 7</b> – 애플리케이션의 URL을 찾아야합니다. 이는 Target Group의 EC2 인스턴스로 요청을 라우팅하는 ALB와 관련된 도메인 이름입니다. AWS 콘솔에서 EC2 서비스로 이동하십시오. 페이지 왼쪽의 탐색 창에서 <b>Load Balancer</b>를 선택하십시오. 로드 밸런서 목록이 표시됩니다. 목록에서 <b>saas-wrkshp-lab1-[REGION]</b>을 선택하면 화면이 다음과 같이 나타납니다.

<p align="center"><img src="../images/lab1/LoadBalancer.png" alt="Load Balancer"/></p>

In the description tab at the bottom of this page, the DNS name is listed under the ARN for the load balancer. Copy this DNS name and we'll use it to access the monolith application.

페이지 하단의 description에 로드 밸런서에 대한 ARN 정보 아래에 DNS 명이 위치해 있습니다. 이 DNS 명을 통해 모놀리식 애플리케이션 접속 할 것이므로 이를 복사해 두십시오.

<b>Step 8</b> – Open a new tab or window in your browser and paste in the DNS name that we collected in the prior step as follows:

<b>Step 8</b> – 브라우저를 열고 복사한 DNS 명을 주소창에 붙여 넣으세요.

    http://saas-wrkshp-lab1-[REGION]-[RANDOM].[REGION].elb.amazonaws.com

This should open the landing page of our sample monolithic application that will appear as follows:

샘플 모놀리식 애플리케이션의 랜딩 페이지가 아래와 같이 열려야 합니다.

<p align="center"><img src="../images/lab1/HomePage.png" alt="Home Page"/></p>

<b>Step 9</b> – We can now begin to exercise the moving parts of our application. Start by logging in. Select the <b>Sign In</b> link at the top right of the page and you will be prompted with the following login form:

<b>Step 9</b> – 이제 애플리케이션의 구성 요소들에 대한 실습을 시작할 수 있습니다. 로그인 하여 시작하십시오. 페이지 오른쪽 상단의 <b>Sign In</b> 링크를 선택하면 다음과 같은 로그인 양식이 표시됩니다.

<p align="center"><img src="../images/lab1/LoginPage.png" alt="Login Page"/></p>

We created a user account for you as part of provisioning the workshop. You can login with the following credentials:

실습을 위해 이 워크샵을 프로비저닝 하는 과정에서 여러분을 위한 user account를 미리 생성했습니다. 다음의 크레덴셜 정보를 통해 로그인할 수 있습니다.

```
Email address: monolith_user@example.com
Password: Monolith123
```

<b>Step 10</b> – Once you land in the application, you'll see that our solution has some very basic functionality. The application that's been built was kept intentionally simple. Our goal is to focus more on the migration process and less on creating a fully functional reference application. The application mimics the functionality you might see in a e-commerce seller's platform. Sellers can manage their product catalog and view orders that have been placed by customers on some fictitious store frontend. Because we don't have a fake shopping site, we have a simple form to enter order data just so there is something to play with. The business logic of the system supports standard create, read, update and delete (CRUD) operations for products and orders along with a dashboard to view basic status information.

Let's start by adding a product. From the navigation options at the top of the page, select <b>Products</b> as shown below:

<b>Step 10</b> – 애플리케이션에 들어가보면 솔루션에 매우 기본적인 기능이 있음을 알 수 있습니다. 만들어진 애플리케이션은 의도적으로 단순하게 만들어져 있습니다. 우리의 목표는 마이그레이션 프로세스에 더 집중하고 완전한 기능을 갖춘 참조 애플리케이션을 만드는 데는 덜 집중하는 것입니다. 이 애플리케이션은 이커머스 판매자의 플랫폼에서 볼 수있는 기능을 모방 했습니다. 판매자는 제품 카탈로그를 관리하고 가상의 상점에서 고객이 주문한 주문을 볼 수 있습니다. 우리는 가짜 쇼핑 사이트를 현재 갖고 있지 않기 때문에 주문 데이터를 입력하는 간단한 양식을 통해 실습을 진행할 예정입니다. 시스템의 비즈니스 로직은 기본 상태 정보를 볼 수있는 대시 보드와 함께 제품 및 주문에 대한 표준 CRUD (Create, Read, Update and Delete) 작업이 가능하도록 되어 있습니다.

제품을 추가하여 시작해 보겠습니다. 페이지 상단의 탐색 옵션에서 다음과 같이 <b>Products</b>을 선택하십시오.

<p align="center"><img src="../images/lab1/Products.png" alt="Products"/></p>

On the products page you'll see some existing items listed. Select the <b>Add Product</b> button to enter a new product to the system. You should see the following form:

제품 페이지에는 몇 가지 기존 항목이 표시됩니다. <b>Add Product</b> 버튼을 선택하여 시스템에 새 제품을 입력하십시오. 다음과 같은 형식이 나타납니다.

<p align="center"><img src="../images/lab1/AddProduct.png" alt="Add Product"/></p>

Fill in the data for a sample product and click the <b>Add Product</b> button to save it. Repeat this process a few times to add a few additional products to the system.

샘플 제품의 데이터를 채우고 <b>Add Product</b> 버튼을 클릭하여 저장하십시오. 이 과정을 몇 차례 반복하여 시스템에 몇 가지 추가 제품을 추가하십시오.

<b>Step 11</b> – Now let's look at the code behind this monolith solution. We won't devote too much time to the monolith code since we're moving away from it, but it's helpful to have a bit more context and detail about the application that we're moving away from. This solution is built with Java, but the concepts here are similar to the patterns that appear in most monoliths.

To explore the code, you must first re-open the Cloud9 service in the AWS console. Once you open the Cloud9, select the <b>Serverless SaaS Workshop IDE</b> that listed as one of the environments. Open this environment by selected the <b>Open IDE</b> button.

Now, expand the list of folders in the left-hand pane drilling into the <b>lab 1</b> folder. In that folder under <b>server</b> you'll find the <b>src</b> folder that holds the source code for our monolith. The main folders of our application are show in the following image:

<b>Step 11</ b> – 이제 이 모놀리식 솔루션의 기본 코드를 살펴 보겠습니다. 우리는 실습 과정에서 모노리스 코드에 너무 많은 시간을 할애하지는 않지만, 애플리케이션에 대한 좀 더 많은 컨텍스트와 세부 사항을 살펴 보는것이 도움이될 것 입니다. 이 솔루션은 Java로 빌드 되었지만 여기서 개념은 대부분의 모놀리식 애플리케이션에서 나타나는 패턴과 유사합니다.

코드를 탐색하려면 먼저 AWS 콘솔에서 Cloud9 서비스를 다시 열어야합니다. Cloud9를 열면 환경 중 하나로 나열된 <b>Serverless SaaS Workshop IDE</b>를 선택하십시오. <b>Open IDE</b> 버튼을 선택하여이 환경을 여십시오.

이제 왼쪽 네비케이션 창의 폴더 목록을 <b>lab 1</b> 폴더로 펼치십시오. <b>server</b> 폴더 아래에는 모놀리식 애플리케이션의 소스 코드가 들어있는 <b>src</b> 폴더가 있습니다. 애플리케이션의 기본 폴더는 다음 이미지에 표시되어 있습니다.

<p align="center"><img src="../images/lab1/CodeTree.png" alt="CodeTree"/></p>

You'll see a traditional Spring Boot project layout. The controller folder holds the code that is the entry point into your business services. It receives HTTP requests, coordinates running your backend service logic, and then triggers the rendering of the view. There are also domain objects that hold the representation of our data as it moves through the system. The repository folder holds the code needed to store and retrieve products and orders from the database. The service folder holds the actual business rules and implementation of our services.

전통적인 Spring Boot 프로젝트 레이아웃이 표시될겁니다. controller 폴더에는 비즈니스 서비스의 진입점이 되는 코드가 있습니다. HTTP 요청을 수신하고 백엔드 서비스 로직 실행을 조정 한 후 뷰의 렌더링을 트리거합니다. 시스템을 통해 데이터가 이동할 때 데이터 표현을 유지하는 도메인 객체도 있습니다. repository 폴더에는 데이터베이스에서 제품 및 주문을 저장하고 검색하는 데 필요한 코드가 있습니다. service 폴더에는 실제 비즈니스 규칙과 서비스 구현이 있습니다.

<b>Step 12</b> – Let's drill into one of the services by opening the service folder. Here you'll see the various product and order related files that implement our services. There are interfaces (<b>OrderService.java</b>, for example) and corresponding implementation files (<b>OrderServiceImpl.java</b>). Let's open the <b>OrderServiceImpl.java</b> file by double-clicking the file name in the left-hand pane. Below is a snippet of code from this file:

<b>Step 12</b> – service 폴더를 열어 서비스 중 하나를 살펴 보겠습니다. 여기에서 애플리케이션 서비스에 구현된 다양한 제품 및 주문 관련 서비스 소스 파일을 볼 수 있습니다. 인터페이스 (예 <b>OrderService.java </b>)와 해당 구현 파일 (<b>OrderServiceImpl.java </b>)이 있습니다. 왼쪽 창에서 파일 이름을 두 번 클릭하여 <b>OrderServiceImpl.java</b> 파일을 엽니다. 아래는 이 파일의 코드 일부 입니다.

```java
@Autowired
private OrderDao orderDao;

@Override
public List<Order> getOrders() throws Exception {
    logger.info("OrderService::getOrders");
    StopWatch timer = new StopWatch();
    timer.start();
    List<Order> orders = orderDao.getOrders();
    timer.stop();
    logger.info("OrderService::getOrders exec" + timer.getTotalTimeMillis());
    return orders;
}

@Override
public Order getOrder(Integer orderId) throws Exception {
    logger.info("OrderService::getOrder" + orderId);
    StopWatch timer = new StopWatch();
    timer.start();
    Order order = orderDao.getOrder(orderId);
    timer.stop();
    logger.info("OrderService::getOrder exec" + timer.getTotalTimeMillis());
    return order;
}
```

These lines show a couple of methods of our Order service that process GET requests from the client. The first method processes request to get all orders and the second method gets a single order based on an order identifier. This is standard code that has no tenant awareness.

이 코드는 주문 서비스가 고객의 GET 요청을 처리하는 몇 가지 방법을 보여줍니다. 첫 번째 방법은 모든 주문을 받도록 요청을 처리하고 두 번째 방법은 주문 식별자에 따라 단일 주문을 받습니다. 이것은 테넌트 인식이 없는 일반적인 표준 코드입니다.

<b>Step 13</b> – The path into these services in the monolith relies on a Model View Controller (MVC) pattern that is commonly supported by frameworks that are used to build monolithic solutions. To get better sense for how requests are processed and pages are rendered in this MVC model, let's start with the UI of our application (the rendered "view" in our MVC model).

Navigate to the application URL that we used above to access the application and sign in with the credentials that were provided. Now, select the "Products" item from the application menu and you will see a list of products.

<b>Step 13</b> – 모놀리식의 이러한 서비스로의 경로는 모놀리식 솔루션을 구축하는 데 사용되는 프레임 워크에서 일반적으로 지원되는 MVC (Model View Controller) 패턴에 의존합니다. 이 MVC 모델에서 요청이 처리되고 페이지가 렌더링되는 방식을 더 잘 이해하기 위해 애플리케이션의 UI (MVC 모델의 "view")로 에서 부터 살펴보기 시작 하겠습니다.

위에서 사용한 애플리케이션 URL로 이동하여 애플리케이션에 액세스 하고 제공된 자격 증명으로 로그인합니다. 이제 애플리케이션 메뉴에서 "Products"항목을 선택하면 제품 목록이 표시됩니다.

<p align="center"><img src="../images/lab1/Products.png" alt="Products"/></p>

Let's try deleting the _680425 Swift iOS SaaS Identity_ product from the catalog. Click the red delete <b>Del</b> icon that appears at the right-hand edge of the row displaying the fictitious iOS book. You will be prompted to confirm that you really want to delete this product:

카탈로그에서 _680425 Swift iOS SaaS Identity_ 제품을 삭제해 봅시다. 가상 iOS 책을 표시하는 행의 오른쪽 가장자리에 나타나는 빨간색 삭제 <b>Del</b> 아이콘을 클릭하십시오. 이 제품을 정말로 삭제할 것인지 묻는 메시지가 나타납니다.

<p align="center"><img src="../images/lab1/DeleteConfirm.png" alt="Confirm Delete Product"/></p>

Click on the <b>Delete Product</b> button. Oops! What happened? Why did we get a 404 error?

<b>Delete Product</b> 버튼을 클릭 합니다. 이런! 404 에러가 발생할겁니다. 왜 그럴까요?

<p align="center"><img src="../images/lab1/DeleteError.png" alt="Delete 404 Error"/></p>

<b>Step 14</b> – Let's troubleshoot our problem by digging into the "Controller" portion of our MVC model to see why our delete request isn't working. Navigate to the Cloud9 service and open the IDE for this workshop. With the source tree that appears on the left, navigate to the <b>lab1/server/src/main/java</b> folder. Within the nested folders (these correspond to Java's package naming organization of code), open the <b>controller</b> folder. This will give you a list of the controllers that are implemented for our monolith. Double click the <b>ProductsController.java</b> file to open the editor for this file.

Within this file, we'll navigate to the <b>updateProduct()</b> and <b>deleteProduct()</b> methods of our Java class. These two methods represent the entry point of our HTTP calls that will be processed by the various methods in this class. You'll notice that, within the body of these methods, we are calling our ProductService that is the actual implementation of our business logic functionality. Let's take a closer look at these two specific methods to see if we can figure out what's broken with the delete product functionality that we observed in the application. The two methods are as follows:

<b>Step 14</b> – MVC 모델의 "Controller"부분을 조사하여 삭제 요청이 작동하지 않는 이유를 확인하여 문제를 해결해 보겠습니다. Cloud9 서비스로 이동하여이 워크샵의 IDE를 엽니다. 왼쪽에 나타나는 소스 트리를 사용하여 <b>lab1/server/src/main/java</b> 폴더로 이동하십시오. 중첩 된 폴더 내에서 (Java의 패키지 이름 지정 코드 구성에 해당) <b>controller</b> 폴더를 여십시오. 모노리스 용으로 구현 된 컨트롤러 목록이 표시됩니다. <b>ProductsController.java </b> 파일을 두 번 클릭하여 이 파일의 편집기를 여십시오.

이 파일에서 Java 클래스의 <b>updateProduct()</b> 및 <b>deleteProduct()</b> 메소드로 이동합니다. 이 두 메소드는이 클래스의 다양한 메소드에 의해 처리 될 HTTP 호출의 시작점을 나타냅니다. 이러한 메소드 본문 내에서 비즈니스 로직 기능의 실제 구현 인 ProductService를 호출하고 있음을 알 수 있습니다. 이 두 가지 메소드를 자세히 살펴보고 애플리케이션에서 관찰 한 제품 삭제 기능의 문제를 만드는 부분을 파악할 수 있는지 살펴보세요. 두 가지 메소드는 다음과 같습니다.

```java
@PostMapping("/updateProduct")
public String updateProduct(@ModelAttribute Product product, Model model) throws Exception {
	LOGGER.info("ProductsController::updateProduct " + product);
	productService.saveProduct(product);
	return "redirect:/products";
}

public String deleteProduct(@ModelAttribute Product product) throws Exception {
	LOGGER.info("ProductsController::deleteProduct " + product.getId());
	productService.deleteProduct(product);
	return "redirect:/products";
}
```

At first glance, there doesn't appear to be anything wrong with the <b>deleteProduct()</b> method. However, if you compare it to the <b>updateProduct()</b> method above it, you'll notice that there is a <b>@PostMapping</b> annotation associated with our <b>updateProduct()</b> method. This annotation is missing from our <b>deleteProduct()</b> method. Without this annotation, the HTTP calls from the browser will have no route to the delete method.

To resolve this, we simply need to add the missing annotation to our <b>deleteProduct()</b> method. Do this by adding the annotation as shown below:

언뜻 보면 <b>deleteProduct()</b> 메소드에 문제가 없는 것 같습니다. 그러나 위의 <b>updateProduct()</b> 메소드와 비교하면 <b>updateProduct()</b> 메소드에는 연결될 annotation <b>@PostMapping</b> 이 있습니다. 하지만 <b>deleteProduct()</b> 메소드에는 이런 annotation이 누락되었습니다. 이 annotation 없이는 브라우저의 HTTP 호출은 delete 메소드로 라우팅 되지 않을 것 입니다.

이 문제를 해결하려면 <b>deleteProduct()</b> 메소드에 누락 된 annotaion을 추가하기만 하면 됩니다. 아래와 같이 annotation을 추가해주십시오.

```java
@PostMapping("/deleteProduct")
public String deleteProduct(@ModelAttribute Product product) throws Exception {
    LOGGER.info("ProductsController::deleteProduct" + product.getId());
    productService.deleteProduct(product);
    return "redirect:/products";
}
```

<b>Step 15</b> – We now need to save the changes we've made to the file and deploy the updated version of our controller. Select the "Save All" option from the "File" menu in Cloud9. Then, issue the following commands from the terminal window of Cloud9 to commit our changes and fire off the deployment of our new version.

<b>Step 15</b> – 이제 파일에 대한 변경 사항을 저장하고 업데이트 된 버전의 컨트롤러를 배포해야 합니다. Cloud9의 "File"메뉴에서 "Save All"옵션을 선택하십시오. 그런 다음 Cloud9의 터미널 창에서 다음 명령을 실행하여 변경 사항을 커밋하고 새 버전의 배포를 시작하십시오.

```
cd /home/ec2-user/environment/saas-factory-serverless-workshop/
git add .
git commit -m "Added annotation to deleteProduct"
git push
```

Committing this change will automatically cause the CodePipeline to trigger a build and deployment of the new product service that includes your changed function.

이 변경 사항을 커밋하면 CodePipeline이 변경된 기능이 포함 된 새 제품 서비스에 대한 빌드 및 배포를 자동으로 트리거합니다.

<b>Step 16</b> – Before we can access the application, we'll first need to confirm that the deployment process has completed. Once again, navigate to CodePipeline in the console. Confirm <b>saas-factory-srvls-wrkshp-pipeline-lab1</b> shows Succeeded.

<b>Step 16</b> – 애플리케이션에 액세스하기 전에 먼저 배포 프로세스가 완료되었는지 확인해야 합니다. 다시 한 번 콘솔에서 CodePipeline으로 이동하십시오. <b>saas-factory-srvls-wrkshp-pipeline-lab1</b>에 성공이 표시되어 있는지 확인하십시오.

<b>Step 17</b> – Let's now verify that our change was applied. Open the application using the URL acquired above and login using the supplied credentials. Now, select the "Products" menu item and again attempt to delete the the _680425 Swift iOS SaaS Identity_ product from the catalog by selecting the delete "Del" button that is on the far right-hand side of the row. Assuming your change was applied correctly and the new version completed deployment, your product should now be deleted successfully.

<b>Step 17</b> – 이제 변경 사항이 적용되었는지 확인하겠습니다. 위에서 얻은 URL을 사용하여 애플리케이션을 열고 제공된 자격 증명을 사용하여 로그인하십시오. 이제 "Products" 메뉴 항목을 선택하고 행의 맨 오른쪽에있는 "Del" 버튼을 선택하여 카탈로그에서 _680425 Swift iOS SaaS Identity_ 제품을 다시 삭제하십시오. 변경 사항이 올바르게 적용되었고 새 버전이 배치를 완료되었다면 이제 제품이 성공적으로 삭제될겁니다.

<b>Step 18</b> – As a final step, we want to take a quick look at the web client for this application. While we won't dig into the code much here, we want to emphasize the fact our monolith is actually rendering and serving all the HTML for our solution from the server. This solution follows the classic MVC pattern. We are using a common Java templating library to represent the UI views of our application. As each request comes into a <b>C</b>ontroller, it is processed and populates <b>M</b>odel object(s) to pass to the <b>V</b>iew as page variables to render in the HTML. After manipulating the <b>M</b>odel, the <b>C</b>ontroller triggers the templating library which binds these model objects to templates to render the HTML <b>V</b>iew that is returned.

The nuances of the MVC framework are not essential here. The key here is to realize that our controller uses Java objects to represent our data and a templating framework to bind these objects to the templates —- all on the server side. To view a sample UI template, navigate to the Cloud9 service in the console and open the IDE for the workshop. Open the <b>lab1/server/src/main/resources</b> folder from the source tree on in the left-hand pane of the IDE. Then, open the <b>templates</b> folder and select the <b>products.html</b> template file by double-clicking on the file name. The following is a snippet from the <b>products.html</b> UI template file that gives you a sense of how the models are bound to templates:

<b>Step 18</b> – 마지막 단계로 이 애플리케이션의 웹 클라이언트를 간단히 살펴보겠습니다. 실습의 목적상 코드를 깊이 살펴 보지 않지만 모놀리스가 실제로 서버에서 솔루션의 모든 HTML을 렌더링하고 제공한다는 사실은 알아 두면 좋겠습니다. 이 솔루션은 고전적인 MVC 패턴을 따릅니다. 우리는 애플리케이션의 UI 뷰를 나타내기 위해 공통 Java 템플릿 라이브러리를 사용하고 있습니다. 각 요청이 <b>C</b>ontroller에 들어오면 요청이 처리되고 <b>M</b>odel 객체가 채워져 HTML로 렌더링하기위해 <b>V</b>iew에 페이지 변수로써 전달 됩니다. <b>M</b>odel을 조작 한 후 <b>C</b>ontroller는 이러한 모델 객체를 템플릿에 바인딩하여 반환되는 HTML <b>V</b>iew를 렌더링하는 템플릿 라이브러리를 트리거합니다.

MVC 프레임워크에 대한 미묘한 차이는 본 실십에서 중요한 것은 아닙니다. 여기서 핵심은 컨트롤러가 Java 객체를 사용하여 데이터를 나타내고 템플릿 프레임 워크를 템플릿에 바인딩하기 위해 템플릿 프레임 워크를 서버 측에서 모두 사용한다는 것을 이해하는 것입니다. 샘플 UI 템플릿을 보려면 콘솔에서 Cloud9 서비스로 이동하여 워크샵을위한 IDE를 엽니다. IDE의 왼쪽 네비게이션 창에있는 소스 트리에서 <b>lab1/server/src/main/resources</b> 폴더를 여십시오. 그런 다음 <b>templates</b> 폴더를 열고 파일 이름을 두 번 클릭하여 <b>products.html</b> 템플릿 파일을 여십시오. 다음은 <b>products.html</b> UI 템플릿 파일의 코드 일부로, 모델이 템플릿에 어떻게 바인딩되는지 보여줍니다.

<p align="center"><img src="../images/lab1/ProductsHTML.png" alt="products.html"/></p>

This snippet represents the portion of the template that is used to populate this list of products in our table. You'll see that there is a for-each loop that wraps the &lt;tr&gt; table row tags and that the &lt;td&gt; tags for the data of our table cells include references to the product model object. For example, the first column resolves to the product SKU with the following syntax: <b>\${product.sku}</b>. The end result here is that your request ends up serving up the HTML that is the binding of your HTML and your model object(s).

이 코드는 화면에 제품 목록을 채우는데 사용되는 템플릿을 나타냅니다. &lt;tr&gt; 테이블 행 태그를 감싸는 for-each 루프가 있고 &lt;td&gt;가 있음을 알 수 있습니다. 테이블 셀의 데이터 태그에는 제품 모델 객체에 대한 참조가 포함됩니다. 예를 들어 첫 번째 열은 <b>\${product.sku}</b> 구문을 사용하여 제품 SKU로 구별됩니다. 여기서 최종 결과는 여러분의 요청에 대한 결과로써 HTML과 모델 객체의 바인딩된 HTML을 제공 한다는 것입니다.

## Review

The goal of Lab 1 was to expose you to the fundamentals of the existing monolith. We reviewed the basic architecture of the monolith environment, acquired the code for our application, and deployed that code to our infrastructure. We then proceeded to exercise that code by running our monolith application. We also explored the underlying code of our monolith to highlight the classic MVC model that is used to build this solution, focusing in on the services and how they are wired up via a controller. In true monolith form, the application was entirely realized from a single unit of deployment. The web application and the supporting application services were all deployed as a unit to a cluster of EC2 instances. There was no awareness of tenancy in this environment whatsoever.

Lab 1의 목표는 기존 모놀리스의 기본 사항에 대한 정보를 제공하는 것이 었습니다. 모놀리스 환경의 기본 아키텍처를 검토하고, 애플리케이션 코드를 얻은 후 해당 코드를 인프라에 배포했습니다. 그런 다음 모노리스 애플리케이션을 실행하여 해당 코드를 연습했습니다. 또한 이 솔루션을 구축하는 데 사용되는 클래식 MVC 모델을 강조하기 위해 모놀리스의 기본 코드를 살펴보고 서비스 및 컨트롤러를 통한 서비스 연결이 어떻게 되는지 살펴보는데 중점을 두었습니다. 진정한 모노리스 형태로, 애플리케이션은 완전히 단일 배포 단위로 배포 되었습니다. 웹 애플리케이션과 이를 지원 하는 모든 애플리케이션 서비스는 EC2 인스턴스 클러스터에 하나의 단위로 배포되었습니다. 여기까지 만든 환경에서는 테넌트에 대한 인식은 전혀 반영되지 않았습니다.

You have now completed Lab 1.

Lab 1을 마칩니다.

[Continue to Lab 2](../lab2/README.md)

[Lab 2 계속 하기](../lab2/README.md)

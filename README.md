# <p align="center">AWS SaaS Factory Monolith to Serverless SaaS Workshop</p>

## Overview

The move from a single-tenant monolithic architecture to a multi-tenant, modern architecture can be challenging for many organizations. The tight coupling and interwoven dependencies of a monolithic environment makes it especially difficult to move your system to microservices. Now, layer on that the goal of moving to a serverless model that supports multi-tenant SaaS and you now have a rather long list of technical, migration, and design challenges that will further complicate this transformation.

단일 테넌트 모놀리식 아키텍처에서 다중 테넌트 현대 아키텍처로의 전환은 많은 조직에게 어려운 과제가 될 수 있습니다. 모놀리식 환경의 밀접한 결합과 종속성으로 인해 시스템을 마이크로서비스로 옮기기가 특히나 어렵습니다. 이제 여러분은 다중테넌트 SaaS를 지원하는 서버리스 모델로 전환한다는 목표를 세우고 이제는 이 전환을 더욱 복잡하게 만드는 다소 긴 기술, 마이그레이션 및 설계 과제 목록을 갖고 있습니다.

Given this complexity, many organizations will attempt to tackle this migration in a more evolutionary fashion where the elements of your system are incrementally transformed to a modern multi-tenant architecture without requiring some “big bang” moment where you completely rewrite your system. This approach also tends to fit better with the business priorities of organizations that are trying to find ways to continue to support existing customers while they gradually move their architecture over to this new model.

이러한 복잡성을 감안할 때 많은 조직에서는 "빅뱅" 으로 여러분의 시스템을 다시 만들기 보다는 점진적으로 현대적인 멀티테넌트 아키텍처로 변환되는 방식으로 마이그레이션을 시도할 것 입니다.
이러한 접근법은 점진적으로 기존 아키텍처를 새로운 아키텍처 모델로 이동 하는 중에 기존 고객들을 계속해서 지원 할 수 있는 방법을 찾고 있는 조직/비지니스에 적합 합니다.

The goal of this lab is to guide you through a monolith to serverless SaaS migration that provides a more in-depth look at the common moving parts of this problem. Certainly, each solution will have its own unique collection of migration challenges. However, seeing a working example can provide you with insights into patterns and strategies that can help shape your approach to moving your monolithic environment to a multi-tenant SaaS model.

In this Lab, we'll start with a traditional monolithic architecture for a working sample application. Then, we'll progressively migrate the elements of the single-tenant monolithic architecture to a modern multi-tenant solution. This will include moving to a modern web application hosted on S3, introducing the API Gateway, decomposing the application tier into serverless microservices, and carving data out of our monolithic database and moving management of this data to the individual microservices that take over ownership of managing this data. Along the way, we'll also introduce the elements needed to introduce multi-tenant best practices into your solution.

<br></br>

## Lab 1 – Deploying, Exploring, and Exercising the Single-Tenant Monolith

[![Lab1](images/lab1.png)](./lab1/README.md "Lab 1")
<br></br>

## Lab 2 – Onboarding, Identity, and a Modern Client

[![Lab2](images/lab2.png)](./lab2/README.md "Lab 2")
<br></br>

## Lab 3 – Carving Out Our First Multi-Tenant Serverless Microservice

[![Lab3](images/lab3.png)](./lab3/README.md "Lab 3")
<br></br>

## Lab 4 – Extracting the Remaining Service — Goodbye Monolith!

[![Lab4](images/lab4.png)](./lab4/README.md "Lab 4")

[Proceed to Lab 1 when you are ready to begin.](./lab1/README.md)

## License

This workshop is licensed under the MIT-0 License. See the LICENSE file.

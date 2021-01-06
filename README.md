# <p align="center">AWS SaaS Factory Monolith to Serverless SaaS Workshop</p>

## Overview

단일 테넌트 모놀리식 아키텍처에서 멀티테넌트 모던 아키텍처로의 전환은 많은 조직에게 어려운 도전이 될 수 있습니다. 모놀리식 환경에서의 밀접한 결합과 종속성으로 인해 시스템을 마이크로서비스로 옮기기가 특히나 어렵습니다.

이러한 복잡성을 감안할 때 많은 조직에서는 "빅뱅" 으로 여러분의 시스템을 다시 만들기 보다는 점진적으로 현대적인 멀티테넌트 아키텍처로 변환되는 방식으로 마이그레이션을 시도할 것 입니다.
이러한 접근법은 점진적으로 기존 아키텍처를 새로운 아키텍처 모델로 이동 하는 중에 기존 고객들을 계속해서 지원 할 수 있는 방법을 찾고 있는 조직/비지니스에 적합 할것 입니다.

이번 워크샵의 목표는 모놀리식에서 서버리스 SaaS 로 마이그레이션을 통해 마이그레이션 과정에서 발생하는 일반적인 문제들을 심층적으로 살펴보는데 있습니다. 확실히 각 솔루션들은 그들만의 고유한 마이그레이션 도전 과제(문제)를 갖고 있을 겁니다. 하지만 실제로 동작하는 예제 애플리케이션을 통해 모놀리식 환경을 멀티테넌트 SaaS 모델로 이동하는 접근 방법을 구체화 하는데 도움이 되는 패턴들과 전략을 얻을 수 있을 것입니다.

이 워크샵에서는 실제로 동작하는 샘플 애플리케이션을 위해 전통적인 모놀리식 아키텍처로 시작할것 입니다. 그런 다음 단일테넌트 모놀리식 아키텍처의 요소를 최신 멀티테넌트 솔루션으로 점진적으로 마이그레이션할 것 입니다. 여기에는 S3에서 호스팅되는 최신 웹 애플리케이션으로의 이동, API Gateway 도입, 애플리케이션 계층을 서버리스 마이크로서비스로 분해, 단일 데이터베이스에서 데이터 분리후 이 데이터에 대한 관리를 대신하는 개별 마이크로서비스로 이동하는 작업이 포함됩니다. 또한 여러분들의 솔루션에 도입할 만 한 멀티테넌트 모범 사례를 소개할 것 입니다.

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

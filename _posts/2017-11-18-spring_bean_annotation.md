---
layout: post
title: Spring Bean inject Annotation
category: spring
tag: spring
---

## Bean inject annotation

### @Componenet

스프링 컴포넌트 클래스를 지정하는 형식 단계의 어노테이션이다. (컨트롤러, 서비스 및 DAO를 지정하는 특화된 어노테이션 등)

하지만 각 컴포넌트 성격에 맞는 어노테이션을 지정하는 것이 좋다



해당 어노테이션이 있는 클래스가 빈으로 등록되기 위해서는 아래와 같은 설정이 있어야한다. (applicationContext.xml)

```Xml
<context:component-scan base-package="sample.spring"/>
```



### @Autowired

형식을 기준으로 한 의존성 자동 연결할 때 사용하는 어노테이션이다, 생성자, 메서드, 필드 단계에 사용할 수 있다

| 단계   | 어떤 빈        |
| ---- | ----------- |
| 필드   | 필드에 명시된 클래스 |
| 메서드  | 메서드의 인자     |
| 생성자  | 생성자의 인자     |



@Autowired을 사용할 때 필수 형식과 일치하는 빈이 발견되지 않으면 예외가 발생한다. 예외가 발생하지 않도록, 즉 의존성 자동 연결을 하지 않으려면 `required` 특성을 false로 하면 된다



### @Qualifier

이름을 기준으로 한 의존성 자동 연결

@Qualifier와 @Autowired를 함께 사용해 이름을 기준으로 의존성을 자동 연결할 수 있다. 스프링은 먼저 @Autowired가 지정된 필드, 생성자, 메서드에서 형식을 기준으로 자동 연결 후보를  찾은 다음, @Qualifier가 지정된 빈 이름을 이용해 자동 연결 후보 목록에서 고유한 빈을 찾는다



### @Inject

JSR 330(자바 의존성 주입)은 자바 플랫폼을 위한 의존성 주입 어노테이션을 표준화해서 스프링의 @Autowired 및 @Qualifier 와 비슷한 @Inject, @Named를 정의하고 있다



이 어노테이션을 사용하기 위해서 아래 dependency가 필요하다

```Xml
<dependency>
	<groupId>javax.inject</groupId>
  	<artifactId>javax.inject</artifactId>
  	<version>1</version>
</dependency>
```



### @Resource

JSR 250의 @Resource을 통해 필드와 메서드를 이름을 기준으로 자동 연결하도록 지원한다. @Autowired, @Qualifier의 조합을 이용하는 것보다 더 좋은 점은 바로 이름을 기준으로 빈을 찾는다는 점이다


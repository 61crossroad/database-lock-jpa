Querydsl을 사용할 때 Q 클래스 에러를 잡기가 까다로웠습니다.

하지만 Gradle이 계속 버전업을 하면서 Annotation Processor가 안정화되고, 이 덕분에 Querydsl을 설정하기가 꽤 수월해졌는데요. 현재 정착된 애노테이션 프로세서 방식을 사용하면 손쉽게 해결할 수 있습니다.

## 참고한 문서

- Gradle의 Annotation Processor 히스토리도 알고 싶다면...

[http://honeymon.io/tech/2020/07/09/gradle-annotation-processor-with-querydsl.html](http://honeymon.io/tech/2020/07/09/gradle-annotation-processor-with-querydsl.html)

## 적용 방법 요약

- build.gradle에 기본적으로 필요한 의존성은 다음과 같습니다. (버전은 mvnrepository에서 적당히...)
- gradle 6.X 이후에는 compile → implementation으로 변경합니다.

```java
// querydsl-jpa: jpa와 연동
compile('com.querydsl:querydsl-jpa:4.1.4')

// querydsl-core: querydsl 핵심 라이브러리
compile('com.querydsl:querydsl-core:4.1.4')

// querydsl-apt: annotationProcessor와 연동
annotationProcessor('com.querydsl:querydsl-apt:4.4.0:jpa')
```

- 일단 위 3줄을 넣고 빌드를 해봅니다.
- 빌드가 실패하면 에러 로그를 잘 보고 각 경우에 해당하는 의존성을 추가로 넣어줍니다.
- api-covers에 이 방식을 처음 적용했는데, 그때는 잘 몰라서 전부 때려넣었습니다. (...)

```java
// java.lang.NoClassDefFoundError(javax.annotation.Entity) 발생 대응
annotationProcessor("jakarta.persistence:jakarta.persistence-api")

// java.lang.NoClassDefFoundError (javax.annotation.Generated) 발생 대응
annotationProcessor("jakarta.annotation:jakarta.annotation-api")

// @Inject 심볼 오류 대응
compile('javax.inject:javax.inject:1')

// javax.annotation-api 찾을 수 없는 경우 (버전은 상황에 맞게..)
compile('javax.annotation:javax.annotation-api:1.3.2')
annotationProcessor('javax.annotation:javax.annotation-api:1.3.2')

// org.hibernate.javax.persistence 오류 대응 (버전은 상황에 맞게..)
annotationProcessor('org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final')
```

- 실제로 오늘 api-product의 gradle_querydsl 브랜치에 추가한 의존성은 아래와 같습니다.

```java
compile('org.projectlombok:lombok')
// api-product에 롬복의 annotationProcessor 설정이 없어서 querydsl과 충돌이 발생했었습니다.
// 빌드 에러가 계속 난다면 애노테이션 프로세서를 사용하는 의존 라이브러리들의
// compile/implementation과 함께 아래 설정도 같이 추가가 되었는지 확인해보면 좋겠습니다.
annotationProcessor('org.projectlombok:lombok')

compile('com.querydsl:querydsl-jpa:4.1.4')
compile('com.querydsl:querydsl-core:4.1.4')
compile('javax.inject:javax.inject:1')
annotationProcessor('com.querydsl:querydsl-apt:4.4.0:jpa')
```

## Q 클래스 생성 위치

- 위의 의존성 설정으로 빌드가 잘 되면 Q 클래스들은 빌드 폴더(out, build, bin 등...)에 생성됩니다.
- 저는 개인적으로 Q 클래스를 빌드된 클래스들과 놓는걸 선호합니다.
- .gitignore에 굳이 src/main/generated를 추가할 필요가 없고, Q 클래스는 엄밀히 말하면 소스에 해당하지는 않다고 생각해서요.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8fe8c46e-9012-4f7c-997e-62cfc147c602/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8fe8c46e-9012-4f7c-997e-62cfc147c602/Untitled.png)

- 이 위치보다는 src/main/generated에 Q클래스를 만들길 윈한다면 약간의 빌드 스크립트를 추가해줍니다.
- 컴파일할 때 지정한 폴더(generated = 'src/main/generated')에 AnnotationProcessor가 생성한 파일을 넣고, gradle clean 실행 시 generated폴더도 같이 삭제합니다.

```java
def generated = 'src/main/generated'

sourceSets {
	main.java.srcDirs += [ generated ]
}

tasks.withType(JavaCompile) {
	options.annotationProcessorGeneratedSourcesDirectory = file(generated)
}

clean.doLast {
	file(generated).deleteDir()
}
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/10baa905-a038-4df1-87d0-b2694da826e1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/10baa905-a038-4df1-87d0-b2694da826e1/Untitled.png)

이런 식으로 src/main/generated에 Q 클래스를 생성했습니다.

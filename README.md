# rsql-jpa-specification

Translate RSQL query into org.springframework.data.jpa.domain.Specification or com.querydsl.core.types.Predicate and support entities association query.

[Supported Operators](https://github.com/perplexhub/rsql-jpa-specification/blob/master/rsql-common/src/main/java/io/github/perplexhub/rsql/RSQLOperators.java)

## Maven Repository

<https://oss.sonatype.org/#nexus-search;gav~io.github.perplexhub~rsql*>

## Add rsql-jpa-spring-boot-starter for RSQL to Spring JPA translation

### Maven dependency for rsql-jpa-spring-boot-starter

```xml
  <dependency>
    <groupId>io.github.perplexhub</groupId>
    <artifactId>rsql-jpa-spring-boot-starter</artifactId>
    <version>5.0.1</version>
  </dependency>
```

### Add JpaSpecificationExecutor to your JPA repository interface classes

```java
package com.perplexhub.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

import com.perplexhub.model.User;

public interface UserRepository extends JpaRepository<User, String>, JpaSpecificationExecutor<User> {
}
```

## Add rsql-querydsl-spring-boot-starter for RSQL to Spring JPA and QueryDSL translation

### Maven dependency for rsql-querydsl-spring-boot-starter

```xml
  <dependency>
    <groupId>io.github.perplexhub</groupId>
    <artifactId>rsql-querydsl-spring-boot-starter</artifactId>
    <version>5.0.1</version>
  </dependency>
```

### Add JpaSpecificationExecutor and QuerydslPredicateExecutor to your JPA repository interface classes

```java
package com.perplexhub.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.querydsl.QuerydslPredicateExecutor;

import com.perplexhub.model.User;

public interface UserRepository extends JpaRepository<User, String>, JpaSpecificationExecutor<User>, QuerydslPredicateExecutor<User> {
}
```

## Use below properties to control the version of Spring Boot, Spring Data and QueryDSL

```xml
  <properties>
    <spring-boot.version>2.0.0.RELEASE</spring-boot.version>
    <spring-data-releasetrain.version>Kay-RELEASE</spring-data-releasetrain.version>
    <querydsl.version>4.1.4</querydsl.version>
  </properties>
```

## RSQL Syntax Reference

```java
filter = "id=bt=(2,4)";// id>=2 && id<=4 //between
filter = "id=nb=(2,4)";// id<2 || id>4 //not between
filter = "company.code=like=em"; //like %em%
filter = "company.code=ilike=EM"; //ignore case like %EM%
filter = "company.code=icase=EM"; //ignore case equal EM
filter = "company.code=notlike=em"; //not like %em%
filter = "company.code=inotlike=EM"; //ignore case not like %EM%
filter = "company.code=ke=e*m"; //like %e*m%
filter = "company.code=ik=E*M"; //ignore case like %E*M%
filter = "company.code=nk=e*m"; //not like %e*m%
filter = "company.code=ni=E*M"; //ignore case not like %E*M%
filter = "company.code=ic=E^^M"; //ignore case equal E^^M
filter = "company.code==demo"; //equal
filter = "company.code=='demo'"; //equal
filter = "company.code==''"; //equal to empty string
filter = "company.code==dem*"; //like dem%
filter = "company.code==*emo"; //like %emo
filter = "company.code==*em*"; //like %em%
filter = "company.code==^EM"; //ignore case equal EM
filter = "company.code==^*EM*"; //ignore case like %EM%
filter = "company.code=='^*EM*'"; //ignore case like %EM%
filter = "company.code!=demo"; //not equal
filter = "company.code=in=(*)"; //equal to *
filter = "company.code=in=(^)"; //equal to ^
filter = "company.code=in=(demo,real)"; //in
filter = "company.code=out=(demo,real)"; //not in
filter = "company.id=gt=100"; //greater than
filter = "company.id=lt=100"; //less than
filter = "company.id=ge=100"; //greater than or equal
filter = "company.id=le=100"; //less than or equal
filter = "company.id>100"; //greater than
filter = "company.id<100"; //less than
filter = "company.id>=100"; //greater than or equal
filter = "company.id<=100"; //less than or equal
filter = "company.code=isnull=''"; //is null
filter = "company.code=null=''"; //is null
filter = "company.code=na=''"; //is null
filter = "company.code=nn=''"; //is not null
filter = "company.code=notnull=''"; //is not null
filter = "company.code=isnotnull=''"; //is not null

filter = "company.code=='demo';company.id>100"; //and
filter = "company.code=='demo' and company.id>100"; //and

filter = "company.code=='demo',company.id>100"; //or
filter = "company.code=='demo' or company.id>100"; //or
```

Syntax Reference: [RSQL / FIQL parser](https://github.com/jirutka/rsql-parser#examples)

## Spring Data JPA Specification

```java
Pageable pageable = PageRequest.of(0, 5); //page 1 and page size is 5

repository.findAll(RSQLSupport.toSpecification(filter));
repository.findAll(RSQLSupport.toSpecification(filter), pageable);

repository.findAll(RSQLSupport.toSpecification(filter, true)); // select distinct
repository.findAll(RSQLSupport.toSpecification(filter, true), pageable);

// use static import
import static io.github.perplexhub.rsql.RSQLSupport.*;

repository.findAll(toSpecification(filter));
repository.findAll(toSpecification(filter), pageable);

repository.findAll(toSpecification(filter, true)); // select distinct
repository.findAll(toSpecification(filter, true), pageable);

// property path remap
filter = "compCode=='demo';compId>100"; // "company.code=='demo';company.id>100" -  protect our domain model #10

Map<String, String> propertyPathMapper = new HashMap<>();
propertyPathMapper.put("compId", "company.id");
propertyPathMapper.put("compCode", "company.code");

repository.findAll(toSpecification(filter, propertyPathMapper));
repository.findAll(toSpecification(filter, propertyPathMapper), pageable);
```

## QueryDSL Predicate (BooleanExpression)

```java
Pageable pageable = PageRequest.of(0, 5); //page 1 and page size is 5

repository.findAll(RSQLSupport.toPredicate(filter, QUser.user));
repository.findAll(RSQLSupport.toPredicate(filter, QUser.user), pageable);

// use static import
import static io.github.perplexhub.rsql.RSQLSupport.*;

repository.findAll(toPredicate(filter, QUser.user));
repository.findAll(toPredicate(filter, QUser.user), pageable);

// property path remap
filter = "compCode=='demo';compId>100"; // "company.code=='demo';company.id>100" - protect our domain model #10

Map<String, String> propertyPathMapper = new HashMap<>();
propertyPathMapper.put("compId", "company.id");
propertyPathMapper.put("compCode", "company.code");

repository.findAll(toPredicate(filter, QUser.user, propertyPathMapper));
repository.findAll(toPredicate(filter, QUser.user, propertyPathMapper), pageable);
```

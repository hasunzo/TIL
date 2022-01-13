## @DataJpaTest
- JPA 구성 요소에만 초점을 맞춘 JPA 테스트에 대한 어노테이션이다.
- @DataJpaTest를 사용하면 full-auto-configuration이 비활성화되고 대신에 JPA 테스트와 관련된 구성만 적용된다.
- 따라서 통합테스트인 @SpringBootTest를 사용했을 때보다 가볍다! 시간도 당연히 빠르다! 단위테스트로 사용하기 적절!
- @DataJpaTest는 트랜잭션이 포함되어 있어, 테스트가 종료되면 자동으로 롤백된다.
- 기본적으로 내장된 인메모리 데이터베이스를 사용한다.
    - 이를 변경하고자 할 시 @AutoConfigureTestDatabase를 사용하면 된다.
  
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(DataJpaTestContextBootstrapper.class)
@ExtendWith({SpringExtension.class})
@OverrideAutoConfiguration(
        enabled = false
)
@TypeExcludeFilters({DataJpaTypeExcludeFilter.class})
@Transactional
@AutoConfigureCache
@AutoConfigureDataJpa
@AutoConfigureTestDatabase
@AutoConfigureTestEntityManager
@ImportAutoConfiguration
public @interface DataJpaTest { ... }
```
- 트랜잭션이 포함되어 있는 것을 확인할 수 있다.

### @DataJpaTest에 실 데이터베이스 적용하기
- 디폴트가 인메모리를 사용하도록 되어있기 때문에 실 데이터베이스를 사용하기 위해서는 추가 설정이 필요하다.
- @AutoConfigureTestDatabase(replace = Replace.NONE)을 추가하면 실 데이터베이스에 테스트가 가능하다.

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@ImportAutoConfiguration
@PropertyMapping("spring.test.database")
public @interface AutoConfigureTestDatabase {
    @PropertyMapping(
        skip = SkipPropertyMapping.ON_DEFAULT_VALUE
    )
    AutoConfigureTestDatabase.Replace replace() default AutoConfigureTestDatabase.Replace.ANY;

    EmbeddedDatabaseConnection connection() default EmbeddedDatabaseConnection.NONE;

    public static enum Replace {
        ANY,
        AUTO_CONFIGURED,
        NONE;

        private Replace() {
        }
    }
}
```
- Replace의 디폴트가 ANY로 설정되어 있는 것을 확인할 수 있다.

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = NONE)
public class InfluencerRepositoryTests {...}
```
- 위와 같이 설정해주면 실 데이터베이스에 테스트를 할 수 있다.

### application으로 설정하기
```properties
spring.test.database.replace: none
```
- 이렇게 설정할 수도 있다.

> 참고
> https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html
> https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/AutoConfigureDataJpa.html

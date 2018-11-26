구성
- spring-boot-starter-webflux (2.1.0.RELEASE)
- springfox-swagger2 (3.0.0-SNAPSHOT)

우선 `pom.xml`에 관련 dependency들을 추가해 줍니다.

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <swagger.version>3.0.0-SNAPSHOT</swagger.version>
</properties>

<dependencies>
    <!-- srping webflux -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <!-- swagger 3 -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>${swagger.version}</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>${swagger.version}</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-spring-webflux</artifactId>
        <version>${swagger.version}</version>
    </dependency>
</dependencies>
```

이후 Docket 빈을 생성해주시면 됩니다.

```java
@Configuration
@EnableSwagger2WebFlux
public class SwaggerConfig {
	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
			.select()
			.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
			.build()
			.genericModelSubstitutes(Optional.class, Flux.class, Mono.class);
	}
}
```

spring webflux를 사용하시면 api 응답값으로 `Mono<T>`, `Flux<T>`를 사용하실 텐데요, `.genericModelSubstitutes(Optional.class, Flux.class, Mono.class);`를 추가해주시지 않으면 아래 이미지 처럼 Response 모델이 정상적으로 보이지 않습니다.

![swagger](https://raw.githubusercontent.com/kkd927/til/master/img/Swagger_UI.png)

정상적으로 SwaggerConfig에 추가해주셨다면 아래처럼 잘 나오게됩니다.

![swagger](https://raw.githubusercontent.com/kkd927/til/master/img/Swagger_UI-2.png)

이제 api 작성을 통해 테스트를 진행하시면 됩니다.

```java
@RestController
public class PostController {
	private final PostService postService;

	public PostController(PostService postService) {
		this.postService = postService;
	}

	@ApiOperation(value = "Post 생성")
	@PostMapping("/boards/{boardId}/posts")
	public Mono<PostDTO> addPost(@PathVariable Long boardId, @RequestBody PostRequestDTO postRequestDTO) {
		// ...
	}
}
```

이제 `http://localhost:8080/swagger-ui.html`에 접속하셔서 정상적으로 swagger가 동작하는지 확인하시면 됩니다.

![swagger](https://raw.githubusercontent.com/kkd927/til/master/img/Swagger_UI-3.png)


## 참고

- https://springfox.github.io/springfox/docs/current/

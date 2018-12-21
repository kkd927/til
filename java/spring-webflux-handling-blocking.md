Obtaining a Flux or a Mono doesn’t necessarily mean it will run in a dedicated Thread. Instead, most operators continue working in the Thread on which the previous operator executed. Unless specified, the topmost operator (the source) itself runs on the Thread in which the subscribe() call was made.

기본적인 controller -> service -> repository 구조로 테스트.

```java
@Slf4j
@RestController
public class MemoController {
	private final MemoService memoService;

	public MemoController(MemoService memoService) {
		this.memoService = memoService;
	}

	@PostMapping("/memos")
	public Mono<MemoResponseDTO> save(@Valid @RequestBody MemoRequestDTO memoRequestDTO) {
		log.info("===== MemoController 시작 =====");

		return memoService.save(memoRequestDTO)
			.map(MemoResponseDTO::of)
			.log();
	}
}
```

```java
@Slf4j
@Service
public class MemoService {
	private final MemoRepository memoRepository;

	public MemoService(MemoRepository memoRepository) {
		this.memoRepository = memoRepository;
	}

	public Mono<Memo> save(MemoRequestDTO memoRequestDTO) {
		log.info("===== MemoService 시작 =====");

		return Mono.just(memoRepository.save(Memo.of(memoRequestDTO)))
			.log();
	}
}
```

```java
@Slf4j
@Repository
public class MemoRepository {
	public Memo save(Memo memo) {
		log.info("===== MemoRepository 시작 =====");
        // ...blocking persistence APIs (JPA, JDBC) or networking APIs to use
		return memo;
	}
}
```

```java
2018-12-21 14:09:20.705  INFO 67826 --- [ctor-http-nio-2] c.e.demo.interfaces.MemoController       : ===== MemoController 시작 =====
2018-12-21 14:09:20.706  INFO 67826 --- [ctor-http-nio-2] c.e.demo.domain.service.MemoService      : ===== MemoService 시작 =====
2018-12-21 14:09:20.711  INFO 67826 --- [ctor-http-nio-2] c.e.d.domain.repository.MemoRepository   : ===== MemoRepository 시작 =====
2018-12-21 14:09:20.728  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.CallableOnAssembly.1        : onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 14:09:20.729  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 14:09:20.729  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | request(unbounded)
2018-12-21 14:09:20.729  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.CallableOnAssembly.1        : request(unbounded)
2018-12-21 14:09:20.730  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.CallableOnAssembly.1        : onNext(com.example.demo.domain.model.Memo@3b09310d)
2018-12-21 14:09:20.730  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | onNext(com.example.demo.domain.dto.MemoResponseDTO@67c2f017)
2018-12-21 14:09:20.751  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.CallableOnAssembly.1        : onComplete()
2018-12-21 14:09:20.752  INFO 67826 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | onComplete()
```

그냥... blocking 코드와 다를바가 없어 보임.


blocking인 `MemoRepository`를 호출하는 `MemoService`에 `subscribeOn()`을 통해 Scheduler만 등록한 경우

```java
public Mono<Memo> save(MemoRequestDTO memoRequestDTO) {
    log.info("===== MemoService 시작 =====");

    return Mono.just(memoRepository.save(Memo.of(memoRequestDTO)))
        .subscribeOn(Schedulers.elastic())
        .log();
}
```


```java
2018-12-21 14:15:52.898  INFO 69150 --- [ctor-http-nio-2] c.e.demo.interfaces.MemoController       : ===== MemoController 시작 =====
2018-12-21 14:15:52.899  INFO 69150 --- [ctor-http-nio-2] c.e.demo.domain.service.MemoService      : ===== MemoService 시작 =====
2018-12-21 14:15:52.904  INFO 69150 --- [ctor-http-nio-2] c.e.d.domain.repository.MemoRepository   : ===== MemoRepository 시작 =====
2018-12-21 14:15:52.927  INFO 69150 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.1                : | onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 14:15:52.928  INFO 69150 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 14:15:52.928  INFO 69150 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | request(unbounded)
2018-12-21 14:15:52.929  INFO 69150 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.1                : | request(unbounded)
2018-12-21 14:15:52.931  INFO 69150 --- [      elastic-2] reactor.Mono.OnAssembly.1                : | onNext(com.example.demo.domain.model.Memo@56624c86)
2018-12-21 14:15:52.932  INFO 69150 --- [      elastic-2] reactor.Mono.OnAssembly.2                : | onNext(com.example.demo.domain.dto.MemoResponseDTO@175f46bd)
2018-12-21 14:15:52.948  INFO 69150 --- [      elastic-2] reactor.Mono.OnAssembly.1                : | onComplete()
2018-12-21 14:15:52.948  INFO 69150 --- [      elastic-2] reactor.Mono.OnAssembly.2                : | onComplete()

```

Blocking 로직이 있는 `MemoRepository`가 최초 요청을 받은 worker thread에서 돌아감. 결국 무의미.

So... [How do I wrap a synchronous, blocking call?](https://projectreactor.io/docs/core/release/reference/#faq.wrap-blocking)

A simple way to evaluate an application is to check its dependencies. If you have blocking persistence APIs (JPA, JDBC) or networking APIs to use, Spring MVC is the best choice for common architectures at least. It is technically feasible with both Reactor and RxJava to perform blocking calls on a separate thread but you would not be making the most of a non-blocking web stack.

Use `Callable` to defer execution. And you should use `Schedulers.elastic` because it creates a dedicated thread to wait for the blocking resource without tying up some other resource.

- Schedulers.immediate() : Current thread.
- Schedulers.single() : A single, reusable thread.
- Schedulers.newSingle() : A per-call dedicated thread.
- Schedulers.elastic() : An elastic thread pool. It creates new worker pools as needed, and reuse idle ones. This is a good choice for I/O blocking work for instance.
- Schedulers.parallel() : A fixed pool of workers that is tuned for parallel work.

```java
public Mono<Memo> save(MemoRequestDTO memoRequestDTO) {
    log.info("===== MemoService 시작 =====");

    return Mono.fromCallable(() -> memoRepository.save(Memo.of(memoRequestDTO)))
            .subscribeOn(Schedulers.elastic())
            .log();
}
```

```java
2018-12-21 13:34:36.926  INFO 52520 --- [ctor-http-nio-2] c.e.demo.interfaces.MemoController       : ===== MemoController 시작 =====
2018-12-21 13:34:36.926  INFO 52520 --- [ctor-http-nio-2] c.e.demo.domain.service.MemoService      : ===== MemoService 시작 =====
2018-12-21 13:34:36.982  INFO 52520 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.1                : | onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 13:34:36.982  INFO 52520 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | onSubscribe([Fuseable] FluxOnAssembly.OnAssemblySubscriber)
2018-12-21 13:34:36.983  INFO 52520 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.2                : | request(unbounded)
2018-12-21 13:34:36.983  INFO 52520 --- [ctor-http-nio-2] reactor.Mono.OnAssembly.1                : | request(unbounded)
2018-12-21 13:34:36.990  INFO 52520 --- [      elastic-2] c.e.d.domain.repository.MemoRepository   : ===== MemoRepository 시작 =====
2018-12-21 13:34:41.994  INFO 52520 --- [      elastic-2] reactor.Mono.OnAssembly.1                : | onNext(com.example.demo.domain.model.Memo@3c0aa92b)
2018-12-21 13:34:41.995  INFO 52520 --- [      elastic-2] reactor.Mono.OnAssembly.2                : | onNext(com.example.demo.domain.dto.MemoResponseDTO@1c14c9b0)
2018-12-21 13:34:42.013  INFO 52520 --- [      elastic-2] reactor.Mono.OnAssembly.1                : | onComplete()
2018-12-21 13:34:42.013  INFO 52520 --- [      elastic-2] reactor.Mono.OnAssembly.2                : | onComplete()
```


## 참고

- https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-framework-choice
- https://spring.io/blog/2016/07/20/notes-on-reactive-programming-part-iii-a-simple-http-server-application#bridging-from-blocking-to-reactive-with-spring-mvc
- https://projectreactor.io/docs/core/release/reference/#schedulers
- https://projectreactor.io/docs/core/release/reference/#faq.wrap-blocking

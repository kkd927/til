### Spring MVC Last-Modified, If-Modified-Since 설정

Spring MVC에서 [Last-Modified](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Last-Modified)와 [If-Modified-Since](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/If-Modified-Since) 헤더를 통한 캐싱방법을 살펴보겠습니다. 웹 캐싱에 대한 이론적인 부분과 종류들은 다른 설명 글들이 많기 때문에 자세한 내용은 생략하도록 하겠습니다 ([링크](http://cyberx.tistory.com/9)).

캐싱의 순서는 다음과 같이 이루어집니다.

1. 브라우저에서 필요한 리소스를 서버에 최초로 요청합니다.

2. 서버에서 응답헤더에 `Last-Modified` 헤더를 셋팅하여 요청한 리소스와 함께 내려보내면, 브라우저는 해당 리소스의 복사본을 생성하여 저장합니다.

3. 브라우저에서 해당 리소스를 재요청할 경우에는 서버에서 `Last-Modified` 헤더에 설정한 값을 `If-Modified-Since` 헤더에 포함시켜 서버에 요청합니다.

4. 서버에서는 `If-Modified-Since` 헤더 값을 통해 해당 리소스가 변경되었으면 `Last-Modified` 헤더에 최신 수정날짜를 설정하여 파일을 전달합니다. 변경되지 않았을 경우 [304 HTTP 상태코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/304)로 응답하면 브라우저는 기존에 저장한 값을 재사용하게 됩니다.


스프링에서는 `ServletWebRequest` 객체에 `checkNotModified()` 메서드를 구현하여 일련의 과정들을 큰 작업없이 가능하게 해줍니다.

해당 메서드는 리소스의 최근 수정일을 인자로 받아 브라우저에서 요청할 때 설정해주는 `If-Modified-Since` 헤더 값과 값을 비교하여 리소스가 변경이 있는 경우 `true`를 반환하고 응답 값에 자동으로 304 NOT Modified를 설정해줍니다. 변경이 없는 경우에는 `false`를 반환합니다.

```java
boolean checkNotModified(long lastModifiedTimestamp);
```

브라우저에서 test.txt 파일을 요청한다고 가정해봅시다.

```java
@GetMapping("/test")
public ResponseEntity<byte[]> test(ServletWebRequest request) throws IOException {
    Resource resource = new ClassPathResource("test.txt");

    // test.txt 파일의 최근 수정일을 인자값으로 넣어
    // 변경이 일어났는지 여부를 확인합니다.
    if (request.checkNotModified(resource.lastModified())) {
        // 변경이 없는 경우(혹은 최초 요청인 경우)
        // 빈 본문 값과 함께 304 상태코드가 반환됩니다.
        return null;
    }

    return ResponseEntity.ok()
        .cacheControl(CacheControl.noCache())
        // Last-Modified 헤더에 파일의 최근 수정일을 넣어 응답합니다.
        .lastModified(resource.lastModified())
        .body(FileCopyUtils.copyToByteArray(resoure.getFile()));
}
```

이제 실제 브라우저에서의 동작이 어떻게 되는지 살펴보겠습니다.

브라우저가 최초 요청 시 서버의 응답을 살펴보면 아래 이미지 처럼 200 상태코드와 함께 `Last-Modified` 헤더에 해당 파일의 최근 수정일을 설정하여 내려보내지게 됩니다.

![response image](https://raw.githubusercontent.com/kkd927/til/master/img/index_template.png)


두 번째 요청 시 아래 이미지 처럼 전에 서버에서 `Last-Modified` 헤더에 응답해준 값을 `If-Modified-Since` 헤더에 설정하여 요청하게 됩니다.

![request image](https://raw.githubusercontent.com/kkd927/til/master/img/cache2.png)

그러면 서버에서는 아래 이미지 처럼 `304 Not Modified` 상태코드를 반환하며 해당 파일의 내용을 전달하지 않습니다. 브라우저는 로컬에 캐싱해둔 파일을 재사용하게 됩니다.

![reponse2 image](https://raw.githubusercontent.com/kkd927/til/master/img/cache3.png)

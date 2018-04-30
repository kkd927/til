Java SE 8이 2014년 3월에 출시되고 3년 반이 흘러 Java SE 9이 2017년 9월에 출시되었습니다. 그리고 2018년 3월에 Java SE 10이 6개월만에 출시되었습니다. Java 10이 6개월만에 출시된 이유는 오라클이 클라우드 주도 개발자(cloud-driven developer)들을 지원하기 위해 앞으로 자바의 릴리즈 주기를 6개월로 변경하였기 때문입니다. 이는 Java 9부터 적용되며 매년 3월, 9월에 새로운 릴리즈가 출시됩니다. 

또한 3년 단위로 엔터프라이즈 환경을 위해 안정성에 초점을 맞춘 Long-Term Support (LTS) 버전이 출시될 예정입니다. Java 11이 차세대 LTS 버전으로 2021년 9월에 출시될 예정입니다. Java Platform Group, Oracle의 Chief Architect인 Mark Reinhold는 다음과 같이 설명했습니다.

> OpenJDK binaries become the primary channel for developers to access the latest innovation in the Java SE platform, the Oracle JDK will remain as a long term support LTS offering for Oracle’s commercial and support customers.

이제부터 Java 10에 새로 추가된 12가지 특징들을 살펴보겠습니다.

### 1. JEP 286: Local-Variable Type Inference

Java 10의 변화 중 코드 측면에서 가장 흥미로운 점을 하나 고르라면 당연 Local Variable Type Inference 입니다. 로컬변수 선언을 `var` 를 이용하여 기존의 엄격한 타입 선언 방식에서 탈피하여 컴파일러에게 타입을 추론하게할 수 있습니다. 기존에 lombok에서 제공하는 val/var 기능을 사용하고 계셨다면 크게 생소하진 않을 내용입니다.

```java
var list = new ArrayList<String>();  // infers ArrayList<String>
var stream = list.stream();          // infers Stream<String>
```

Local Variable Type Inference는 다음과 같은 상황에서만 사용할 수 있습니다.

- 초기화된 로컬 변수 선언 시
- 반복문에서 지역변수 선언 시 (enhanced for loop 포함)

```javascript
var numbers = List.of(1, 2, 3, 4, 5);

for (var number : numbers) {
	System.out.println(number);
}

for (var i = 0; i < numbers.size(); i++) {
	System.out.println(numbers.get(i));
}
```

Java 9에서 등장한 [jshell](https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm#JSHEL-GUID-630F27C8-1195-4989-9F6B-2C51D46F52C8)를 이용하면 쉽게 테스트할 수 있습니다.

```bash
jshell> var a = new ArrayList<>();
a ==> []

jshell> a.add("abc")
$2 ==> true

jshell> a.add(123)
$3 ==> true

jshell> a.get(0)
$4 ==> "abc"

jshell> a.get(1)
$5 ==> 123

jshell> a = "123"
|  Error:
|  incompatible types: java.lang.String cannot be 
|  converted to java.util.ArrayList<java.lang.Object>
|  a = "123"
|      ^---^

```

참고로 다이아몬드 연산자 안에 타입을 넣지 않으면 ArrayList<Object>로 추론합니다.

단, 주의해야할 점은 기존에 자바의 RHS(lamda, generics, diamond)에서 이미 타입 추론을 하고 있기 때문에, 해당 표현식의 LHS에 `var` 로 대입하면 추론이 실패될 수 있습니다.

```bash
jshell> var f = () -> { };
|  Error:
|  cannot infer type for local variable f
|    (lambda expression needs an explicit target-type)
|  var f = () -> { };
|  ^----------------^

```

Local Variable Type Inference에 대해 더 자세한 내용은 [OpenJDK : JEP 286](http://openjdk.java.net/jeps/286)에서 확인하실 수 있습니다.

아래 차트는 Oracle이 Java 커뮤니티에서 Local Variable Type Inference에 관해 진행한 설문 결과입니다. 대다수가 이 변화를 환영하는 분위기네요.

![survey](https://raw.githubusercontent.com/kkd927/til/master/img/Local_Variable_Type_Inference_for_Java_-_%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%83%E1%85%A1%E1%86%B8___SurveyMonkey.png)

### 2. JEP 296: Consolidate the JDK Forest into a Single Repository

수많은 JDK forest 저장소들을 단일 저장소로 통합시켰다는 내용으로, 내부 유지보수에 관한 것으로 설명은 생략하겠습니다.

더 자세한 내용은 [OpenJDK : JEP 296](http://openjdk.java.net/jeps/296)에서 확인하실 수 있습니다.

### 3. JEP 304: Garbage-Collector Interface

간결한 garbage collector(GC) 인터페이스를 새로 추가하여 서로 다른 gc들의 코드를 분리할 수 있도록 개선시켰습니다. GC 혹은 HotSpot 개발자들에게 관련된 사항으로 설명은 생략하겠습니다.

더 자세한 내용은 [OpenJDK : JEP 304](http://openjdk.java.net/jeps/304)에서 확인하실 수 있습니다.

### 4. JEP 307: Parallel Full GC for G1

Java 9에서 default Garbage Collector인 [G1](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573)은 full GC를 피하도록 설계되었지만, concurrent GC에서 충분한 young area를 확보하지 못하면 full GC가 발생합니다. 기존 G1에서 full GC를 수행할 때 싱글 쓰레드 방식으로 동작하는 mark-sweep-compact 알고리즘을 사용하였습니다. G1 이전의 default 였던 parallel collector 처럼 G1에서도 full GC를 병렬화 시켜 G1의 최악의 케이스에서 지연 시간을 개선시켰습니다. 

mark-sweep-compact 알고리즘을 병렬로 수행하도록 변경되었으며, 이 때 Young 과 Mixed GC와 동일한 쓰레드 수를 이용힙니다. `-XX:ParallelGCThreads` 옵션을 이용해 쓰레드 수를 조절할 수 있으며, 이 옵션을 수정하면 Young 과 Mixed GC에도 변경사항이 적용됩니다.

더 자세한 내용은 [OpenJDK : JEP 307](http://openjdk.java.net/jeps/307)에서 확인하실 수 있습니다.

### 5. JEP 310: Application Class-Data Sharing

기존의 Class-Data Sharing(CDS) 기능을 확장해 애플리케이션 클래스를 공유 아카이브에 배치하고 서로 다른 자바 프로세스들이 공유함할 수 있도록 개선함으로써, startup 시간을 단축시키고 메모리 사용량을 최적화 시켰습니다. 기존에 AppCDS 기능은 상업용으로 Oracle JDK에서만 제공되었으나, 오픈소스화 되어 Open JDK에도 사용할 수 있게 되었습니다.

더 자세한 내용은 [OpenJDK : JEP 310](http://openjdk.java.net/jeps/310)에서 확인하실 수 있습니다.

### 6. JEP 312: Thread-Local Handshakes

[JVM safepoint](http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html)는 Stop the World를 의미합니다. Thread-Local Handshake는 이런 safepoint가 발생하는 지점들을 줄이기 위한 프로젝트입니다. 모든 쓰레드를 동시에 멈춰야 했던 기존과 달리 쓰레드를 개별로 멈출 수 있게 되었고, VM safepoint를 수행하지 않고도 개별 Thread에서 콜백을 실행할 수 있습니다.

더 자세한 내용은 [OpenJDK : JEP 312](http://openjdk.java.net/jeps/312)에서 확인하실 수 있습니다.

### 7. JEP 313: Remove the Native-Header Generation Tool (javah)

`${JAVA_HOME}/bin` 디렉터리 안에는 JDK에서 제공하는 수많은 툴들이 존재합니다. javah는 그 중에 하나로, 코드에 native 메서드들을 사용할 경우 JNI 헤더 파일들을 생성해줍니다. Java 8부터 javac에서 JNI 헤더 파일 생성을 지원하기 때문에 javah이 더 이상 필요하지 않아 제거되었습니다.

더 자세한 내용은 [OpenJDK : JEP 313](http://openjdk.java.net/jeps/313)에서 확인하실 수 있습니다.

### 8. JEP 314: Additional Unicode Language-Tag Extensions

Java SE 7부터 [BCP 47 언어 태그](https://ko.wikipedia.org/wiki/IETF_%EC%96%B8%EC%96%B4_%ED%83%9C%EA%B7%B8)를 지원하기 시작했습니다. 언어 태그는 하이픈에 따라서 나뉘는, 하나 이상의 하위 태그(subtag)로부터 구성됩니다. 일반적으로는 다음의 순서로 쓰입니다. 

- language(언어)
- script(문자 체계)
- region(지역)
- variant(변이형)
- extension(확장)
- private use(개인용)

`en-GB` 와 `en-US` 언어태그를 예로 살펴보면, en은 영어를 의미하고 GB(Great Britain)와 US(United States)는 지역을 나타냅니다. Java 9에서는 [BCP 47 U 언어태그 확장](http://www.oracle.com/technetwork/java/javase/documentation/java9locales-3559485.html) ca와 nu를 지원하기 시작했습니다. 예를들어 `th-TH-u-ca-buddhist-nu-thai`는 불교달력(buddhist)와 태국 숫자 체계(nu-thai)기반의 태국지역(TH)의 태국어(th)를 나타냅니다. Java 10에서는 다음의 추가 확장을 지원하며, 이를 지원하기 위해 java.util.Local와 관련된 API들이 보완되었습니다.

- cu (currency type)
- fw (first day of week)
- rg (region override)
- tz (time zone)

더 자세한 내용은 [OpenJDK : JEP 314](http://openjdk.java.net/jeps/314)에서 확인하실 수 있습니다.

### 9. JEP 316: Heap Allocation on Alternative Memory Devices

JVM heap 영역을 [NVDIMM](https://en.wikipedia.org/wiki/NVDIMM)(비휘발성 NAND 플래시 메모리) 혹은 사용자 지정과 같은 대체 메모리 장치에 할당할 수 있게됩니다. (이제 정전 걱정은 할 필요 없는 것일까요?) 이 기능은 새로 추가된 옵션인 `-XX:AllocateHeapAt=<path>`를 이용해 사용할 수 있습니다.

더 자세한 내용은 [OpenJDK : JEP 316](http://openjdk.java.net/jeps/316)에서 확인하실 수 있습니다.

### 10. JEP 317: Experimental Java-Based JIT Compiler

현재 HotSpot JVM에는 C1, C2라는 두개의 런타임 컴파일러를 가지고 있습니다. 이 외에도 Java 언어 기반의 [JIT 컴파일러](https://ko.wikipedia.org/wiki/JIT_%EC%BB%B4%ED%8C%8C%EC%9D%BC)인 [Graal](https://github.com/oracle/graal)이 있으며, JDK 9에 소개된 JVM compiler interface(JVMCI)를 사용하고 있습니다. Graal은 이미 JDK에 포함되어 있으며, JDK 10에서 Linux/x64 플랫폼에서 실험적으로 JIT 컴파일러로 사용할 수 있습니다. Graal을 JIT 컴파일러로 설정하기 위해선, `-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler` 옵션을 사용하면 됩니다.

더 자세한 내용은 [OpenJDK : JEP 317](http://openjdk.java.net/jeps/317)에서 확인하실 수 있습니다.

### 11. JEP 319: Root Certificates

HTTPS 암호화 통신에 쓰는 SSL/TLS 인증서를 발급해주는 인증기관을 Certificate Authority(CA)라고 부릅니다. 보통 브라우저는 [최상위 인증기관(trusted root CA)](https://en.wikipedia.org/wiki/Root_certificate) 목록을 자체적으로 가지고 있어, 인증서가 신용있다고 판단한 CA로부터 서명된 것인지 확인합니다.

Java도 root CA를 위한 저장소를 가지고 있습니다. `${JAVA_HOME}/lib/security` 디렉터리 하위에 있는 `cacerts` 파일로, 단순한 keystore입니다. 기존에 OpenJDK 버전은 빈파일이 기본 셋팅이었지만, JDK 10부터는 Oracle JDK 버전에만 있던 root CA 목록이 기본으로 포함됩니다.

더 자세한 내용은 [OpenJDK : JEP 319](http://openjdk.java.net/jeps/319)에서 확인하실 수 있습니다.

### 12. JEP 322: Time-Based Release Versioning

처음에 언급한 자바의 새로운 릴리즈 주기를 위한 버저닝(versioning) 포맷을 위한 변경사항입니다. 새로 적용될 포맷은 `[1-9][0-9]*((\.0)*\.[1-9][0-9]*)*` 형식으로 `$FEATURE.$INTERIM.$UPDATE.$PATCH` 의미를 가집니다.

- $FEATURE - 6개월 주기로 증가됩니다. 2018년 3월 버전은 JDK 10, 2018년 9월 버전은 JDK 11입니다.
- $INTERIM - 6개월 주기 릴리즈 모델에서는 interim 릴리즈가 없어 항상 0 입니다.
- $UPDATE - 보안 이슈, 버그 등의 수정이 업데이트된 버전을 나타내며 1달 단위로 증가됩니다. 2018년 4월 버전은 JDK 10.0.1, 5월 버전은 JDK 10.0.2 입니다.
- $PATCH - 치명적인 이슈를 수정하여 배포할 때만 생기는 긴급 패치 버전입니다.

![java release model](https://raw.githubusercontent.com/kkd927/til/master/img/java_release.jpg)
(이미지 출처: [Java 공식 트위터](https://twitter.com/java/status/969287657015664640/photo/1?tfw_site=laytoun&ref_src=twsrc%5Etfw&ref_url=http%3A%2F%2Faboullaite.me%2F10-new-features-in-java-10%2F))

더 자세한 내용은 [OpenJDK : JEP 322](http://openjdk.java.net/jeps/322)에서 확인하실 수 있습니다.

## 참조

- http://openjdk.java.net/projects/jdk/10/
- https://blogs.oracle.com/java-platform-group/introducing-java-se-10
- http://blog.ragozin.info/2012/10/safepoints-in-hotspot-jvm.html
- https://docs.oracle.com/javase/9/index.html
- https://docs.oracle.com/javase/10/index.html

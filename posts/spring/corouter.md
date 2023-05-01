# Coroutine & Corouter 문제

- Spring Webflux에서 Corouter에 요청 당 "traceId"를 부여해서 사용하는 경우에 발생한 문제

- - -

## 환경

```kotlin
dependencies {  
    implementation("org.springframework.boot:spring-boot-starter-webflux")  
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")  
    implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")  
    implementation("org.jetbrains.kotlin:kotlin-reflect")  
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")  
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-slf4j")  
    testImplementation("org.springframework.boot:spring-boot-starter-test")  
    testImplementation("io.projectreactor:reactor-test")  
    implementation("org.springframework.boot:spring-boot-starter-log4j2")  
}  
  
configurations {  
    all {  
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")  
        exclude(group = "org.slf4j", module = "slf4j-reload4j")  
    }  
}
```

- - -

## 문제

#### 의도
- "traceId"는 각 요청마다 임의의 값이 부여된다.
- "traceId"는 MDC에 저장된다.
- MDC를 Coroutine과 함께 사용하기 위해서 kotlin에서 제공하는 *MDCContext* 이용
- coroutine이 suspend 되어서 다른 thread에서 수행되어도 문제 없이 mdc에서 값을 가져올 수 있어야 한다
- 여러 요청이 동시에 들어와도 "tracedId"가 유실되거나 혼용되는 경우가 없어야 한다

### 코드

```kotlin
@Configuration  
class SampleRouter(  
    private val sampleHandler: SampleHandler,  
    private val sampleFilter: SampleFilter,  
) {  
    @Bean  
    fun sampleRouterFunction() = coRouter {  
        (ROOT and accept(MediaType.APPLICATION_JSON)).nest {  
            GET("", sampleHandler::doGet)  
        }  
        filter(sampleFilter)  
    }  
  
    companion object {  
        private val ROOT = "/a"  
    }  
}  
  
@Service  
class SampleHandler {  
    suspend fun doGet(req: ServerRequest): ServerResponse {  
        logger.info("Handler")  
        return ServerResponse.ok().bodyValueAndAwait("RES")  
    }  
  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}  
  
@Service  
class SampleFilter: suspend (ServerRequest, suspend (ServerRequest) -> ServerResponse) -> ServerResponse {  
    override suspend fun invoke(req: ServerRequest, handler: suspend (ServerRequest) -> ServerResponse): ServerResponse {  
        MDC.put("traceId", UUID.randomUUID().toString())  
        return withContext(MDCContext()){  
            logger.info("[REQ]")  
            handler(req)  
                .also { logger.info("[RES]") }  
        }    }  
  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}
```

### 결과

```
2023-05-01 15:28:15.878       INFO      868e2073-b341-4898-a5cc-d277e64c41fa     17654 [ctor-http-nio-2] c.e.d.SampleFilter$invoke$2              : [REQ]
2023-05-01 15:28:15.887       INFO      868e2073-b341-4898-a5cc-d277e64c41fa     17654 [ctor-http-nio-2] c.e.d.SampleHandler                      : Handler
2023-05-01 15:28:15.892       INFO      868e2073-b341-4898-a5cc-d277e64c41fa     17654 [ctor-http-nio-2] c.e.d.SampleFilter$invoke$2              : [RES]
```

### 문제가 되는 상황

- 문제가 되는 상황은 *delay* 같이 coroutine을 suspend하는 경우에 발생한다. DB에 접근하는 경우에도 발생
- *delay* 이후에 MDC 값을 제대로 가져오지 못하는 문제 발생

```kotlin
@Service  
class SampleHandler {  
    suspend fun doGet(req: ServerRequest): ServerResponse {  
        logger.info("Start Handler")  
        delay(1000)  
        logger.info("After delay")  
        return ServerResponse.ok().bodyValueAndAwait("RES")  
    }  
  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}
```

```
2023-05-01 15:31:09.973       INFO      566db61d-6ec2-4fcb-a5b4-5d9e5bf50081     26707 [ctor-http-nio-2] c.e.d.SampleFilter$invoke$2              : [REQ]
2023-05-01 15:31:09.983       INFO      566db61d-6ec2-4fcb-a5b4-5d9e5bf50081     26707 [ctor-http-nio-2] c.e.d.SampleHandler                      : Start Handler
2023-05-01 15:31:10.994       INFO                   26707 [DefaultExecutor] c.e.d.SampleHandler                      : After delay
2023-05-01 15:31:11.022       INFO      566db61d-6ec2-4fcb-a5b4-5d9e5bf50081     26707 [DefaultExecutor] c.e.d.SampleFilter$invoke$2              : [RES]
```

### 문제의 이유(예상)

- *Corouter*에서 handler를 등록할 때 *Dispathers.Unconfined*라는 coroutine context에서 등록.
- Dispatchers.Unconfied이기 때문에 첫 suspension까지만 해당 coroutine을 호출 스레드에서 동작하고 나머지는 다른 thread에서 수행한다.
- MDCContext는 Thread가 바뀔때마다 Context를 가져오도록 되어 있다. 그러나 *mono*를 통해서 만들때 MDCContext가 아니라 *Unconfined*이다. 결국 첫 suspension을 하게 되면 첫 thread(MDC가 저장되어 있는)에서 벗어나게 되면서 MDC 값이 없는 thread에서 동작하는 것이다.  

```kotlin
public class MDCContext(  
    /**  
     * The value of [MDC] context map.     */    @Suppress("MemberVisibilityCanBePrivate")  
    public val contextMap: MDCContextMap = MDC.getCopyOfContextMap()  
) : ThreadContextElement<MDCContextMap>, AbstractCoroutineContextElement(Key) {  
    /**  
     * Key of [MDCContext] in [CoroutineContext].  
     */    public companion object Key : CoroutineContext.Key<MDCContext>  
  
    /** @suppress */  
    override fun updateThreadContext(context: CoroutineContext): MDCContextMap {  
        val oldState = MDC.getCopyOfContextMap()  
        setCurrent(contextMap)  
        return oldState  
    }  
  
    /** @suppress */  
    override fun restoreThreadContext(context: CoroutineContext, oldState: MDCContextMap) {  
        setCurrent(oldState)  
    }  
  
    private fun setCurrent(contextMap: MDCContextMap) {  
        if (contextMap == null) {  
            MDC.clear()  
        } else {  
            MDC.setContextMap(contextMap)  
        }  
    }  
}

// CoroutineDSL
fun GET(pattern: String, f: suspend (ServerRequest) -> ServerResponse) {  
   builder.GET(pattern, asHandlerFunction(f))  
}

private fun asHandlerFunction(init: suspend (ServerRequest) -> ServerResponse) = HandlerFunction {  
   mono(Dispatchers.Unconfined) {  
      init(it)  
   }  
}

public fun <T> mono(  
    context: CoroutineContext = EmptyCoroutineContext,  
    block: suspend CoroutineScope.() -> T?  
): Mono<T> {  
    require(context[Job] === null) { "Mono context cannot contain job in it." +  
            "Its lifecycle should be managed via Disposable handle. Had $context" }  
    return monoInternal(GlobalScope, context, block)  
}

private fun <T> monoInternal(  
    scope: CoroutineScope, // support for legacy mono in scope  
    context: CoroutineContext,  
    block: suspend CoroutineScope.() -> T?  
): Mono<T> = Mono.create { sink ->  
    val reactorContext = context.extendReactorContext(sink.currentContext())  
    val newContext = scope.newCoroutineContext(context + reactorContext)  
    val coroutine = MonoCoroutine(newContext, sink)  
    sink.onDispose(coroutine)  
    coroutine.start(CoroutineStart.DEFAULT, coroutine, block)  
}
```

- - -

## 해결 방법

- [힌트를 제공하는 링크](https://github.com/Kotlin/kotlinx.coroutines/issues/985#issuecomment-535075092)
- Corouter가 핸들러를 등록할 때 MDCContext를 추가해준다. 
- 첫번째 suspension하기 전에 MDCContext -> Unconfined -> MDCContext로 context가 바뀌고 최종적으로 MDCContext에서 동작하기 때문에 suspension해서 다른 thread에서 동작한다 해도 MDC 값을 제대로 복구할 수 있다.

```kotlin
@Configuration  
class SampleRouter(  
    private val sampleHandler: SampleHandler,  
    private val sampleFilter: SampleFilter,  
) {  
    @Bean  
    fun sampleRouterFunction() = coRouter {  
        (ROOT and accept(MediaType.APPLICATION_JSON)).nest {  
            Get("", sampleHandler::doGet)  
        }  
        filter(sampleFilter)  
    }  
  
    companion object {  
        private val ROOT = "/a"  
    }  
    fun CoRouterFunctionDsl.Get(pattern: String, f: suspend (ServerRequest) -> ServerResponse) {  
        this.GET(pattern) { withContext(MDCContext()){ f(it) } }  
    }  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}  
  
@Service  
class SampleHandler {  
    suspend fun doGet(req: ServerRequest): ServerResponse {  
        logger.info("Start Handler")  
        delay(1000)  
        logger.info("After delay")  
        return ServerResponse.ok().bodyValueAndAwait("RES")  
    }  
  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}  
  
@Service  
class SampleFilter: suspend (ServerRequest, suspend (ServerRequest) -> ServerResponse) -> ServerResponse {  
    override suspend fun invoke(req: ServerRequest, handler: suspend (ServerRequest) -> ServerResponse): ServerResponse {  
        MDC.put("traceId", UUID.randomUUID().toString())  
        return withContext(MDCContext()){  
            logger.info("[REQ]")  
            handler(req)  
                .also { logger.info("[RES]") }  
        }    }  
  
    private val logger by lazy {  
        LoggerFactory.getLogger(this::class.java)  
    }  
}
```

```
2023-05-01 15:40:22.082       INFO      f7e73ee4-acf4-42a7-870c-99c49893937d     62100 [ctor-http-nio-2] c.e.d.SampleFilter$invoke$2              : [REQ]
2023-05-01 15:40:22.096       INFO      f7e73ee4-acf4-42a7-870c-99c49893937d     62100 [ctor-http-nio-2] c.e.d.SampleHandler                      : Start Handler
2023-05-01 15:40:23.109       INFO      f7e73ee4-acf4-42a7-870c-99c49893937d     62100 [DefaultExecutor] c.e.d.SampleHandler                      : After delay
2023-05-01 15:40:23.126       INFO      f7e73ee4-acf4-42a7-870c-99c49893937d     62100 [DefaultExecutor] c.e.d.SampleFilter$invoke$2              : [RES]
```

- - -

## Coroutine Context

- CoroutineContext: 코루틴 컨텍스트는 여러 요소들을 포함한다. Coroutine은 Coroutine context들의 값을 바탕으로 동작한다.
- CoroutineContext는 CoroutineDispather를 포함한다
	- CoroutineDispatcher: 코루틴 실행을 어떤 thread에서 진행할지등을 결정. 특정 thread, thread pool등에서 코루틴이 실행되도록 제한하는 기능을 한다(혹은 제한을 없애기도 함)
 - 모든 coroutine builder(async, launch) 등은 CoroutineContext 타입의 객체를 선택적으로 인자로 받는다. 해당 매개변수는 빌더를 통해서 만들어지는 coroutine에 CoroutineDispatcher를 제공한다
- 몇 가지 CoroutineContext 들
	- Dispatchers.Unconfined: main thread에서 동작하게 한다
	- Dispatchers.Default
	- newSingleThreadContext: 코루틴을 수행할 새로운 스레드 생성

- Dispathers.Unconfined는 사실은 조금 더 복잡하다. 첫 suspension까지는 호출 스레드에서 수행한다. suspension 이후에는 다른 스레드에서 코루틴을 수행. 
- GlobalScope는 Dispatchers.Default에서 수행.

- 코루틴이 내부에서 다른 코루틴을 시작하면 자식 코루틴은 부모 코루틴의 Context를 물려 받는다.

### withContext

- [내용]([withContext (kotlinlang.org)](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html))
- 현재 coroutine을 suspension하고 withContext 블록을 수행한다.
- Coroutine Context는 현재의 CoroutineContext + 제공된 Context이다.

- - -

## 참고 자료

- https://github.com/Kotlin/kotlinx.coroutines/issues/985#issuecomment-535075092
- [배민광고리스팅 개발기 (feat. 코프링과 DSL 그리고 코루틴) | 우아한형제들 기술블로그 (woowahan.com)](https://techblog.woowahan.com/7349/)
- [Java 로깅전략 with MDC - JH Blog (jehuipark.github.io)](https://jehuipark.github.io/java/java-logging-mdc)
- [Coroutine context and dispatchers | Kotlin Documentation (kotlinlang.org)](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
- [Spring WebFlux 에서 coRouter filter를 이용하여 request, response 로깅하기 | by Riiid Teamblog | Riiid Teamblog KR | Medium](https://medium.com/riiid-teamblog-kr/spring-webflux-%EC%97%90%EC%84%9C-corouter-filter%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-request-response-%EB%A1%9C%EA%B9%85%ED%95%98%EA%B8%B0-df56f9d9680)
- 
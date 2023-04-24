---
title: Web
tags: [java, spring, web]
sidebar_label: Web
sidebar_position: 1
---

# 网络处理相关注解

```java
@Slf4j
@RestController
@RequestMapping("goods/v1")
@AllArgsConstructor
public class GoodsController {
    private final GoodsService goodsService;

    /**
     * `curl https://example.com/goods/v1/3567898/info?source=search&input=iphone13`
     */
    @GetMapping("{goods_id}/info")
    public GoodsInfo goods(@PathVariable("goods_id") Long goodsId,
                           @RequestParam("viewSource") String viewSource, @RequestParam("userInput") String userInput) {
        if (StringUtils.isNotBlank(viewSource)) {
            log.info("get goods from {} {}", viewSource, userInput);
        }
        return goodsService.getGoodsInfo(goodsId);
    }

    /**
     * `curl -X POST -d '{"goods_name":"xxxx","goodsPrice":1000}' https://example.com/goods/v1/3567898/info`
     */
    @PostMapping("{goods_id}/info")
    public BaseResponse<Void> updateGoods(@PathVariable("goods_id") Long goodsId, @RequestBody GoodsInfo goodsInfo) {
        goodsService.updateGoods(goodsInfo);
        return BaseResponse.<Void>success().build();
    }

    /**
     * `curl https://example.com/goods/v1/list?page=0&size=20`
     */
    @GetMapping(value = "list")
    public BaseResponse<PageResponse<GoodsInfo>> listGoods(@RequestParam("page") Integer pageNo, @RequestParam("size") Integer pageSize) {
        PageResponse<GoodsInfo> response = goodsService.listGoods(pageNo, pageSize);
        return BaseResponse.<PageResponse<GoodsInfo>>success(response).build();
    }

    /**
     * `curl -X POST -d '<xml><goodsId>1234456</goodsId><goodsName>xxxxxx</goodsName></xml>' https://example.com/goods/v1/3567898/info`
     */
    @PostMapping(value = "{goods_id}/edit", params = "goodsId", consumes = MediaType.TEXT_XML_VALUE, produces = MediaType.TEXT_XML_VALUE)
    public BaseResponse<Void> editGoods(@RequestBody EditGoodsRequest request) {
        goodsService.edit(request);
        return BaseResponse.<Void>success().build();
    }
}
```

## @Controller 和 @RestController

被注解的类表示是一个web controller，可以接收和处理HTTP请求。

`@RestController`注解其实是在`@Controller`的基础上，添加了`@ResponseBody`注解。在用法上，两者的区别如下：

* `@RestController`无法返回指定页面，而`@Controller`可以。
* `@Controller`中的方法要返回JSON、XML等格式的数据，需要补充`@ResponseBody`注解，而`@RestController`则不需要。

## @RequestMapping 和 @XxxMapping

`@RequestMapping` 注解用于将网络请求的地址映射到控制器`Controller`的方法处理器上。`@RequestMapping`只能注解到类和方法上，类注解用于定义控制器中所有方法的公共属性。例如，上面的`goods/v1`和`/{goods_id}`共同组成了URI，`goods/v1`是公共前缀。

* name：名称。
* value：网络请求的URI构成部分，例如`goods/{goods_id}/details`，可设置多个。
* path：同value功能。
* method：HTTP请求方法，GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE。
* params：指定请求中参数的要求，满足要求的才能使用该方法处理。所以，可以多个方法配置了同样的URI，但是有不同的参数处理方式。
* header：指定请求中的header的要求，满足要求的才能使用该方法处理。
* consumes：指定请求中的`Content-Type`的`MediaType`必须满足的要求。
* produces：指定返回头中的`Accept`的`MediaType`必须满足的要求。

除此之外，Spring提供了 `@GetMapping` , `@PostMapping` ， `@PutMapping` , `@DeleteMapping` , `@PatchMapping` 等注解用于简化 `@RequestMapping` 的配置，区别点只是在于指定了HTTP请求的方法。

## @RequestVariable 和 @RequestParam

`@RequestVariable` 用于把方法中的参数和HTTP请求中的URI路径子串进行绑定。

`@RequestParam` 用于把方法中的参数和HTTP请求中的URI参数进行绑定。

## @RequestBody 和 @ResponseBody

`@RequestBody` 注解用于把方法参数和HTTP请求体的数据进行绑定，并依赖 `HttpMessageConverter` 接口实现对数据进行转换。一般来说，可以和 `@Valid` 注解一同使用，来校验请求类的数据。

`@ResponseBody` 用于类或方法，注解表示方法返回值是一个网络请求的Body。

### @RequestHeader

`@RequestHeader` 用于方法参数和HTTP的Header绑定。

```java
@RequestMapping(value="/requestHeaderTest")
public void requestHeaderTest(
    @RequestHeader("User-Agent") String userAgent,
    @RequestHeader(value="Accept", required=false) String[] accepts) {
    ...
}
```

`@CookieValue` 用于方法参数和HTTP中Header的Cookie进行绑定。

```java
@RequestMapping(value="/requestCookieTest")
public void requestHeaderTest(
    @CookieValue("xxx_token") String xxxToken,
    @CookieValue(value="name", required=false) String name) {
    ...
}
```

## @ControllerAdvice

对声明 `@ExceptionHandler` 、`@InitBinder` 或 `@ModelAttribute` 方法的类进行 `@Component` 的特化，以在多个 `@Controller` 类之间共享。

`@ControllerAdvice` ，是一个非常有用的注解，增强了 `Controller` ，使用这个注解，可以实现三个方面的功能：

1. 全局异常处理：结合方法型注解 `@ExceptionHandler` ，用于捕获 `@Controller` 中抛出的指定类型的异常，从而达到不同类型的异常区别处理的目的。
2. 全局数据绑定：结合方法型注解 `@InitBinder` ，用于 `request` 中自定义参数解析方式进行注册，从而达到自定义指定格式参数的目的。
3. 全局数据预处理：结合方法型注解 `@ModelAttribute` ，表示其注解的方法将会在目标 `@Controller` 方法执行之前执行。

```java
@Slf4j
@ControllerAdvice
public class ApiControllerAdvice {
    private HttpServletRequest request;

    @Autowired
    public void setRequest(HttpServletRequest request) {
        this.request = request;
    }

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("time", Instant.now());
    }

    @ExceptionHandler(Throwable.class)
    @ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR)
    @ResponseBody
    public BaseResponse<?> handleAny(Throwable th) {
        log.error(th.getMessage(), th);
        return BaseResponse.newFailResponse()
                .errorCode(ErrorCode.SYSTEM_EXCEPTION.getCode())
                .errorMsg("System Error")
                .build();
    }

    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(value = HttpStatus.INTERNAL_SERVER_ERROR)
    @ResponseBody
    public BaseResponse<?> handleRuntimeException(RuntimeException ex) {
        log.error(ex.getMessage(), ex);
        return BaseResponse.newFailResponse()
                .errorCode(ErrorCode.SYSTEM_EXCEPTION.getCode())
                .errorMsg("System Error")
                .build();
    }

    @ExceptionHandler(HttpMessageNotReadableException.class)
    @ResponseStatus(value = HttpStatus.OK)
    @ResponseBody
    public BaseResponse<?> handleNotReadable(HttpMessageNotReadableException hre) {
        log.warn("request not readable, {}", hre.getMessage(), hre);
        return BaseResponse.newFailResponse()
                .errorCode(ErrorCode.BAD_PARAMS.getCode())
                .errorMsg("bad request params or body")
                .build();
    }
}
```

# 参考资料
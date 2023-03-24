
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

### @Controller 和 @RestController

被注解的类表示是一个web controller，可以接收和处理HTTP请求。

`@RestController`注解其实是在`@Controller`的基础上，添加了`@ResponseBody`注解。在用法上，两者的区别如下：

* `@RestController`无法返回指定页面，而`@Controller`可以。
* `@Controller`中的方法要返回JSON、XML等格式的数据，需要补充`@ResponseBody`注解，而`@RestController`则不需要。

### @RequestMapping 和 @XxxMapping

`@RequestMapping`注解用于将网络请求的地址映射到控制器`Controller`的方法处理器上。`@RequestMapping`只能注解到类和方法上，类注解用于定义控制器中所有方法的公共属性。例如，上面的`goods/v1`和`/{goods_id}`共同组成了URI，`goods/v1`是公共前缀。

* name：名称。
* value：网络请求的URI构成部分，例如`goods/{goods_id}/details`，可设置多个。
* path：同value功能。
* method：HTTP请求方法，GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE。
* params：指定请求中参数的要求，满足要求的才能使用该方法处理。所以，可以多个方法配置了同样的URI，但是有不同的参数处理方式。
* header：指定请求中的header的要求，满足要求的才能使用该方法处理。
* consumes：指定请求中的`Content-Type`的`MediaType`必须满足的要求。
* produces：指定返回头中的`Accept`的`MediaType`必须满足的要求。



@GetMapping

@PostMapping

@RequestParam

@RequestVariable

### @RequestHeader

用于方法参数和HTTP的Header绑定。

```java
@RequestMapping(value="/requestHeaderTest")
public void requestHeaderTest(
    @RequestHeader("User-Agent") String userAgent,
    @RequestHeader(value="Accept", required=false) String[] accepts) {
    ...
}
```

@CookieValue
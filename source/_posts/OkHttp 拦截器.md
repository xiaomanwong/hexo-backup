# OkHttp Interceptor



OKHttp 在开发中经常用到,这里介绍一下 OKHttp 的拦截器(interceptor)的几个使用例子



要是用拦截器很简单,我们只需要自定义一个 interceptor 类, 并实现 Interceptor 接口, 同时重写 intercept 方法.



这里介绍两种 Interceptor 的实例



## Log 日志 (LogInterceptor)

Log 在开发调试中的重要性, 不言而喻, 一个好的日志数据,能够为开发时提供很多帮助,可以通过 Log 来获取当前程序的执行状态, 顺序等等.



这里介绍的 Log 日志拦截器,主要是负责在网络请求时,截获请求中的信息,并将请求信息以 Log 的方式输出到控制台上,可展现当前请求地址,请求参数,以及请求结果等.



话不多说, 上代码.



```java
/**
* 自定义 log 拦截器,输入请求地址,请求参,请求结果
*/
public class LogInterceptor implements Interceptor {

        @Override
        public Response intercept(Chain chain) throws IOException {
            // 拦截请求信息,获取请求地址及请求参数
            Request request = chain.request();

            long t1 = System.nanoTime();
            // 获取请求方式
            String method = request.method();
            if ("POST".equals(method)) {
                StringBuilder sb = new StringBuilder();
                if (request.body() instanceof FormBody) {
                    FormBody body = (FormBody) request.body();
                    for (int i = 0; i < body.size(); i++) {
                        sb.append(body.encodedName(i) + "=" + body.encodedValue(i) + ",");
                    }
                    sb.delete(sb.length() - 1, sb.length());
                    LogUtils.d(TAG, String.format("Sending request %s on %s %n%s %nRequestParams:{%s}",
                            request.url(), chain.connection(), request.headers(), sb.toString()));
                }
            } else {
                LogUtils.d(TAG, String.format("Sending request %s on %s %n%s",
                        request.url(), chain.connection(), request.headers()));
            }
			// 获取响应信息
            Response response = chain.proceed(request);
            long t2 = System.nanoTime();
            LogUtils.d(TAG, String.format("Received response for %s in %.1fms%n%s",
                    response.request().url(), (t2 - t1) / 1e6d, response.headers()));

            MediaType contentType = response.body().contentType();
            String content = response.body().string();
            LogUtils.d(TAG, content);
            ResponseBody wrappedBody = ResponseBody.create(contentType, content);
            return response.newBuilder().body(wrappedBody).build();
        }
    }
```





## 加参



加参的意义在于,在请求过程中, 需要传递一些基本参数,这些参数是基本保持不变的, 而这些参数又是后台需要校验的依靠,我们俗称他们为**公参**, 如果在请求过程中在每一个接口里面都去添加这些参数,显得非常繁琐,而且麻烦,后续人员接手,又会不知所措,维护起来相当不方便.



因此,在这里我们还是通过请求拦截器,通过拦截器去添加这些参数.

### 公参拦截器

在请求中,增加一些公共的参数.

```java
public class CommonParamsInterceptor implements Interceptor {

    private Map<String, String> queryParamsMap = new HashMap<>();
    private Map<String, String> paramsMap = new HashMap<>();
    private Map<String, String> headerParamsMap = new HashMap<>();
    private List<String> headerLinesList = new ArrayList<>();

    @Override
    public Response intercept(Chain chain) throws IOException {

        Request request = chain.request();
        Request.Builder requestBuilder = request.newBuilder();

        // process header params inject
        Headers.Builder headerBuilder = request.headers().newBuilder();
        if (headerParamsMap.size() > 0) {
            Iterator iterator = headerParamsMap.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry entry = (Map.Entry) iterator.next();
                headerBuilder.add((String) entry.getKey(), (String) entry.getValue());
            }
        }

        if (headerLinesList.size() > 0) {
            for (String line : headerLinesList) {
                headerBuilder.add(line);
            }
            requestBuilder.headers(headerBuilder.build());
        }
        // process header params end


        // process queryParams inject whatever it's GET or POST
        if (queryParamsMap.size() > 0 && "GET".equals(request.method())) {
//        if (queryParamsMap.size() > 0) {
            request = injectParamsIntoUrl(request, requestBuilder, queryParamsMap);
        }

        // process post body inject
        if (paramsMap != null && paramsMap.size() > 0 && "POST".equals(request.method())) {
            if (request.body() instanceof FormBody) {
                FormBody.Builder newFormBodyBuilder = new FormBody.Builder();
                if (paramsMap.size() > 0) {
                    Iterator iterator = paramsMap.entrySet().iterator();
                    while (iterator.hasNext()) {
                        Map.Entry entry = (Map.Entry) iterator.next();
                        newFormBodyBuilder.add((String) entry.getKey(), (String) entry.getValue());
                    }
                }

                FormBody oldFormBody = (FormBody) request.body();
                int paramSize = oldFormBody.size();
                if (paramSize > 0) {
                    for (int i = 0; i < paramSize; i++) {
                        newFormBodyBuilder.add(oldFormBody.name(i), oldFormBody.value(i));
                    }
                }

                requestBuilder.post(newFormBodyBuilder.build());
                request = requestBuilder.build();
            } else if (request.body() instanceof MultipartBody) {
                MultipartBody.Builder multipartBuilder = new MultipartBody.Builder().setType(MultipartBody.FORM);

                Iterator iterator = paramsMap.entrySet().iterator();
                while (iterator.hasNext()) {
                    Map.Entry entry = (Map.Entry) iterator.next();
                    multipartBuilder.addFormDataPart((String) entry.getKey(), (String) entry.getValue());
                }

                List<MultipartBody.Part> oldParts = ((MultipartBody) request.body()).parts();
                if (oldParts != null && oldParts.size() > 0) {
                    for (MultipartBody.Part part : oldParts) {
                        multipartBuilder.addPart(part);
                    }
                }

                requestBuilder.post(multipartBuilder.build());
                request = requestBuilder.build();
            }

        }
        return chain.proceed(request);
    }

    private boolean canInjectIntoBody(Request request) {
        if (request == null) {
            return false;
        }
        if (!TextUtils.equals(request.method(), "POST")) {
            return false;
        }
        RequestBody body = request.body();
        if (body == null) {
            return false;
        }
        MediaType mediaType = body.contentType();
        if (mediaType == null) {
            return false;
        }
        if (!TextUtils.equals(mediaType.subtype(), "x-www-form-urlencoded")) {
            return false;
        }
        return true;
    }

    // func to inject params into url
    private Request injectParamsIntoUrl(Request request, Request.Builder requestBuilder, Map<String, String> paramsMap) {
        HttpUrl.Builder httpUrlBuilder = request.url().newBuilder();
        if (paramsMap.size() > 0) {
            Iterator iterator = paramsMap.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry entry = (Map.Entry) iterator.next();
                httpUrlBuilder.addEncodedQueryParameter((String) entry.getKey(), (String) entry.getValue());
            }
            requestBuilder.url(httpUrlBuilder.build());
            return requestBuilder.build();
        }

        return request;
    }

    private static String bodyToString(final RequestBody request) {
        try {
            final RequestBody copy = request;
            final Buffer buffer = new Buffer();
            if (copy != null)
                copy.writeTo(buffer);
            else
                return "";
            return buffer.readUtf8();
        } catch (final IOException e) {
            return "did not work";
        }
    }

    public static class Builder {

        CommonParamsInterceptor interceptor;

        public Builder() {
            interceptor = new CommonParamsInterceptor();
        }

        public Builder addParam(String key, String value) {
            interceptor.paramsMap.put(key, value);
            return this;
        }

        public Builder addParamsMap(Map<String, String> paramsMap) {
            interceptor.paramsMap.putAll(paramsMap);
            return this;
        }

        public Builder addHeaderParam(String key, String value) {
            interceptor.headerParamsMap.put(key, value);
            return this;
        }

        public Builder addHeaderParamsMap(Map<String, String> headerParamsMap) {
            interceptor.headerParamsMap.putAll(headerParamsMap);
            return this;
        }

        public Builder addHeaderLine(String headerLine) {
            int index = headerLine.indexOf(":");
            if (index == -1) {
                throw new IllegalArgumentException("Unexpected header: " + headerLine);
            }
            interceptor.headerLinesList.add(headerLine);
            return this;
        }

        public Builder addHeaderLinesList(List<String> headerLinesList) {
            for (String headerLine : headerLinesList) {
                int index = headerLine.indexOf(":");
                if (index == -1) {
                    throw new IllegalArgumentException("Unexpected header: " + headerLine);
                }
                interceptor.headerLinesList.add(headerLine);
            }
            return this;
        }

        public Builder addQueryParam(String key, String value) {
            interceptor.queryParamsMap.put(key, value);
            return this;
        }

        public Builder addQueryParamsMap(Map<String, String> queryParamsMap) {
            interceptor.queryParamsMap.putAll(queryParamsMap);
            return this;
        }

        public CommonParamsInterceptor build() {
            return interceptor;
        }
    }
}
```



### 加签拦截器

互联网是一个开放的环境,危险无处不在,加密通信是安全的基础.

加密的方式又有很多,比如对称加密/非对称加密/Hash(严格的说不是加密),这里先不对加密进行介绍,我们聊聊加签;

加签,其实就是给报文做一个摘要,相同的签名算法得到的摘要是相同的,比如MD5, SH1, SH256等, 简单的加签并不能防止篡改,因为攻击者可以篡改后,自己生成新的签名.服务端验签还是可以通过的,因此加签时一定要包含一些私有的东西,比如私钥.



这里介绍一种加签方式,

加密规则

1.  根据请求参数 key 进行排序
2.  按排好的顺序组装成 key=value&key=value 形式的字符串
3.  将上述字符串拼接 mid, timestamp, key(私钥) ,最终形成 key=value&key=value&mid&timestamp&key 的字符串
4.  将字符串 md5 32位小写加密, 生成 auth.

一个简单的加签逻辑就是这样,下面代码就是对这种规则的实现,看代码↓↓↓↓

```java
public class AuthorizeInterceptor implements Interceptor {

    private static final String TAG = "AuthorizeInterceptor";

    /**
     * 生成 auth 的私钥
     */
    private String authKey;

    /**
     * 当前设备 mid,用来生成 auth
     * 数据由 Builder 类传入
     */
    private String mid = "";

    private AuthorizeInterceptor() {
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        Request.Builder requestBuilder = request.newBuilder();
        String url = request.url().toString();
        LogUtils.d(TAG, url);
        if ("POST".equals(request.method())) {
            request = injectionParamIntoBody(request, requestBuilder);
        } else if ("GET".equals(request.method())) {
            request = injectionAuthIntoUrl(request, requestBuilder);
        }
        return chain.proceed(request);
    }

    /**
     * GET 请求方式, 生成授权和添加时间戳
     *
     * @param request
     * @param requestBuilder
     * @return
     */
    private Request injectionAuthIntoUrl(Request request, Request.Builder requestBuilder) {
        //获取到请求地址api
        HttpUrl newHttpUrl = request.url();
        TreeMap<String, String> authMap = new TreeMap<>();
        //通过请求地址(最初始的请求地址)获取到参数列表
        Set<String> parameterNames = newHttpUrl.queryParameterNames();
        long timestamp = System.currentTimeMillis();
        for (String key : parameterNames) {
            //循环参数列表
            if (!"basic".equals(key)) {
                // 获取参数value,
                String paramValue = newHttpUrl.queryParameter(key);
                if (!TextUtils.isEmpty(paramValue)) {
                    authMap.put(key, paramValue);
                }
            }
        }
        LogUtils.d(TAG, "===================================== Get 开始生成 auth ==========================================");
        HttpUrl.Builder newBuilder =
                request.url().newBuilder()
                        .addEncodedQueryParameter("auth", assembleAuth(authMap, timestamp, false))
                        .addEncodedQueryParameter("timestamp", String.valueOf(timestamp));
        requestBuilder.url(newBuilder.build());
        return requestBuilder.build();
    }

    /**
     * 将参数注入到 POST 请求的 body 中
     *
     * @param request
     * @param requestBuilder
     * @return
     */
    private Request injectionParamIntoBody(Request request, Request.Builder requestBuilder) {
        if (request.body() instanceof FormBody) {
            // 处理正常表单请求方式
            FormBody oldFormBody = (FormBody) request.body();
            return assembleFormBody(request, oldFormBody, requestBuilder);
        } else if (request.body() instanceof MultipartBody) {
            return assembleMultipartBody(request, requestBuilder);
        } else {
            return request;
        }
    }

    /**
     * 构建流媒体参数的 body
     *
     * @param request
     * @param builder
     * @return
     */
    private Request assembleMultipartBody(Request request, Request.Builder builder) {
        // 处理流的请求方式
        MultipartBody body = (MultipartBody) request.body();
        TreeMap<String, String> authMap = new TreeMap<>();
        long timestamp = System.currentTimeMillis();
        // 获取参数 key 及 value 数据, 将数据写入到 TreeMap 中进行排序
        if (body != null && body.parts().size() > 0) {
            for (MultipartBody.Part part : body.parts()) {
                // 只处理 contentType 为空,或为 text 的情况, image 上传图片的字段,不进行处理
                if (part.body().contentType() == null
                        || !"image".equals(part.body().contentType().type())
                        || "text".equals(part.body().contentType().type())) {
                    Headers headers = part.headers();
                    // 从 header 中获取 Key, form-data; name= 的字段过滤获取key
                    // 筛选掉 basic, 获取到当前 header,对应的 RequestBody, 通过 buffer 的方式,获取到 value
                    // 将 key 和 value 填充到 TreeMap 中
                    for (int i = 0; i < headers.names().size(); i++) {
                        String headerName = headers.value(i);
                        if (headerName.contains("form-data; name=")) {
                            String key = headerName.replace("form-data; name=", "").replace("\"", "");
                            if (!"basic".equals(key)) {
                                String value = body2String(part.body());
                                if (!TextUtils.isEmpty(value)) {
                                    authMap.put(key, body2String(part.body()));
                                }
                            }
                        }
                    }
                }
            }
        }

        LogUtils.d(TAG, "============================ MultipartBody 准备生成 Auth =====================================");
        MultipartBody.Builder multipartBuilder = new MultipartBody.Builder().setType(MultipartBody.FORM);
        multipartBuilder.addFormDataPart("auth", assembleAuth(authMap, timestamp, false));
        multipartBuilder.addFormDataPart("timestamp", String.valueOf(timestamp));
        List<MultipartBody.Part> oldParts = ((MultipartBody) request.body()).parts();
        if (oldParts != null && oldParts.size() > 0) {
            for (MultipartBody.Part part : oldParts) {
                multipartBuilder.addPart(part);
            }
        }
        builder.post(multipartBuilder.build());
        return builder.build();
    }

    /**
     * 构建 Auth 数据
     * <p>
     * TreeMap 数据,按照字母顺序自动排序后,遍历数据,拼接成 key=value&key=value 的形式,
     * Map 拼接完成后,在其后需要再次拼接上 &mid&timestamp&key
     * <p>
     * 最后将数据 MD5 转化为 32 位小写并返回
     *
     * @param treeMap      构建 Auth 的具体数据
     * @param timestamp    当前时间戳
     * @param isNeedDecode 是否需要解码
     * @return 返回 auth 数据 --- 32位小写
     */
    private String assembleAuth(TreeMap<String, String> treeMap, long timestamp, boolean isNeedDecode) {
        StringBuilder stringBuilder = new StringBuilder();
        if (treeMap.size() > 0) {
            for (Map.Entry<String, String> stringStringEntry : treeMap.entrySet()) {
                try {
                    stringBuilder.append(stringStringEntry.getKey())
                            .append("=")
                            .append(isNeedDecode
                                    ? URLDecoder.decode(stringStringEntry.getValue(), "UTF-8")
                                    : stringStringEntry.getValue())
                            .append("&");
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        }
        treeMap.clear();
        stringBuilder.append(mid)
                .append("&")
                .append(timestamp)
                .append("&")
                .append(authKey);
        LogUtils.d(TAG, stringBuilder.toString());
        LogUtils.d(TAG, "============================ Auth 生成完成 =====================================");
        return MD5.MD5_32(stringBuilder.toString());
    }

    /**
     * body 转 string 获取实际参数值
     *
     * @param body Part body
     * @return part 中写如的参数数据
     */
    private String body2String(RequestBody body) {
        if (body != null) {
            Buffer buffer = new Buffer();
            try {
                body.writeTo(buffer);
                Charset charset = Charset.forName("UTF-8");
                MediaType contentType = body.contentType();
                if (contentType != null) {
                    charset = contentType.charset(charset);
                }
                assert charset != null;
                return buffer.readString(charset);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return "";
    }

    /**
     * 组装 Form 表单请求数据
     *
     * @param request
     * @param oldFormBody
     * @param requestBuilder
     * @return
     */
    private Request assembleFormBody(Request request, FormBody oldFormBody, Request.Builder requestBuilder) {
        TreeMap<String, String> authMap = new TreeMap<>();
        // 遍历请求参数,非空参数,添加到集合中
        for (int i = 0; i < oldFormBody.size(); i++) {
            if (!"basic".equals(oldFormBody.encodedName(i)) && !TextUtils.isEmpty(oldFormBody.encodedValue(i))) {
                authMap.put(oldFormBody.encodedName(i), oldFormBody.encodedValue(i));
            }
        }
        long timestamp = System.currentTimeMillis();
        if (authMap.size() != 0) {
            LogUtils.d(TAG, "============================ FormBody 准备生成 auth ===========================");
            // 生成 auth 数据
            FormBody.Builder newFormBody = new FormBody.Builder();
            for (int i = 0; i < oldFormBody.size(); i++) {
                newFormBody.addEncoded(oldFormBody.encodedName(i), oldFormBody.encodedValue(i));
            }
            newFormBody.add("auth", assembleAuth(authMap, timestamp, true));
            newFormBody.add("timestamp", String.valueOf(timestamp));
            requestBuilder.method(request.method(), newFormBody.build());
            return requestBuilder.build();
        }

        return request;
    }


    /**
     * Builder 构造函数,用来设置 Authorize 的相关参数,及创建工作
     */
    public static class Builder {
        AuthorizeInterceptor authorizeInterceptor;

        public Builder() {
            authorizeInterceptor = new AuthorizeInterceptor();
        }

        public Builder setMid(String mid) {
            authorizeInterceptor.mid = mid;
            return this;
        }

        public Builder setAuthKey(String key) {
            authorizeInterceptor.authKey = key;
            return this;
        }

        public AuthorizeInterceptor build() {
            return authorizeInterceptor;
        }
    }
}
```



该加签方式,讲请求参数拼接为 `key=value` 的方式, 难点在于如何从 OkHttp 中获取这些参数,在 GET 请求和 POST 的处理方式又不同,代码中 POST 请求方式,又会根据请求传递的 `contentType` 而又有所不同,这里介绍了`Form`表单提交和 `Multipart` 上传文件的参数获取方式,其他的请举一反三.
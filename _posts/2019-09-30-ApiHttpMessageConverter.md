---
layout: post
title: 自定义消息转换器 WebMvcConfigurationSupport
date: 2019-09-30
tag: SpringBoot
---
### 自定义消息转换器 WebMvcConfigurationSupport

```java
@Configuration
public class WebMvcConfigurationConfig extends WebMvcConfigurationSupport{

	/**
	 * 配置自定义消息转换器 ApiHttpMessageConverter
	 * @return
	 */
    public ApiHttpMessageConverter apiHttpMessageConverter() {
		
        return new ApiHttpMessageConverter();
    } 

	/**
	 *fastJson转换器配置，并且将自定义的转换器添加进converters集合
	 */
	@Override
	protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
		// TODO Auto-generated method stub
		super.configureMessageConverters(converters);
		// 1.需要先定义一个convert 转换消息的对象
	    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
	    // 2.添加fastJson的配置信息,比如，是否需要格式化返回的json数据
	    FastJsonConfig fastJsonConfig = new FastJsonConfig();
	    
	    // 空值特别处理
	    // WriteNullListAsEmpty 将Collection类型字段的字段空值输出为[]
	    // WriteNullStringAsEmpty 将字符串类型字段的空值输出为空字符串 ""
	    // WriteNullNumberAsZero 将数值类型字段的空值输出为0
	    // WriteNullBooleanAsFalse 将Boolean类型字段的空值输出为false
	    fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat, SerializerFeature.WriteNullListAsEmpty,
	        SerializerFeature.WriteNullStringAsEmpty);
	    
	    // 处理中文乱码问题
	    List<MediaType> fastMediaTypes = new ArrayList<>();
	    fastMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
	    fastConverter.setSupportedMediaTypes(fastMediaTypes);
	    
	    // 3.在convert中添加配置信息
	    fastConverter.setFastJsonConfig(fastJsonConfig);
	    // 4.将convert添加到converters当中
	    converters.add(fastConverter);
	    //将自定义的转换器添加进converters集合 这里有个前后顺序 0表示最高优先级
	    converters.add(0,apiHttpMessageConverter());
	}
	
	/**
	 *如果继承了WebMvcConfigurationSupport，则在配置文件在中配置的相关内容会失效，需要重新指定静态资源
	 *这里指定swagger的html
	 */
	@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations(
                "classpath:/static/");
        registry.addResourceHandler("swagger-ui.html").addResourceLocations(
                "classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations(
                "classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }
	
	/**
	 * 跨域配置
	 */
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**").allowedOrigins("*").allowCredentials(true)
				.allowedMethods("GET", "POST", "DELETE", "PUT","OPTIONS").maxAge(3600);
	}
}

```
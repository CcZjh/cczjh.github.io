---
title: Springboot配置过滤器Filter
date: 2019-05-17 09:57:57
coverImg: /images/image-01.jpg
tags: ["Java", "Springboot", "Filter"]
categories: ["后端"]
---
### Springboot 配置 Filter过滤器
上代码:

#### 1. 首先自定义MyFilter 继承类 Filter
我们需要实现做的就是编写doFIlter类实现过滤逻辑

----------

	@Component
	@WebFilter(urlPatterns = "/*", filterName = "myFilter") // 这种是注解配置  下面有配置文件配置
	public class MyFilter implements Filter {

	    private static Logger log = LoggerFactory.getLogger(MyFilter.class);
	
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	        log.info("MyFilter参数初始化: "+filterConfig);
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        filterChain.doFilter(servletRequest, servletResponse);
	    }
	
	    @Override
	    public void destroy() {
	        log.info("MyFilter开始销毁...");
	    }
	}

----------

#### 2. 在Springboot 启动类中添加过滤器


----------

	// 配置文件配置
	@Bean
    public FilterRegistrationBean testFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean(new MyFilter());
        registration.addUrlPatterns("/*");
        registration.addInitParameter("paramName", "paramValue");
        registration.setName("myFilter");
		registration.setOrder(1); // 多个过滤器可控制执行顺序 数字越小优先级越高
        return registration;
    }

----------


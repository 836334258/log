- 拦截器

  ```java
  package work.king.blog.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.stereotype.Component;
  import org.springframework.web.servlet.LocaleResolver;
  import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
  import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
  
  import java.util.ArrayList;
  
  // 自定义mvc配置
  @Configuration
  public class MyMvcConfig implements WebMvcConfigurer {
      @Override
      public void addViewControllers(ViewControllerRegistry registry) {
          registry.addViewController("/mvc").setViewName("index");
      }
  
      /**
       * 拦截器
       *
       * @param registry
       */
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
          .excludePathPatterns("/error","/login","static/**");
      }
  }
  
  ```

  ```java
  package work.king.blog.config;
  
  import org.springframework.web.servlet.HandlerInterceptor;
  
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  
  //拦截器
  public class LoginHandlerInterceptor implements HandlerInterceptor {
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          Object username = request.getSession().getAttribute("username");
          if (username==null){
              request.getRequestDispatcher("/error").forward(request,response);
              return false;
          }
          return true;
      }
  }
  
  ```

  
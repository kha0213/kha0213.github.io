---
title:  "스프링 시큐리티 (Spring Security)"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - Spring
tags:
  - Spring security
last_modified_at: 2022-07-12T00:40:00-00:00
---

## 1. Spring Security

## Security Configuration

**👏TIP** Spring 2.7 이상이거나 Spring Security 5.7.1 이상에서   
**WebSecurityConfigurerAdapter** 은 **deprecated** 되었다.    
WebSecurityConfigurerAdapter을 상속받아 configure 재정의하는 방식에서    
SecurityFilterChain과 WebSecurityCustomizer을 Bean으로 등록하면 된다.         
{: .notice--info}   

```java
// Spring 2.7 or Spring Security 5.7.1 이전의 방식
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/login").permitAll()
            .antMatchers("/users/**", "/settings/**").hasAuthority("Admin")
            .hasAnyAuthority("Admin", "Editor", "Salesperson")
            .hasAnyAuthority("Admin", "Editor", "Salesperson", "Shipper")
            .anyRequest().authenticated()
            .and().formLogin()
            .loginPage("/login")
            .usernameParameter("email")
            .permitAll()
            .and()
            .rememberMe().key("AbcdEfghIjklmNopQrsTuvXyz_0123456789")
            .and()
            .logout().permitAll();

        http.headers().frameOptions().sameOrigin();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/images/**", "/js/**", "/webjars/**");
    }
}
```

```java
// Spring 2.7 or Spring Security 5.7.1 이후의 방식
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/login").permitAll()
            .antMatchers("/users/**", "/settings/**").hasAuthority("Admin")
            .hasAnyAuthority("Admin", "Editor", "Salesperson")
            .hasAnyAuthority("Admin", "Editor", "Salesperson", "Shipper")
            .anyRequest().authenticated()
            .and().formLogin()
            .loginPage("/login")
            .usernameParameter("email")
            .permitAll()
            .and()
            .rememberMe().key("AbcdEfghIjklmNopQrsTuvXyz_0123456789")
            .and()
            .logout().permitAll();
        
        http.headers().frameOptions().sameOrigin();
        
        return http.build();
    }

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring().antMatchers("/images/**", "/js/**", "/webjars/**");
    }
}
```
[https://codejava.net/frameworks/spring-boot/fix-websecurityconfigureradapter-deprecated](https://codejava.net/frameworks/spring-boot/fix-websecurityconfigureradapter-deprecated)   

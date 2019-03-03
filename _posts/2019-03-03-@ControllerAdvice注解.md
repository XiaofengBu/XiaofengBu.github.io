---
layout: post
title:  "SpringBoot中的@ControllerAdvice注解"
categories: SpringBoot
date:  2019-3-3 13:51:54
tags:  SpringBoot  
author: Xiaofeng
---

* content
{:toc}


在Springboot3.2中新增注解@CrontrollerAdvice用于定义@ExceptionHandler、@InitBinder、@ModelAttribute,并将其应用到所有的@RequestMapping中去,是一种切面的应用。其中最常用的是@ExceptionHandler,下面我们将重点介绍其在全局异常处理中的用法

### @InitBinder和@ModelAttribute的用法
```
@ControllerAdvice
public class MyExceptionHandler {

    /**
     * 应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器
     * @param binder
     */
    @InitBinder
    public void initWebBinder(WebDataBinder binder){
        //对日期的统一处理
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
        //添加对数据的校验
        //binder.setValidator();
    }

    /**
     * 把值绑定到Model中，使全局@RequestMapping可以获取到该值
     * @param model
     */
    @ModelAttribute
    public void addAttribute(Model model) {
        model.addAttribute("attribute",  "The Attribute");
    }
}
```
用法:
```
@Controller
public class DemoController {
  
    /**
   * 关于@ModelAttribute,
   * 可以使用ModelMap以及@ModelAttribute()来获取参数值。
   */    
    @RestMapping("/test1")
    public String testError(ModelMap modelMap ) {
         modelMap.get("attribute");//获取放入的The Attribute
    }

    @RestMapping("/test2")
    public String testTwo(@ModelAttribute("attribute") String attribute) {
    }
}
```
### @ExceptionHandler中的用法
异常处理是@ControllerAdvice中最常见的用法
我们定义一个全局异常处理类
```
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
    @ExceptionHandler(value = {BindException.class,GlobalException.class}) //拦截所有Controller里面发生的参数绑定异常和自定义的全局异常
    public Result<String> globalExceptionHandler(HttpServletRequest request, Exception e){ //返回一个定义好的Result 作为我们Controller的标准返回类
        e.printStackTrace();
        if(e instanceof BindException){//绑定时异常发生参数检查时 由spring-boot-starter-validation提供 注解来验证请求中的参数是否合法
            BindException bindException = (BindException) e;
            List<ObjectError> exceptionList = bindException.getAllErrors();//绑定异常可能有多个参数引发
            String msg = exceptionList.get(0).getDefaultMessage();
            return Result.error(CodeMsg.BIND_ERROR.fillArgs(msg));//CodeMsg主要是自定义的错误码已经对应的说明
        }else if(e instanceof LoginException){//登录异常为自定义异常类型,即密码不正确,使用这种方式可以简化Controller里面的代码
            return Result.error(CodeMsg.PASSWORD_ERROR);
        }else if (e instanceof MobileNotExistException){
            return Result.error(CodeMsg.MOBILE_NOT_EXIST);
        }else {
            return Result.error(CodeMsg.SERVER_ERROR);
        }

    }
}
```
定义一组自定义异常 其中全局异常为抽象类,是所有异常的父类 继承了RuntimeException,在Spring中运行时异常可以触发回滚操作。
```
public abstract class GlobalException extends RuntimeException{
    private static final long serialVersionUID = 1L;

    private CodeMsg cm;

    public GlobalException(CodeMsg cm) {
        super(cm.toString());
        this.cm = cm;
    }

    public CodeMsg getCm() {
        return cm;
    }
}
public class LoginException extends GlobalException {
    private static final long serialVersionUID = 1L;
    public LoginException() {
        super(CodeMsg.PASSWORD_ERROR);
    }
    public CodeMsg getCm() {
        return super.getCm();
    }
}
public class MobileNotExistException extends GlobalException {
    private static final long serialVersionUID = 1L;
    public MobileNotExistException() {
        super(CodeMsg.MOBILE_NOT_EXIST);
    }
    public CodeMsg getCm() {
        return super.getCm();
    }
}
public class ServerException extends GlobalException {
    private static final long serialVersionUID = 1L;
    public ServerException() {
        super(CodeMsg.SERVER_ERROR);
    }
    public CodeMsg getCm() {
        return super.getCm();
    }
}

登录相关Controller:
@Controller
@RequestMapping("/login")
public class LoginController {

    private static Logger logger = LoggerFactory.getLogger(LoginController.class);
    @Autowired
    private UserService userService;

    @RequestMapping("/doLogin")
    @ResponseBody
    public Result<Boolean> doLogin(@Valid LoginVO loginVO){
        userService.login(loginVO);
        return Result.success(true);
    }
}

参数类:
import javax.validation.constraints.NotNull;
@Data
public class LoginVO {
    @NotNull //validation提供非空注解
    @IsMobile //该注解为自定义注解不展开说明
    private String mobile;
    @NotNull
    private String password;
}

Service层:
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public User getUserById(Long id){
        return userMapper.getUserById(id);
    }

    public Boolean login(LoginVO loginVO) throws GlobalException{
        if(loginVO == null || loginVO.getPassword() == null || loginVO.getMobile() == null){
            throw new ServerException(); //参数检查时会过滤null 此分支如果走到抛出一个ServerException异常
        }
        String moblie = loginVO.getMobile();
        String password = loginVO.getPassword();
        User user = getUserById(Long.parseLong(moblie));
        if(user == null){
            throw new MobileNotExistException();//抛出用户不存在异常
        }
        String dbPass = user.getPassword();
        String dbSalt = user.getSalt();
        String calcPass = MD5Util.tranPass2DBPass(password,dbSalt);
        if(dbPass.equals(calcPass)){
            return true;//返回成功
        }else{
            throw new LoginException();//抛出登录失败异常
        }
    }

}
```
```
graph TD
A[请求/login/doLogin] -->B{检查参数是否抛出BindException}
    B --> |是|C[GlobalExceptionHandler拦截异常]
    C -->D[返回对应的错误码和错误信息Result]
    B --> |否|E[验证登录信息]
    E --> F{登录是否成功}
    F -->|是|G[返回成功的Result]
    F -->|否|J[返回失败的Result]
```
可以看出在Controller中我们不在使用if else判断登录行为以及检查参数,将所有非正常情况交给全局异常处理类去处理,业务只用关心最核心的部分,大大简化了业务代码,提高了重用性








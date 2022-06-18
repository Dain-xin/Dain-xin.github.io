---
title: Spring底层理解
date: 2022-06-15 21:50:17
categories:
- JAVA
- Spring
---

![spring](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206152205158.png)

# Spring运行思想

首先基本思路就是：配置、初始化、运行。

## 配置阶段

就是Spring使用，配饰web.xml，设定配置文件路径，配置服务层，控制层，参数，请求路径的注解。

## 初始化阶段

继承HttpServlet重写dopost、doget、和init方法。

```java
private static final String LOCATION = "contextConfigLocation";
//保存Spring主配置文件
private Properties p = new Properties();
//缓存保存的类名
private List<String> classNames = new ArrayList<String>();
//存储Bean的IoC容器声明
private Map<String, Object> ioc = new HashMap<String, Object>();
//保存所有的Url和方法的映射关系 --- 缓存url的对应关系
private List<Handler> handlerMapping = new ArrayList<Handler>();
```

### 加载配置文件

通过输入流读取配置文件，加载到主配置文件中。

```java
//1、加载配置文件
doLoadConfig(config.getInitParameter(LOCATION));


private void doLoadConfig(String location) {
        InputStream fis = null;
        try {
            fis = this.getClass().getClassLoader().getResourceAsStream(location);
            //1、读取配置文件
            p.load(fis);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != fis) {
                    fis.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

### 扫描包中的所有的类

将保存在主配置文件中的数据递归遍历，将得到的classnames文件名保存起来。

```java
//2、扫描所有相关的类
doScanner(p.getProperty("scanPackage"));

    /**
     * 递归扫描出所有Class文件
     *
     * @param packageName
     */
    private void doScanner(String packageName) {
        //将所有的包路径转换为文件路径
        URL url = this.getClass().getClassLoader().getResource("/" + packageName.replaceAll("\\.", "/"));
        File dir = new File(url.getFile());
        for (File file : dir.listFiles()) {
            //如果是文件夹，继续递归
            if (file.isDirectory()) {
                doScanner(packageName + "." + file.getName());
            } else {
                classNames.add(packageName + "." + file.getName().replace(".class", "").trim());
            }
        }
    }
```



### 初始化所有相关类的实例，并保存到IOC容器中

将得到的classnames文件名反射，将带有注解的的类，处理变成符合开发条件的命名规则，然后进行实例化，保存在ioc的map类型的集合中。

```java
//3、初始化所有相关类的实例，并保存到IOC容器中
doInstance();

private void doInstance() {
        if (classNames.size() == 0) {
            return;
        }

        try {
            for (String className : classNames) {
                Class<?> clazz = Class.forName(className);
                if (clazz.isAnnotationPresent(GPController.class)) {
                    //默认将首字母小写作为beanName
                    String beanName = lowerFirst(clazz.getSimpleName());
                    ioc.put(beanName, clazz.newInstance());
                } else if (clazz.isAnnotationPresent(GPService.class)) {

                    GPService service = clazz.getAnnotation(GPService.class);
                    String beanName = service.value();
                    //如果用户设置了名字，就用用户自己设置
                    if (!"".equals(beanName.trim())) {
                        ioc.put(beanName, clazz.newInstance());
                        continue;
                    }

                    //如果自己没设，就按接口类型创建一个实例
                    Class<?>[] interfaces = clazz.getInterfaces();
                    for (Class<?> i : interfaces) {
                        ioc.put(i.getName(), clazz.newInstance());
                    }

                } else {
                    continue;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



### 依赖注入DI

将上一步ioc集合实例对象拿到全部的属性，遍历对象进行注入。反射可以访问类私有属性。

```java
//4、依赖注入
doAutowired();


    private void doAutowired() {
        if (ioc.isEmpty()) {
            return;
        }

        for (Entry<String, Object> entry : ioc.entrySet()) {
            //拿到实例对象中的所有属性
            Field[] fields = entry.getValue().getClass().getDeclaredFields();
            for (Field field : fields) {

                if (!field.isAnnotationPresent(GPAutowired.class)) {
                    continue;
                }

                GPAutowired autowired = field.getAnnotation(GPAutowired.class);
                String beanName = autowired.value().trim();
                if ("".equals(beanName)) {
                    beanName = field.getType().getName();
                }
                field.setAccessible(true); //设置私有属性的访问权限
                try {
                    field.set(entry.getValue(), ioc.get(beanName));
                } catch (Exception e) {
                    e.printStackTrace();
                    continue;
                }
            }
        }
    }
```

### 构造HandlerMapping。

通过类加载器找到控制层的url路径和方法的url路径，将得到的路径初始化给handlermapping 他是List集合。

```java
//5、构造HandlerMapping
initHandlerMapping();


    private void initHandlerMapping() {
        if (ioc.isEmpty()) {
            return;
        }

        for (Entry<String, Object> entry : ioc.entrySet()) {
            Class<?> clazz = entry.getValue().getClass();
            if (!clazz.isAnnotationPresent(GPController.class)) {
                continue;
            }

            String url = "";
            //获取Controller的url配置
            if (clazz.isAnnotationPresent(GPRequestMapping.class)) {
                GPRequestMapping requestMapping = clazz.getAnnotation(GPRequestMapping.class);
                url = requestMapping.value();
            }

            //获取Method的url配置
            Method[] methods = clazz.getMethods();
            for (Method method : methods) {

                //没有加RequestMapping注解的直接忽略
                if (!method.isAnnotationPresent(GPRequestMapping.class)) {
                    continue;
                }

                //映射URL
                GPRequestMapping requestMapping = method.getAnnotation(GPRequestMapping.class);
                String regex = ("/" + url + requestMapping.value()).replaceAll("/+", "/");
                Pattern pattern = Pattern.compile(regex);
                handlerMapping.add(new Handler(pattern, entry.getValue(), method));
                System.out.println("mapping " + regex + "," + method);
            }
        }
    }
```

## 运行阶段

调用dopost、doget方法，根据输入的路径获取到url的数据，找到调用的方法，然后利用反射方法返回数据，将数据打印到浏览器

doPost、doGet方法执行业务调用doDispatch调度方法。得到请求路径的匹配对象的参数列表，然后设置方法中的request和response对象，然后通过动态代理invke调用方法。将结果打印即可。

### doDispatch调度

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        this.doPost(req, resp);
    }

    /**
     * 执行业务处理
     */
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        try {
            doDispatch(req, resp); //开始始匹配到对应的方法

        } catch (Exception e) {
            //如果匹配过程出现异常，将异常信息打印出去
            resp.getWriter().write("500 Exception,Details:\r\n" + Arrays.toString(e.getStackTrace()).replaceAll("\\[|\\]", "").replaceAll(",\\s", "\r\n"));
        }
    }



    /**
     * 匹配URL
     *
     * @param req
     * @param resp
     * @return
     * @throws Exception
     */
    private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws Exception {

        try {
            Handler handler = getHandler(req);

            if (handler == null) {
                //如果没有匹配上，返回404错误
                resp.getWriter().write("404 Not Found");
                return;
            }

            //获取方法的参数列表
            Class<?>[] paramTypes = handler.method.getParameterTypes();

            //保存所有需要自动赋值的参数值
            Object[] paramValues = new Object[paramTypes.length];

            Map<String, String[]> params = req.getParameterMap();
            for (Entry<String, String[]> param : params.entrySet()) {
                String value = Arrays.toString(param.getValue()).
						replaceAll("\\[|\\]", "").
						replaceAll(",\\s", ",");

                //如果找到匹配的对象，则开始填充参数值
                if (!handler.paramIndexMapping.containsKey(param.getKey())) {
                    continue;
                }
                //得到顺序
                int index = handler.paramIndexMapping.get(param.getKey());
                paramValues[index] = convert(paramTypes[index], value);
            }


            //设置方法中的request和response对象
            int reqIndex = handler.paramIndexMapping.get(HttpServletRequest.class.getName());
            paramValues[reqIndex] = req;
            int respIndex = handler.paramIndexMapping.get(HttpServletResponse.class.getName());
            paramValues[respIndex] = resp;

            handler.method.invoke(handler.controller, paramValues);

        } catch (Exception e) {
            throw e;
        }
    }

```

### getHandler记录对应关系

通过request请求参数，记录Controller中的RequestMapping和Method的对应关系——handler

```java
private Handler getHandler(HttpServletRequest req) throws Exception {
    if (handlerMapping.isEmpty()) {
        return null;
    }

    String url = req.getRequestURI();
    String contextPath = req.getContextPath();
    url = url.replace(contextPath, "").replaceAll("/+", "/");

    for (Handler handler : handlerMapping) {
        try {
            Matcher matcher = handler.pattern.matcher(url);
            //如果没有匹配上继续下一个匹配
            if (!matcher.matches()) {
                continue;
            }

            return handler;
        } catch (Exception e) {
            throw e;
        }
    }
    return null;
}


private Object convert(Class<?> type, String value) {
    if (Integer.class == type) {
        return Integer.valueOf(value);
    }
    return value;
}

/**
 * Handler记录Controller中的RequestMapping和Method的对应关系
 *
 * @author Tom
 * 内部类
 */
private class Handler {

    protected Object controller;    //保存方法对应的实例
    protected Method method;        //保存映射的方法
    protected Pattern pattern;        //用正则保存URL的规则
    protected Map<String, Integer> paramIndexMapping;    //形式参数列表的顺序

    /**
     * 构造一个Handler基本的参数
     *
     * @param controller
     * @param method
     */
    protected Handler(Pattern pattern, Object controller, Method method) {
        this.controller = controller;
        this.method = method;
        this.pattern = pattern;

        paramIndexMapping = new HashMap<String, Integer>();
        putParamIndexMapping(method);
    }

    private void putParamIndexMapping(Method method) {

        //提取方法中加了注解的参数
        Annotation[][] pa = method.getParameterAnnotations();
        for (int i = 0; i < pa.length; i++) {
            for (Annotation a : pa[i]) {
                if (a instanceof GPRequestParam) {
                    String paramName = ((GPRequestParam) a).value();
                    if (!"".equals(paramName.trim())) {
                        paramIndexMapping.put(paramName, i);
                    }
                }
            }
        }

        //提取方法中的request和response参数
        Class<?>[] paramsTypes = method.getParameterTypes();
        for (int i = 0; i < paramsTypes.length; i++) {
            Class<?> type = paramsTypes[i];
            if (type == HttpServletRequest.class ||
                    type == HttpServletResponse.class) {
                paramIndexMapping.put(type.getName(), i);
            }
        }
    }
}
```

# 顶层设计

```java
//初始化阶段
@Override
public void init(ServletConfig config) throws ServletException {
    //===========  IOC和DI     ============
    context = new GPApplicationContext(config.getInitParameter("contextConfigLocation"));
    //===========  MVC九大组件  ============
    initStrategies(context);
    System.out.println("GP Spring framework is init.");
}
```



# IOC-DI顶层设计

为了符合单一职责原则的设计模式原则，要把原来（加载配置文件、扫描包、初始化所有相关类的实例）IOC和DI（依赖注入）部分提取出来。

![微信图片_20220616210843](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206162109009.png)

调用IOC容器，先从ApplicationContext的getBean方法开始。

```java
//容器对象完成IOC和DI
public GPApplicationContext(String... configLocations) {
        this.configLocations = configLocations;

        try {
            //1、读取配置文件
            reader = new GPBeanDefinitionReader(configLocations);

            //2、解析配置文件，将配置信息变成BeanDefinition对象
            List<GPBeanDefinition> beanDefinitions = reader.loadBeanDefinitions();

            //3、把BeanDefinition对应的实例注册到beanDefinitionMap key=beanName,value=beanDefinition对象
            doRegisterBeanDefinition(beanDefinitions);

            //配置信息初始化阶段就完成

            //4、完成依赖注入，什么时候注入？在调用getBean()的才注入
            doAutowrited();
        }catch (Exception e){
            e.printStackTrace();
        }

    }

//得到Bean对象IOC
public Object getBean(String beanName){
        //1、获取BeanDefinition配置信息
        GPBeanDefinition beanDefinition = this.beanDefinitionMap.get(beanName);

        //2、用反射实例化
        Object instance = instantiateBean(beanName,beanDefinition);

        //3、将创建出来的实例包装为BeanWrapper对象
        GPBeanWrapper beanWrapper = new GPBeanWrapper(instance);

        //4、把BeanWrapper对象存入到真正的IoC容器中
        this.factoryBeanInstanceCache.put(beanName,beanWrapper);

        //5、执行依赖注入
        populateBean(beanName,beanDefinition,beanWrapper);

        return this.factoryBeanInstanceCache.get(beanName).getWrapperInstance();
    }

//2、创建实例化对象
    private Object instantiateBean(String beanName, GPBeanDefinition beanDefinition) {
        String className = beanDefinition.getBeanClassName();
        Object instance = null;
        try {
            Class<?> clazz = Class.forName(className);
            instance = clazz.newInstance();

            //Spring内部的容器不止一个
            factoryBeanObjectCache.put(beanName,instance);
        }catch (Exception e){
            e.printStackTrace();
        }
        return instance;
    }


//5、完成依赖注入DI
    private void populateBean(String beanName, GPBeanDefinition beanDefinition, GPBeanWrapper beanWrapper) {

        Object instance = beanWrapper.getWrapperInstance();
        Class<?> clazz = beanWrapper.getWrapperClass();

        //只有加了注解的才进行依赖注入
        //@Component
        if(!(clazz.isAnnotationPresent(GPController.class) || clazz.isAnnotationPresent(GPService.class))){
            return;
        }

        //拿到IoC容器所有实例的字段（属性）
        //private protected default public
        //OOP 只能拿到public属性
        Field[] fields = clazz.getDeclaredFields();

        for (Field field : fields) {
            if(!field.isAnnotationPresent(GPAutowired.class)){
                continue;
            }

            GPAutowired autowired = field.getAnnotation(GPAutowired.class);
            String autowiredBeanName = autowired.value().trim();
            if("".equals(autowiredBeanName)){
                autowiredBeanName = field.getType().getName();
            }

            //暴力访问
            field.setAccessible(true);

            try {
                //field相当于 @GPAutowired private IDemoService demoService;
                //entry.getValue()相当于DemoAction的实例
                //ioc.get(beanName)相当于从IoC容器中去拿到key为com.gupaoedu.demo.service.IDemoService 对应的实例
                // 即DemoService的实例
                if(this.factoryBeanInstanceCache.get(autowiredBeanName) == null){
                    continue;
                }
                field.set(instance,this.factoryBeanInstanceCache.get(autowiredBeanName).getWrapperInstance());
            } catch (IllegalAccessException e) {
                e.printStackTrace();
                continue;
            }

        }

    }


```

先初始化ApplicationContext对象，构造方法中

1、加载配置文件

2、将所有的配置文件解析为BeanDefinition对象，保存在List集合中。

3、将BeanDefinition对象缓存到BeanDefinitionMap中。通过doRegistryBeanDefiniition方法，把List遍历put到BeanDefinitionMap中。

4、最后创建对象。通过doCreateBean方法，得到Map中entry，通过getBean方法得到实例。

在getBean方法中，通过Map得到beanName和BeanDefinition实例化，然后把创建的instance实例封装进BeanWrapper（装饰器模式）中，最后将beanWapper（Map集合）缓存到IOC容器

5、完成依赖注入。通过populateBean方法，通过beanWrapper得到实例，然后拿到全部的属性，遍历对象进行依赖注入。



名词理解：

getBean：从IOC中获取Bean

BeanDefinition：定义bean，将配置文件读取到内存后避免反复读取和解析，为什莫需要封装一个Definition：多种配置文件，Annotation，yml等。

BeanWrapper：包装bean，存储对象，利用装饰器模式，把原生对象和代理对象全部包装起来。

BeanDefinitionReader：工具类，读取配置文件，把配置文件解析为beanDefinition对象。



# MVC顶层设计

MVC九大组件:

![image-20220617201855374](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206172018459.png)

请求转发:forword,请求参数不丢失

重定向:redirect,请求参数不保留

处理映射器(HandlerMapping)、处理适配器(HandlerAdapter)、视图解析器(ModelAndView)

```java
private void initStrategies(GPApplicationContext context) {
    //handlerMapping
    initHandlerMappings(context);
    //初始化参数适配器，url和method的匹配。对应了前面的 构造HandlerMapping
    initHandlerAdapters(context);
    //初始化视图转换器
    initViewResolvers(context);
}


//方法映射

 private void initHandlerMappings(GPApplicationContext context) {
        if(context.getBeanDefinitionCount() == 0){ return; }

        String [] beanNames = context.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            Object instance = context.getBean(beanName);
            Class<?> clazz = instance.getClass();
            if(!clazz.isAnnotationPresent(GPController.class)){ continue; }

            String baseUrl = "";
            if(clazz.isAnnotationPresent(GPRequestMapping.class)){
                GPRequestMapping requestMapping = clazz.getAnnotation(GPRequestMapping.class);
                baseUrl = requestMapping.value();
            }

            //默认只获取public方法
            for (Method method : clazz.getMethods()) {
                if(!method.isAnnotationPresent(GPRequestMapping.class)){continue;}
                GPRequestMapping requestMapping = method.getAnnotation(GPRequestMapping.class);
                //   //demo//query
                String regex = ("/" + baseUrl + "/" + requestMapping.value().replaceAll("\\*",".*")).replaceAll("/+","/");
                Pattern pattern = Pattern.compile(regex);
                handlerMappings.add(new GPHandlerMapping(pattern,instance,method));
                System.out.println("Mapped " + regex + "," + method);

            }
        }
    }



    private void initHandlerAdapters(GPApplicationContext context) {
        for (GPHandlerMapping handlerMapping : handlerMappings) {
            this.handlerAdapters.put(handlerMapping,new GPHandlerAdapter());
        }
    }


    private void initViewResolvers(GPApplicationContext context) {
        //模板引擎的根路径
        String tempateRoot = context.getConfig().getProperty("templateRoot");
        String templateRootPath = this.getClass().getClassLoader().getResource(tempateRoot).getFile();

        File templateRootDir = new File(templateRootPath);
        for (File file : templateRootDir.listFiles()) {
            this.viewResolvers.add(new GPViewResolver(tempateRoot));
        }
    }

```





## 运行阶段

```java
//运行阶段
    private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws Exception {

        //1、根据URL拿到对应的HandlerMapping对象
        GPHandlerMapping handler = getHandler(req);

        if(null == handler){
            processDispatchResult(req,resp,new GPModelAndView("404"));
            return;
        }

        //2、根据HandlerMapping获得一个HandlerAdapter
        GPHandlerAdapter ha = getHandlerAdapter(handler);

        //3、根据HandlerAdapter拿到一个ModelAndView
        GPModelAndView mv = ha.handle(req,resp,handler);

        //4、根据ViewResolver根据ModelAndView去拿到View
        processDispatchResult(req,resp,mv);

    }


 //404 、500 、自定义模板
    private void processDispatchResult(HttpServletRequest req, HttpServletResponse resp, GPModelAndView mv) throws Exception {
        if(null == mv){ return; }

        if(this.viewResolvers.isEmpty()){return;}

        for (GPViewResolver viewResolver : this.viewResolvers) {
            GPView view = viewResolver.resolveViewName(mv.getViewName());
            view.render(mv.getModel(),req,resp);
            return;
        }
    }
```


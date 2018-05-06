---
layout: post
title:  "3 types of delegation pattern"
categories: computer
tags:  computer copy java design_pattern aop
author: "岑宇"
source: "https://www.cnblogs.com/cenyu/p/6289209.html"
---

* content
{:toc}



Java的三种代理模式
===========

1.代理模式
------

代理(Proxy)是一种设计模式,提供了对目标对象另外的访问方式;即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能.  
这里使用到编程中的一个思想:不要随意去修改别人已经写好的代码或者方法,如果需改修改,可以通过代理的方式来扩展该方法

举个例子来说明代理的作用:假设我们想邀请一位明星,那么并不是直接连接明星,而是联系明星的经纪人,来达到同样的目的.明星就是一个目标对象,他只要负责活动中的节目,而其他琐碎的事情就交给他的代理人(经纪人)来解决.这就是代理思想在现实中的一个例子

用图表示如下:

![](data:image;base64,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)

代理模式的关键点是:代理对象与目标对象.代理对象是对目标对象的扩展,并会调用目标对象

<!--more-->

### 1.1.静态代理

静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类.

下面举个案例来解释:  
模拟保存动作,定义一个保存动作的接口:IUserDao.java,然后目标对象实现这个接口的方法UserDao.java,此时如果使用静态代理方式,就需要在代理对象(UserDaoProxy.java)中也实现IUserDao接口.调用的时候通过调用代理对象的方法来调用目标对象.  
需要注意的是,代理对象与目标对象要实现相同的接口,然后通过调用相同的方法来调用目标对象的方法

代码示例:  
接口:IUserDao.java

    /**
     * 接口
     */
    public interface IUserDao {
    
        void save();
    }

目标对象:UserDao.java

    /**
     * 接口实现
     * 目标对象
     */
    public class UserDao implements IUserDao {
        public void save() {
            System.out.println("----已经保存数据!----");
        }
    }

代理对象:UserDaoProxy.java

    /**
     * 代理对象,静态代理
     */
    public class UserDaoProxy implements IUserDao{
        //接收保存目标对象
        private IUserDao target;
        public UserDaoProxy(IUserDao target){
            this.target=target;
        }
    
        public void save() {
            System.out.println("开始事务...");
            target.save();//执行目标对象的方法
            System.out.println("提交事务...");
        }
    }

测试类:App.java

    /**
     * 测试类
     */
    public class App {
        public static void main(String[] args) {
            //目标对象
            UserDao target = new UserDao();
    
            //代理对象,把目标对象传给代理对象,建立代理关系
            UserDaoProxy proxy = new UserDaoProxy(target);
    
            proxy.save();//执行的是代理的方法
        }
    }

**静态代理总结:**  
1.可以做到在不修改目标对象的功能前提下,对目标功能扩展.  
2.缺点:

*   因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护.

如何解决静态代理中的缺点呢?答案是可以使用动态代理方式

### 1.2.动态代理

**动态代理有以下特点:**  
1.代理对象,不需要实现接口  
2.代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)  
3.动态代理也叫做:JDK代理,接口代理

**JDK中生成代理对象的API**  
代理类所在包:java.lang.reflect.Proxy  
JDK实现代理只需要使用newProxyInstance方法,但是该方法需要接收三个参数,完整的写法是:

    static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )

注意该方法是在Proxy类中是静态方法,且接收的三个参数依次为:

*   `ClassLoader loader,`:指定当前目标对象使用类加载器,获取加载器的方法是固定的
*   `Class<?>[] interfaces,`:目标对象实现的接口的类型,使用泛型方式确认类型
*   `InvocationHandler h`:事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入

代码示例:  
接口类IUserDao.java以及接口实现类,目标对象UserDao是一样的,没有做修改.在这个基础上,增加一个代理工厂类(ProxyFactory.java),将代理类写在这个地方,然后在测试类(需要使用到代理的代码)中先建立目标对象和代理对象的联系,然后代用代理对象的中同名方法

代理工厂类:ProxyFactory.java

    
    /**
     * 创建动态代理对象
     * 动态代理不需要实现接口,但是需要指定接口类型
     */
    public class ProxyFactory{
    
        //维护一个目标对象
        private Object target;
        public ProxyFactory(Object target){
            this.target=target;
        }
    
       //给目标对象生成代理对象
        public Object getProxyInstance(){
            return Proxy.newProxyInstance(
                    target.getClass().getClassLoader(),
                    target.getClass().getInterfaces(),
                    new InvocationHandler() {
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            System.out.println("开始事务2");
                            //执行目标对象方法
                            Object returnValue = method.invoke(target, args);
                            System.out.println("提交事务2");
                            return returnValue;
                        }
                    }
            );
        }
    
    }

测试类:App.java

    
    /**
     * 测试类
     */
    public class App {
        public static void main(String[] args) {
            // 目标对象
            IUserDao target = new UserDao();
            // 【原始的类型 class cn.itcast.b_dynamic.UserDao】
            System.out.println(target.getClass());
    
            // 给目标对象，创建代理对象
            IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
            // class $Proxy0   内存中动态生成的代理对象
            System.out.println(proxy.getClass());
    
            // 执行方法   【代理对象】
            proxy.save();
        }
    }

**总结:**  
代理对象不需要实现接口,但是目标对象一定要实现接口,否则不能用动态代理

### 1.3.Cglib代理

上面的静态代理和动态代理模式都是要求目标对象是实现一个接口的目标对象,但是有时候目标对象只是一个单独的对象,并没有实现任何的接口,这个时候就可以使用以目标对象子类的方式类实现代理,这种方法就叫做:Cglib代理

Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.

*   JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
*   Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的interception(拦截)
*   Cglib包的底层是通过使用一个小而块的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

Cglib子类代理实现方法:  
1.需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,所以直接引入`pring-core-3.2.5.jar`即可.  
2.引入功能包后,就可以在内存中动态构建子类  
3.代理的类不能为final,否则报错  
4.目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.

代码示例:  
目标对象类:UserDao.java

    /**
     * 目标对象,没有实现任何接口
     */
    public class UserDao {
    
        public void save() {
            System.out.println("----已经保存数据!----");
        }
    }

Cglib代理工厂:ProxyFactory.java

    /**
     * Cglib子类代理工厂
     * 对UserDao在内存中动态构建一个子类对象
     */
    public class ProxyFactory implements MethodInterceptor{
        //维护目标对象
        private Object target;
    
        public ProxyFactory(Object target) {
            this.target = target;
        }
    
        //给目标对象创建一个代理对象
        public Object getProxyInstance(){
            //1.工具类
            Enhancer en = new Enhancer();
            //2.设置父类
            en.setSuperclass(target.getClass());
            //3.设置回调函数
            en.setCallback(this);
            //4.创建子类(代理对象)
            return en.create();
    
        }
    
        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            System.out.println("开始事务...");
    
            //执行目标对象的方法
            Object returnValue = method.invoke(target, args);
    
            System.out.println("提交事务...");
    
            return returnValue;
        }
    }

测试类:

    /**
     * 测试类
     */
    public class App {
    
        @Test
        public void test(){
            //目标对象
            UserDao target = new UserDao();
    
            //代理对象
            UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();
    
            //执行代理对象的方法
            proxy.save();
        }
    }

在Spring的AOP编程中:  
如果加入容器的目标对象有实现接口,用JDK代理  
如果目标对象没有实现接口,用Cglib代理

作者：[岑宇](http://www.cnblogs.com/cenyu/)   
出处：[http://www.cnblogs.com/cenyu/](http://www.cnblogs.com/cenyu/)   
本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。   
如果文中有什么错误，欢迎指出。以免更多的人被误导。





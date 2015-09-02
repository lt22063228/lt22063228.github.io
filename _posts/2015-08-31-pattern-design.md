---
layout: post
title: "pattern design:设计模式"
description: ""
category: pattern design
tags: [pattern design]
---
{% include JB/setup %}

### 策略模式

> ***策略模式***定义了算法族,分别封装起来,让它们之间可以 互相替换,此模式让算法的变化独立于算法的客户.

设计思路是:让代码中变化的部分与不变的部分分离开来.

例子: 你要实现一个定价系统,用户分低'中'高等,你希望给不同的用户制定不同的定价策略

#### 错误实现
设计一个Price类,所有不同的定价策略继承这个类,实现自己的定价策略.
定价策略是多种因素的组合.它可能包括会员级别,可能包括节假日折扣.那么,对于每种会员级别,节假日折扣的组合,都必须要有一个子类完成实现.
这样会造成子类数目迅速膨胀.

#### 使用策略模式
设计一个价格类Price,其中包含定价策略PriceStragedy.
分别为三个等级的用户提供定价策略的实现.然后价格类中包含PriceStragedy的引用.

这里,Price类代表代码中不变的部分,而PriceStragedy就是可变部分,因为今后可能有新的定价策略,因此PriceStragedy将代码中可变部分抽离出来,实现了代码的解耦.

### 观察者模式

> ***观察者模式***定义了对象之间的一对多依赖,这样一来,当一个对象改变状态时,它的所有依赖者都会收到通知并自动更新

设计思路:让观察者与主题松耦合,松耦合可以通过接口来实现.
在观察者那一方,主题提供了注册(register)和注销(unregister)的接口,让观察者可以加入和退出对主题的观察,而不管主题是什么样的主题.如果将来对其他主题感兴趣,那么还能使用一样的代码,因为主题的接口不变,始终是注册和注销.
在主题那一方,观察者提供了更新(update)的接口,这样主题就能及时地将状态更新通知给观察者,而不管是何观察者.如果有其他的观察者加入,还能使用一样的代码,因为观察者的接口不变,始终是更新(update)

例子:要实现一个气象站布告板,提供WeatherData供各个布告板获取数据,但是将来可能有不同的布告板.
#### 错误实现
直接给代码,这里通知观察者的代码"面向实现编程",当有新的观察者加入之后,需要改变主题部分的代码:
{% highlight java %}
public class WeatherData {
	public void measurementsChanged() {
    	float temp = getTemperature();
        float humidity = getHumidity();
        float pressure = getPressure();
        
        currentConditionsDisplay.update(temp, humidity, pressure);
        statisticsDisplay.update(temp, humidity, pressure);
        forecastDisplay.update(temp, humidity, pressure);
    }
}
{% endhighlight %}

这里,调用update方法的观察者是改变的部分,因此应该将这部分进行封装,面向接口编程.
如果将来经常改变的还包括气象数据,那么就应该将气象数据定义成一个接口,使用通用的获取数据getData接口.

#### 改进的实现
{% highlight java %}
public class WeatherData {
	List<Observer> observers = new ArrayList<Observers>(); //将观察者存在一个容器中, Observer是观察者接口
	public void measurementsChanged() {
    	for(Observer ob : observers) ob.update();
    }
}
{% endhighlight %}

#### JDK中的观察者模式

java的Swing组件中的按钮JButton就是一个主题(Subject),提供addActionListener注册观察者,每个观察者是一个ActionListener接口,其中的actionPerformed(ActionEvent event)相当于观察者的update方法.

#### 设计原则
###### 找出程序中会变化的方面,然后将其和固定不变的方面相分离

> 观察者就是会变化的方面,主题通过将观察者统一成同一接口,就可以在观察者改变的情况下提供更新通知的服务了(只要观察者提供了更新的接口)

###### 针对接口编程,不针对实现编程

> 观察者在主题类中的引用就是一系列接口

###### 多用组合,少用继承

> 如果一个类既是观察者,又是主题.那么就可以用组合的方式,维护两个引用,一个是观察者接口,一个是主题接口.提供了接口,将来两端(左边的主题,右边的观察者)的代码就可以不用改变.即使会出现多种观察者和多种主题,那么这个类也无需改动.

### 装饰者模式

例子:假设要设计一个Coffee类,其中包含各种类型的咖啡:HouseBlend, DarkRoast, Espresso, Decaf.
还包括各种类型的调料:Milk, Mocha, Soy, Whip.
客户可能订购其中任何一种咖啡和任意调料的任意组合,如:HouseBlend, 1 Milk, 1 Mocha, 2 Soy.

这里,改变的因素是调料:可能存在多分的调料,调料类型可能增加

#### 错误的设计
可能的选择是:定义一个Coffee抽象类,其中包含调料的成员变量.
这么做的问题是:每当增加一个调料,就得改变超类的定义,增加新的成员定义

#### 使用装饰者

咖啡类与调料类共享一个超类Coffee.(装饰这类反映出被装饰的组件类型,通过共享接口或者父类来实现).
调料类在可以在咖啡(被装饰者)定价行为之前或者之后加入自己的行为,甚至将被装饰者的行为整个取代掉,而达到装饰的目的.
关键代码是,定价的时候,加入自己的行为:
{% highlight java %}
public class Ingredients extends Coffee {
	Coffee coffee; //传入的参数是未被当前装饰器装饰的类
    int cost; // 调料的价格
    public Ingredients(Coffee coffee) {
    	this.coffee = coffee;
    }
	public int cost() {
    	return coffee.cost() + cost;
    }
}
{% endhighlight %}

重点是:1.装饰者和被装饰者共享类型. 2.装饰者在被装饰者的行为上进行扩展.

#### JDK中的装饰者模式

> 共享的类型:InputStream
> 基本的被装饰对象:FileInputStream, StringBufferInputStream, ByteArrayInputStream
> 抽象装饰者:FilterInputStream
> 具体的装饰这:PushbackInputStream, BufferedInputStream, DataInputStream, LineNumberInputStream.

### 工厂模式

#### 定义

> ***工厂方法模式***定义了一个创建对象的接口,但由子类决定要实例化的类是哪一个,工厂方法让类把实例化推迟到子类
> ***抽象工厂模式***提供一个接口,用于创建与之相关或者其所依赖的对象的家族,而不需要明确指明具体类型.

#### 具体实例

要实现订购披萨的方法,其中一种可能实现如下:
{% highlight java %}
public Pizza orderPizza(String type) {
	Pizza pizza;
    
    // following code is coupled with implementation.
    if(type.equals("cheese")) {
    	pizza = new CheesePizza();
    } else if (type.equals("greek")) {
    	pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
    	pizza = new PepperoniPizza();
    }
    
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
{% endhighlight %}

虽然大部分代码的确是针对接口编程,实现了和具体类型的解耦.但是,在对象创建的部分,还是难免接触到实现的类型.将来若有更多的类型,那么这段代码需要修改.

#### 工厂方法

上述问题可以通过提供工厂方法来解决,工厂方法提供一个接口,由子类决定创建的具体类型,使orderPizza不依赖于任何具体实现,如下:
{% highlight java %}
public Pizza orderPizza(String type) {
	// 通过抽象方法来返回具体类型
	Pizza pizza = createPizza(type);
    
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    
    return pizza;
}

// 这个方法由子类来实现
public abstract Pizza createPizza(String type);

{% endhighlight %}

可以发现,上述代码没有接触任何具体的实现,如果想要加入一种披萨,只要继承它,实现相应的createPizza.

#### 抽象工厂

如果类中还有其他的依赖对象,例如原料,可以使用一个抽象接口来实现.
{% highlight java %}
// 抽象的接口
Ingredient ingredient;
// 实例化时候传入相应的原料实现
public void setIngredient(Ingredient ingredient) {
	this.ingredient = ingredient;
}

// 任何需要具体调料的方法,又接口ingredient返回
public void changeTast() {
	pizza.bake(ingredient.getIngredient());
}
{% endhighlight %}

#### 总结

在实现一个类时,类中每当存在一个对象引用,那么它的依赖就多一些.如果这些对象对应的类不会发生变化,那么这个类也不会变化.
如果对应的类频繁的变化,就应该将类的引用替换为接口的引用,试图保证接口是相对稳定的,而改变的是具体的类.这样,使用接口的类的代码就不需要频繁的修改.

### 单件模式

***单件模式***确保一个类只有一个实例,并提供一个全局访问点.

经典实现:
{% highlight java %}
public class Singleton {
	private static Singleton uniqueInstance = null;
    // 私有构造器保证外部代码无法实例化该类
    private Singleton() {}
    
    // 该单例类提供的全局访问点
    public static Singleton getInstance() {
    	return uniqueInstance;
    }
}
{% endhighlight %}

在多线程环境下,需要确保单件的产生被同步,有三种方法完成单件的正确初始化.
1. 静态初始化
2. 将getInstance方法锁定
3. 双重检查加锁(为了应付加锁操作造成的高开销):
 {% highlight java %}
 public class Singleton {
 	private volatile static Singleton uniqueInstance = null;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
    	if(uniqueInstance == null) {
        	synchronized(Singleton.class) {
            	if(uniqueInstance == null) {
                	uniqueInstance = new Singleton();
                }
            }
        }
        
        return uniqueInstance;
    }
 }
 {% endhighlight %}
 
###  命令模式

***命令模式***将"请求"封装成对象,以便使用不同的请求,队列或者日志来参数化其他对象.命令模式也支持可撤销的操作.

当编写一个类,其中引用到一些其他类的对象来实现一些操作,并且知道这些对象的类型很可能在将来发生变化.
一种很自然的想法是使用*策略模式*,给每一个不同的类型实现一个子类
但是命令模式的重点在于实现"命令请求者和命令接收者之间的解耦",这意味着命令的发出者不需要知道命令的接受者是什么类型,它有很广阔的一种类型--那就是"命令".
而策略模式不同,它需要提供不同的算法实现,那么侧重点在于可扩展性,命令的发出者(算法的使用者)至少还是知道算法是什么类型.
换一句话说,"命令"所站的抽象层次比"策略"更高,一个"命令"可能在系统中从一个类传播到另一个类,不断传播(如队列系统,日志系统).但是一个策略一般只会有一个直接的使用者,并不会在系统中不断传播.
这两个是概念上的区别,一个是系统层次,一个是实现方式上.

举个栗子,例如一个遥控器,有7个插槽,每个插槽对应一个电器的开与关.但是,电器的类型目前未知,并且会不断变化.
如果使用策略模式,那么代码会如下所示:
{% highlight java %}
public class RemoteControl {
	private Slot[] slots = new Slot[7];
    
    public void onPush(int index) {
    	slots[index].on();
    }
    public void offPush(int index) {
    	slots[index].off();
    }
}

// 每个电器类型必须实现Slot接口
public class Light implements Slot {
	public void on() {
    	....
    }
    public void off() {
    	....
    }
}
{% endhighlight %}

可以看到,这里命令的接收者必须至少实现Slot接口.现实中,很多情况下我们有的只是一系列已经实现好的类,只是想整合这些类到这个系统中.换句话说,接收者不可能实现特定的接口.
(在队列系统中就是这种情况,我们不能指望每一个接收者都实现这个队列中用到的接口,因为可能接收者类比队列系统更早被编写)

为了让我们的命令发出者与接收者解耦,由上面可以看出接收者不可能迁就发出者(实现相应接口);同样的,我们不希望发出者迁就接收者,因为将来可能会有更多的接收者.

那么,必须提供一个中间的解耦层.

具体的,我们对每种电器的命令实现一个类.如下:
{% highlight java %}
public interface Slot {
	public void execute();
}

public class LightOnCommand implements Slot {
	Light light;
    public LightOnCommand(Light light) { this.light = light; }
	public void execute() {
    	light.on();
    }
}
public class LightOffCommand implements Slot {
	Light light;
    public LightOffCommand(Light light) { this.light = light; }
    public void execute() {
    	light.off();
    }
}
{% endhighlight %}

#### JDK中的命令模式

以下纯属个人看法.
线程框架多少使用了命令模式,发出命令的一方是jvm.jvm并不知道命令的接收者是谁,也不知道接收者线程的具体类型.
那么每个实现了Runnable的类都是一个命令类.


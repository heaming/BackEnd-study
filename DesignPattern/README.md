# 디자인패턴
<br/>

## 목차
1. [Proxy 프록시 패턴](#proxy-------)
2. [싱글톤](#-------)

<br/>
<br/>

## **Proxy 프록시 패턴**

- 실제 비즈니스 로직과 그 외의 코드를 분리하는 객체를 만드는 패턴
- Proxy는 `대리인`이라는 뜻을 가지고 있다. Java에서 RealSubject는 자신의 기능에만 집중을 하고, 그 이외의 `부가 기능 제공, 접근 제어, 로깅, 트랜잭션` 등의 역할을 Proxy 객체에게 위임한다.
- `SRP`를 지향하는 코드를 작성할 수 있다.
- interface 사용시, 구현체로 Proxy를 사용할 수 있다. 이 경우, RealSubject에 직접 접근하지 않고, 한번 우회하여 접근하도록 되어있다. 즉, **흐름제어**가 가능하다.

[![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/374eaf6c-1adb-415d-83dc-6d3e7e4969fc/Untitled.png)](https://www.notion.so/iamheaming/String-Boxing-099aee48a9ff4c95af039f3c45aa7eed#3442e34d832b4fb58aba243b7da481e2)

```java
// interface
public interface Subject {
    String request();
}

// RealSubject
public class RealSubject implements Subject {

    @Override
    public String request() {
        return "HelloWorld";
    }
}

// Proxy
public class Proxy implements Subject {

    private final RealSubject realSubject = new RealSubject();

    @Override
    public String request() {
				System.out.println("Proxy calls RealSubject");
        return realSubject.request();
    }

}

//---
public class Main {
    public static void main(String[] args) {
        // Subject가 아닌, Proxy의 메소드를 호출한다.
        Subject subject = new Proxy();
        System.out.println(subject.request()); 
    }
}
```

- 프록시를 사용하는 이유
    - **코드 변경의 최소화** : 실제 메소드가 호출되기 전에, 필요한 기능(전처리, 보안 등)을 구현 객체 변경 없이 추가할 수 있다.
    - **캐시 사용** : 데이터가 캐시에 존재하지 않는 경우에만 주체 클래스가 실행되도록, 제어할 수 있다.


<br/>

### **Dynamic Proxy**

`런타임`에 특정 인터페이스들을 구현하는 클래스, 인스턴스를 만드는 기술

- Spring Data JPA, Spring AOP, Mockito, Hibernate lazy initialzation

프록시 패턴에는 `직접 구현`, `프록시 클래스 내 중복 발생`이라는 문제가 있다. 다이나믹 프록시는 해당 문제를 해결하기 위한 기술이다.

- java.lang.reflect.Proxy → **newProxyInstance()** & **InvocationHandler**

    ```java
    /* newProxyInstance : 직접 구현 해결
     * new 연산자로 Proxy의 객체를 생성할 수 없다.
     * newProxyInstance() 메소드를 통해서만 생성이 가능하다.
     */
    
    public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) { ... }
    
    /* InvocationHandler : 중복 코드 해결
     * 호출 핸들러
     * invoke() 메소드 하나만 가지는 함수형 인터페이스
     */
    public interface InvocationHandler {
        public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable;
    }
    ```


즉, 다이나믹 프록시 사용시, 클래스마다 프록시 클래스를 만들지 않고, `프록시를 적용할 코드 하나`만 만들어두고, 프록시 객체를 `런타임`시점에만 생성하면 된다.

<br/><br/>

## **싱글톤**

생성 패턴 중 하나로, 인스턴스를 만드는 절차를 추상화하는 패턴이다. 생성 패턴은 객체를 생성, 합성하는 방법이나, 객체의 표현 방법을 시스템과 분리해준다.

싱글톤 패턴은 어떤 클래스의 인**스턴스가 메모리 상에 오직 하나**임을 보장하며, 이 인스턴스에 대해 **어디에서나 접근**할 수 있도록 하는 패턴이다. 즉, 메모리 할당이 최초에만 일어나는 것이다. `DB Connection Pool`, `Logger`, `Thread Pool` , `윈도우 관리자` 처럼 여러 객체를 관리하는 객체는 프로그램 내에서 단 하나의 인스턴스를 갖는 것이 좋다. 이러한 객체를 위해 싱글톤 패턴이 만들어졌다.

하지만 싱글톤 인스턴스가 너무 많은 데이터를 공유할 경우, 인스턴스 간 결합도가 높아져, **OCP를 위배**한다.

- **접근 방법**

  어디에서나 접근할 수 있지만, “전역 변수”와는 다르게 구현하는 것이 좋다. 생성자를 `private` 하게 만들어, 클래스 외부에서는 인스턴스를 생성하지 못하게 차단하고, 내부에서는 단 하나의 인스턴스를 생성해, 접근 방법을 제공하도록 한다.


- **구현**
  - `private` 생성자를 사용해, 외부로부터의 인스턴스 생성을 차단한다.
  - 클래스 내부에 private static 객체 변수를 만든다.
  - `public static` 메서드를 통해, 외부에서 접근할 수 있도록 한다.


- **구현방법**
  1. **Eager Initialization**

     클래스 로딩 단계에서 인스턴스를 생성한다. 하지만 해당 인스턴스를 사용하지 않더라도, 생성되기 때문에 메모리 낭비가 발생할 수 있다. Exception에 대한 Handling 또한 제공하지 않는다.
     File, DB 등 큰 리소스를 다루는 싱글톤을 구현할 때는, `getInstance()`가 호출될 때까지 싱글톤 인스턴스를 생성하지 않는 것이 좋다. 즉, 추천하지 않음

  2. **Static Block Initialization**

     Eager Initialization과 마찬가지로, 클래스 로딩 단계에서 인스턴스를 생성한다. 하지만 Exception Handling에 대한 옵션을 제공한다.

  3. **Lazy Initialization**

     public static getInstance()를 호출할 때, 인스턴스가 없다면 생성한다. 하지만 멀티스레드 환경에 적합하지 않다. 인스턴스가 생성되지 않은 시점에 여러 스레드가 메서드를  호출한다면, 단 하나의 인스턴스를 생성한다는 싱글톤 패턴에 위배된다.

  4. **Thread Safe Singleton**

     Lazy Initialization을 해결하기 위한 방법으로, getInstance()에 synchronized를 걸어두는 방식이다.
     synchronized는 임계 영역(Critical Section)을 형성해, 해당 영역에 오직 하나의 스레드만 접근 가능하게 한다.

     하지만 synchronized 자체에 대한 비용이 크기 때문에, 성능이 떨어질 수 있다. 그를 위해 double checked locking이 등장했다. getInstance() 에 lock을 거는 대신, 인스턴스가 null인 경우에만 synchronized가 동작하도록 하는 방식이다.

  5. **⭐Bill Pugh Singleton Implementation**

     가장 널리 쓰이는 싱글톤 구현 방법이다. inner static helper 클래스를 사용하고 있다. private inner static class를 두어, getInstance() 호출 시에만 인스턴스가 JVM 메모리에 로드된다. 또한, synchronized를 사용하지 않아, 성능 저하 문제도 해결된다.

  6. **Enum Singleton**

     앞선 방식들은 Java의 Reflection을 통해서 파괴될 수 있다. 때문에 Enum으로 싱글톤을 구현하는 방식이 제안 되었다. 하지만 메모리 문제와 유연성 문제가 남아있다.


    ```java
    // 1. Eager Initialization
    public class Singleton {
        
        private static final Singleton instance = new Singleton();
        
        private Singleton(){}
     
        public static Singleton getInstance(){
            return instance;
        }
    }
    
    // 2. Static Block Initialization
    public class Singleton {
     
        private static Singleton instance;
        
        private Singleton(){}
        
        // for exception handling
        static{
            try{
                instance = new Singleton();
            }catch(Exception e){
                throw new RuntimeException("Exception occured in creating singleton instance");
            }
        }
        
        public static Singleton getInstance(){
            return instance;
        }
    }
    
    // 3. Lazy Initialization
    public class Singleton {
     
        private static Singleton instance;
        
        private Singleton(){}
        
        public static Singleton getInstance(){
            if(instance == null){
                instance = new Singleton();
            }
            return instance;
        }
    }
    
    // 4. Thread Safe Singleton
    public class Singleton {
     
        private static Singleton instance;
        
        private Singleton(){}
        
    		/*
        public static synchronized Singleton getInstance(){
            if(instance == null){
                instance = new Singleton();
            }
            return instance;
        }
    	  */  
    	
    		// double checked
    		public static Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class) {
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    	}
    }
    
    // 5. ** Bill Pugh Singleton Implementation **
    public class Singleton {
     
        private Singleton(){}
        
        private static class SingletonHelper{
            private static final Singleton INSTANCE = new Singleton();
        }
        
        public static Singleton getInstance(){
            return SingletonHelper.INSTANCE;
        }
    }
    
    // 6. Enum Singleton
    public enum EnumSingleton {
     
        INSTANCE;
        
        public static void doSomething(){
            //do something
        }
    }
    ```

<br/>
🍃 **Spring과 싱글톤**

싱글톤 패턴에는 문제점들이 있다. 주로, 어플리케이션의 **유연성**을 떨어트리는 단점이다. 하지만 **SPIRNG**에서 대부분의 단점을 해결해주고 있다.

<br/>

> 📌단점
> - 싱글톤 패턴을 구현하는 코드 양이 많아진다.
> - 클라이언트가 구현 클래스에 의존한다. → DIP, OCP 위반
> - 테스트가 어렵다.
> - 내부 속성을 수정, 초기화 하기 어렵다.
> - private 생성자로 자식 클래스를 만들기 어렵다.

<br/>

**스프링 컨테이너**는 `Bean`에 객체를 단 1개 등록함으로써, 객체 인스턴스를 싱글톤으로 관리한다. 이런 스프링의 객체 생성, 관리 기능을 `Singleton Registry`이라고 한다. 하지만 객체 상태를 **stateful 하게 설계하면 절대** 안된다. 객체 인스턴스를 하나만 생성해서 공유하는 상황에서 반드시 주의할 점이다.

<br/>

> 📌싱글톤 주의점
> - 무상태(stateless)로 설계한다.
> - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
> - 공유 필드 대신, local field, parameter, ThreadLocal을 사용한다.

<br/>

- **@Configuration**

  `@Configuration`이 있는 경우, 모든 `@Bean`은 한번씩 호출되지만, 그렇지 않을 경우 @Bean은 호출될 때마다 생성된다. 싱글톤 보장을 위해서, config를 쓴 후, bean을 등록하자!

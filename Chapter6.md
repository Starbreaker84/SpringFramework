### Аспекты ###
Аспект — это объект, перехватывающий вызов метода и выполняющий некую логику до, после или даже вместо выполнения перехваченного метода. **Это позволяет отделить часть кода от бизнес-логики**, благодаря чему приложение становится проще в обслуживании.

Используя аспект, можно написать логику, которая будет выполняться вместе с методом, но размещаться отдельно от него. Таким образом, когда кто-нибудь будет читать код, он увидит только то, что имеет отношение к бизнес-логике приложения.

Аспекты — опасный инструмент. Код, перегруженный ими, лишь усложняет обслуживание приложения. Не используйте аспекты где попало. Принимая решение об их применении, убедитесь, что они действительно улучшат реализацию.

Аспекты лежат в основе многих важных функций Spring, таких как транзакции и методы обеспечения безопасности.

Чтобы создать аспект в Spring, нужно добавить к классу, в котором описана логика аспекта, аннотацию @Aspect. Однако, поскольку Spring должен управлять экземпляром этого класса, нужно также не забыть создать бин этого типа и поместить его в контекст Spring.
```Java
@Aspect
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("execution(* services.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable{
        //какая-то логика
        
        joinPoint.proceed(); // передача управления методу
        
        //какая-то логика
    }
}
```

Чтобы сообщить Spring, какие методы должен перехватывать аспект, используются выражения срезов AspectJ. Эти конструкции служат значениями для аннотаций советов.

![AspectJ example](https://github.com/Starbreaker84/saff/blob/main/AspectJ%20example.png?raw=true)

Создание своей аннотации:
1. Определить аннотацию и сделать ее доступной при выполнении приложения.
```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ToLog{
}
```
```@Retention(RetentionPolicy.RUNTIME)``` - По умолчанию в Java аннотации не могут перехватываться во время выполнения приложения. Нужно конкретно заявить, что нечто может перехватывать аннотации, определив политику хранения как RUNTIME.
```@Target(ElementType.METHOD)``` - @Target определяет, для каких элементов языка может использоваться конкретная аннотация. По умолчанию ею можно сопровождать любые элементы, но всегда лучше ограничить область применения только тем, для чего она создается.

2. С помощью еще одного выражения среза AspectJ сообщить методу аспекта, что нужно перехватывать методы с новой аннотацией.
```Java
@Aspect
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable{
        //код метода
    }
}
```

В Spring есть пять аннотаций советов: @Around, @Before, @After, @AfterThrowing и @AfterReturning. Чаще всего применяется @Around,как наиболее универсальная.
- ```@Before``` — вызывает метод, определяющий логику аспекта, перед выполнением перехватываемого метода;
- ```@AfterReturning``` — вызывает метод, определяющий логику аспекта, после успешного завершения перехватываемого метода. При этом значение, возвращаемое перехваченным методом, передается методу аспекта в качестве параметра. Если перехваченный метод выдаст исключение, метод аспекта не вызывается;
- ```@AfterThrowing``` — вызывает метод, определяющий логику аспекта, если перехваченный метод выдает исключение. Экземпляр исключения передается методу аспекта в качестве параметра;
- ```@After``` — вызывает метод, определяющий логику аспекта, после выполнения перехватываемого метода, независимо от того, завершится ли перехваченный метод успешно или выдаст исключение.

Вызов одного и того же метода может перехватываться несколькими аспектами. В данном случае рекомендуется задавать последовательность выполнения аспектов с помощью аннотации ```@Order(n)``` - где n - число, которое чем ниже, тем предполагает более ранний порядок выполнения.

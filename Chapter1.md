**Контекст** Spring - область памяти, выделяемая для бинов Spring'а. Имеенно оттуда Spring осуществляет управление и интеграцию с программой.
**Бины** - экземляры объектов, помещаемые в контекст Spring'а. Бинами могут быть экземпляры классов, создаваемых программистом, так и библиотечные.
Spring «видит» только те экземпляры, которые были добавлены в контекст.
В реальных приложениях далеко не все объекты включаются в контекст Spring.

Создание контекста:
```Java
AnnotationConfigApplicationContext() context = new AnnotationConfigApplicationContext(ProjectConfig.class);
```
Получение ссылки на бин из контекста:
```Java
Parrot parrot = context.getBean("rico", Parrot.class);
```

Есть три способа добавления бинов в контекст Spring: с помощью аннотации @Bean, стереотипных аннотаций, а также программно.

1. Аннотация @Bean (применяют для создания бинов из классов библиотек - больше кода):
```Java
@Configuration
public class ProjectConfig(){

    @Bean(name/value = "myparrot")
    Parrot parrot(){ //ВНИМАНИЕ! Название метода - имя существительное (повторяет класс)
        Parrot p = new Parrot();
        p.setName("Rico");
        return p;
    }
    
    @Bean
    @Primary //Сделает бин первым вызываемым этого класса (если вызывается не один бин того же класса)
    Integer ten(){
        return 10;
    }
    
    @Bean
    String hello(){
        return "Hello!";
    }
}
```

2. Стереотипные аннотации(применяют к классам приложения - меньше кода):
```Java
@Configuration
@ComponentScan(basePackages = "main") //Указываем, где хранятся нужные классы
public class ProjectConfig(){
}

@Component //Указываем, что нужно создать бин по этому классу
public class Parrot{
    private String name;
    
    @PostConstruct //init-метод, но нужна зависимость Javax.annotation-api
    public void init() {
        this.name = "Kiki";
    }
    //Код клаасса
}
```
@PreDestroy - по-возможности не используем (завершение соединения с базой данных, которого не было)

3. Программное создание (позволяет реализовать собственную логику добавления бинов в контекст Spring):
```Java
@Configuration
public class ProjectConfig {
}

Parrot x = new Parrot();
x.setName("Kiki");

Supplier<Parrot> parrotSupplier = () -> x;

context.registerBean("parrot1",
    Parrot.class,
    parrotSupplier,
    bdc -> bdc.setPrimary(true));	
```

**Supplier** — это функциональный интерфейс, который входит в пакет Java.util.function. Назначение реализации Supplier состоит в том, чтобы
возвращать заданное значение, не принимая параметров.

**BeanDefinitionCustomizer** — просто еще один интерфейс, который используется для настройки различных свойств бина, например, чтобы сделать бин первичным.

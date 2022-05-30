# Задача "Понимание JVM"

## Код для исследования
```java

public class JvmComprehension {

    public static void main(String[] args) {
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }

    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6
    }
}

```
---
- ClassLoader'ы: 
---
Загрузчик классов анализируя файлы увидит класс JvmComprehension и с помощью Class Loading загрузит его. Application
ClassLoader загрузит класс JvmComprehension. Далее Platform ClassLoader загрузит платформенно специфичные классы. 
Далее Bootstrap ClassLoader самые базовые классы. Application ClassLoader перед загрузкой уточнит у Platform ClassLoader
нет ли у него класса JvmComprehension,в свою очередь Platform ClassLoader уточнит у Bootstrap ClassLoader
нет ли у него класса JvmComprehension убедившись в этом загрузит его. Если Bootstrap ClassLoader при проверке найдет 
класс JvmComprehension то он загрузит его, если нет то сообщит об этом Platform ClassLoader. Если Platform ClassLoader
найдет класс JvmComprehension то он загрузит его, если нет то сообщит об этом Application ClassLoader. 
Application ClassLoader загрузит класс JvmComprehension при условии отсутствия своих ClassLoader-ов, при наличии таковых
цепочка начнется с них.
 
После идет блок связывание(Linking):

- Verify - проверка валидности кода;
- Prepare - подготовка примитивов в статических полях;
- Resolve - связывание ссылок на другие классы.

После идет блок инициализации(Initialization). 
Выполняются все статические методы, в данном примере таких нет.

Класс JvmComprehension загружается в Metaspace.

---
- области памяти (стэк (и его фреймы), хип, метаспейс) : 
---
0. В Stack Memory добавилась область main
1. Переменная i добавляется в Stack Memory в область main
2. Создан объект Object в heap(куча) и была создана переменная o, 
   ссылка добавлена в Stack Memory в область main
3. Переменная ii добавляется в Stack Memory в область main
4. В Stack Memory добавилась область printAll и в этой области создана ссылка на переменную o и переданы переменные i, ii
5. Создан объект Integer в heap(куча) и была создана переменная uselessVar, 
   ссылка добавлена в Stack Memory в область printAll
6. В Stack Memory добавилась область println, в heap(куча) создан объект String 
   и создана ссылка в Stack Memory в область println
7. В Stack Memory добавилась область println, в heap(куча) создан объект String 
   и создана ссылка в Stack Memory в область println
   
---   
- сборщик мусора: 
---  
Сначала сборщик проверит количество ссылок на объеты и в методе после передачи переменной i удалит ее
за ненадобностью, далее в методе printAll после его выполнения удаляться объекты Integer(uselessVar) и String, 
созданный после конкатенации строк, а также Object o и Integer ii так как далее они не используются.
После очиситься объект String созданный в методе main. Время жизни у Object o и Integer ii будет наибольшее, 
так как на них ссылки в двух областях в Stack Memory.
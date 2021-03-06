# Понимание JVM

```java
public class JvmComprehension {

    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6
    }

    public static void main(String[] args) {
        int i = 1;                      // 1
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }
}
```

В тот момент когда _JVM_ запрашивает класс *JvmComprehension*, загрузчик классов _ClassLoader_ пытается найти данный
класс и загрузить определение класса в среду выполнения. Вначале работы программы создается 3 основных загрузчика
классов:

1. базовый загрузчик,
1. загрузчик расширений,
1. системный загрузчик.

Системный загрузчик проверяет, не загружался ли данный класс ранее. Если класс уже был загружен и загрузчик знает о его
местонахождении, то будет возвращен объект _Class_ этого класса из кэша. Если нет, системный загрузчик делегирует поиск
класса родительскому классу-загрузчику. Поиск будет продолжаться до тех пор, пока не дойдет до базового загрузчика. Если
и в базовом загрузчике нет информации об искомом классе, будет выполнен поиск этого класса по расположению классов, о
котором знает данный загрузчик, и, если загрузить класс не удастся, управление вернется обратно загрузчику-потомку,
который будет пытаться выполнить загрузку из известных ему источников. Загруженный класс *JvmComprehension* помещается в
особую область памяти, называемой _Metaspace_.

До начала выполнения метода *main()*, в стеке будет выделено пространство для хранения примитивов и ссылок этого метода:

* примитивное значение _i (1)_ типа _int_ будет храниться непосредственно в стеке со значением _1_;
* ссылочная переменная _o (2)_ типа _Object_ будет создана в стеке, но сам объект будет храниться в куче;
* ссылочная переменная _ii (3)_ типа _Integer_ будет создана в стеке, а сам объект будет храниться в куче.

В методе *main()* дополнительно вызывается метод *printAll() (4)*, для которого будет выделен блок памяти в стеке поверх
предыдущего вызова. Этот блок выделит пространство для хранения примитивов и ссылок этого метода:

* ссылочная переменная _o_ типа _Object_ будет создана в стеке и ссылаться на объект, уже хранящийся в куче;
* примитивное значение _i_ типа _int_ будет храниться непосредственно в стеке со значением _1_;
* ссылочная переменная _ii_ типа Integer будет создана в стеке и ссылаться на объект, уже хранящийся в куче;
* ссылочная переменная _uselessVar (5)_ типа _Integer_ будет создана в стеке, но сам объект будет храниться в куче.

В методе *printAll()* дополнительно вызывается метод *toString()*, для которого будет выделен новый блок памяти в стеке
поверх предыдущего вызова. Этот блок выделит пространство для хранения ссылочной переменной, но сама строка будет
храниться в *String Pool*, которая является частью кучи. Как только метод заканчивает работу, блок памяти также
перестает использоваться, тем самым предоставляя доступ к следующему методу, т.е. *printAll()*.

Конкатенация строк в блоке памяти *printAll()* выделяет пространство для хранения ссылочной переменной типа
*StringBuilder*, которая указывает на объект, расположенный в куче. Поверх *printAll()* в стеке выделяется новый блок
памяти, содержащий примитивы и ссылки на другие объекты, необходимые для работы метода *append()* при добавлении
результата _o.toString()_ к объекту StringBuilder. После завершения метода *append()* поверх *printAll()* в стеке снова
выделяется блок памяти, необходимый для работы уже метода *toString()* для получения объекта типа *String* с объекта
*StringBuilder*. И уже после завершения метода *toString()* аналогичный процесс наблюдается при работе следующей
конкатенации.

В методе *printAll()* дополнительно вызывается метод *println() (6)*, для которого будет выделен новый блок памяти в
стеке поверх предыдущего вызова. После выполнения метода *println()* метод *printAll()* завершает свою работу,
предоставляя доступ методу *main()*. Поверх метода *main()* в стеке создается новый фрейм для работы *println() (7)*.
Стековая память автоматически освобождается, когда методы завершают свою работу. Память в куче освобождается сборщиком
мусора, когда в приложении не остается ни одной ссылки на объект или объект является недостижимым.

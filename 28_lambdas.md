# 14. Лямба-функции и элементы функционального программирования

27 56:00
## Базовый синтаксис

Возникла задача отсортировать массив с помощью нетривиальной функции порядка. Например, по удаленности от некоторого числа. Допотопные технологии (до С++11) предлагают действовать вот так

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct Compare {
    bool operator()(int x, int y) const {
        return std::abs(x - 5) < std::abs(y - 5); 
    }
};

int main() {
    std::vector<int> v{4, 3, 6, 2, 8, 5, 7, 1, 4, 2};
    std::sort(v.begin(), v.end(), Compare());

    for (auto x: v) {
        std::cout << x << " ";
    }
}
```

Но начиная с C++11 появился более выразительный синтаксис лямбда-функций. Появилась возможность создавать функции непосредственно по месту использования. В такой ситуации им даже имя не требуется. 

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v{4, 3, 6, 2, 8, 5, 7, 1, 4, 2};
    std::sort(v.begin(), v.end(), [](int x, int y) {
        return std::abs(x - 5) < std::abs(y - 5); 
    });

    for (auto x: v) {
        std::cout << x << " ";
    }
}
```

Вопрос: а если захотелось такую функцию сохранить, а потом переиспользовать, как это сделать?

<details><summary>code snippet</summary>
<p>
Достаточно создать переменную и присвоить ей выражение, описывающее лямбда-функцию

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v{4, 3, 6, 2, 8, 5, 7, 1, 4, 2};
    
    auto f = [](int x, int y) {
        return std::abs(x - 5) < std::abs(y - 5); 
    };

    std::sort(v.begin(), v.end(), f);
    for (auto x: v) {
        std::cout << x << " ";
    }
}
```
</p>
</details>

Механизм лямбда-функций не добавляет новых возможностей к языку, но существенно повышает удобство работы с функциями как самостоятельными объектами.

Вопрос: как вернуть лямбда-функцию в качестве результата исполнения обыкновенной функции?

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

auto getCompare() {
    return [](int x, int y) {
        return std::abs(x - 5) < std::abs(y - 5); 
    };
}

int main() {
    std::vector<int> v{4, 3, 6, 2, 8, 5, 7, 1, 4, 2};

    std::sort(v.begin(), v.end(), getCompare());
    for (auto x: v) {
        std::cout << x << " ";
    }
}
```
</p>
</details>


Вопрос: корректен ли следующий код?
```cpp
#include <iostream>

int main() {
    [](int x) {  
        std::cout << x << std::endl;
    }(5)         
}
```

<details><summary>Ответ</summary>
Да, код корректный. Это просто объявление лямбда функции и её вызов. Более того, корретна даже вот такая запись 

```cpp
[](){}();
```

это вызов лямбда-функции, которая не принимает параметров и ничего не делает.
</details>

Можно самостоятельно объявить возвращаемый тип, как для удобства чтения, так и в качестве помощи компилятору

```cpp
auto getCompare() {
    return [](int x, int y) -> bool {
        if (x == y) 
            return 0;
        else
            return std::abs(x - 5) < std::abs(y - 5); 
    };
}
```

## Захват контекста

Как действовать в ситуации, когда лямбда-функции помимо параметров потребовались какие-то внешние переменные?

```cpp
#include <iostream>

int main() {
    a = 1;
    
    [](int x) {  
        std::cout << a + x << std::endl;
    }(5)         
}
```
Такие переменные должны быть предварительно захвачены. Список захватываемых переменных нужно указать в квадратных скобках при объявлении лямбда-функции.

```cpp
#include <iostream>

int main() {
    a = 1;
    
    [a](int x) {  
        std::cout << a + x << std::endl;
    }(5)         
}
```

28 00:00

Вопрос: если попробовать изменить захваченную переменную а, что произойдет?

```cpp
#include <iostream>

int main() {
    a = 1;
    
    [a](int x) {  
        ++a;
        std::cout << a + x << std::endl;
    }(5)         
}
```

<details><summary>Ответ</summary>
Код даже не скомпилируется. С точки зрения лямбда функции, захваченная таким образом переменная является константой. Чтобы изменять переменные, придется добавить слово mutable

```cpp 
#include <iostream>

int main() {
    a = 1;
    
    [a](int x) mutable {  
        ++a;
        std::cout << a + x << std::endl;
    }(5)         
}
```

При этом переменные, изменяемые внутри лямбда функции, будут скопированы по значению.

```cpp 
#include <iostream>

int main() {
    a = 1;
    
    [a](int x) mutable {  
        ++a; // теперь это локальная переменная лямбда-функции
        std::cout << a + x << std::endl;
    }(5)

    std::cout << a << std::endl;
}
```

</details>

Другой способ захвата переменных &ndash; по ссылке (синтаксис при этом выглядит странновато)

```cpp 
#include <iostream>

int main() {
    a = 1;
    
    [&a](int x) mutable {  
        ++a;
        std::cout << a + x << std::endl;
    }(5)
    std::cout << a << std::endl;
}
```

Зачем вообще нужны списки захвата, почему бы просто не передавать больше параметров? Дело в том, что лямбда-функции часто применяются там, где набор аргументов уже загреплен (например, компаратор в сортировке и т.п.), поэтому для передачи дополнительных параметров приходится использовать такой хак.

Вопрос: что произойдет, если лямбда-функция живёт дольше, чем захваченная по ссылке переменная? 

<details><summary>Ответ</summary> UB </details>

## Лямбда-функции как объекты

Попробуем посмотреть, как выгядит тип лямбда-функции

```cpp 
#include <iostream>

template <typename T>
void g(const T&) = delete;

int main() {
    auto f = [](int x, int y) {
        return x < y;
    };

    g(f);
}
```

компилятор подкидывает заглушку и не признается. Другой способ

```cpp 
#include <iostream>

int main() {
    auto f = [](int x, int y) {
        return x < y;
    };

    std::cout << typeid(f).name();
}
```

компилятор выдает снова странное название. Но тем не менее лямбда-функция имеет внутреннее наименование типа, которое не обязано быть человекочитаемым. Каков размер 

```cpp 
#include <iostream>

int main() {
    auto f = [](int x, int y) {
        return x < y;
    };

    std::cout << sizeof(f) << std::endl;
}
```

Размер: 1 байт. Это пустой объект ненулевого размера.

Вопрос: а как изменится размер при захвате переменной? (см. код)

```cpp 
#include <iostream>

int main() {
    int a = 1;
    auto f = [a](int x, int y) {
        return x < y;
    };

    std::cout << sizeof(f) << std::endl;
}
```

<details><summary>Ответ</summary> 
Теперь размер 4 байта. Дело в том, что лямбда-функция &ndash; это объект, у которого определен константный оператор скобки "()", а все захваченные переменные превращаются в поля объекта, инициализированные захваченными значениями.
</details>

Для лямбда-функций комплятор создает конструктор копирования и перемещения.

```cpp 
#include <iostream>

int main() {
    std::string s = "abc";

    auto f = [s](int x, int y) {
        std::cout << s.size() << std::endl;
        return x < y;
    };

    auto ff = f;

    std::cout << sizeof(f) << std::endl;
}
```

```cpp 
#include <iostream>

int main() {
    std::string s = "abc";

    auto f = [s](int x, int y) {
        std::cout << s.size() << std::endl;
        return x < y;
    };

    auto ff = std::move(f);

    std::cout << sizeof(f) << std::endl;
}
```

Вопрос: будет ли работать оператор присваивания? 

<details><summary>Ответ</summary> 
Не будет

```cpp 
#include <iostream>

int main() {
    std::string s = "abc";

    auto f = [s](int x, int y) {
        std::cout << s.size() << std::endl;
        return x < y;
    };

    auto ff = std::move(f);

    f = ff; // ошибка
}
```
</details>

36:40
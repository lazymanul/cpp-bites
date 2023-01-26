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

Вопрос: а если захотелось такую функцию сохранить, а потом переиспользовать, можно ли это сделать?

<details><summary>code snippet</summary>
<p>
Можно, достаточно создать переменную и присвоить ей выражение, описывающее лямбда-функцию

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
Да, код корректный. Это просто объявление лямбда функции и её вызов. Более того, можно корретна даже вот такая запись 

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
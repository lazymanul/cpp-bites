# Шаблонное метапрограммирование и SFINAE

Будем считать, что метапрограммирование занимается программированием над типами. То есть аргументами метафункции являются не объекты, а типы. В предыдущих сериях уже встречались некоторые метафункции, например, при реализации std::move понадобилась функция std::remove_reference_t.

Прежде чем погрузиться в метапрограмирование, рассмотрим одну из особенностей шаблонного вывода

SFINAE = Substitution Failure Is Not An Error (неудачная шаблонная подстановка &ndash; не ошибка компиляции)

Вопрос для разминки: какая из функций будет вызвана?

```cpp
template <typename T>
void f(const T&) {
    std::cout << 1;
}

void f(...) {
    std::cout << 2; }


int main() {
    f(5);
}
```

<details><summary>Ответ</summary> 
Первая, потому что она более специализирована, а для компилятора "частное лучше общего".
</details>

Рассмотрим более изощренный пример. Вопрос прежний: какая из функций будет вызвана?

```cpp
#include <iostream>
#include <vector>

template <typename T>
auto f(const T&) -> decltype(T().size()) {
    std::cout << 1;
    return 0;
}

int f(...) {
    std::cout << 2; 
    return 0;
}

int main() {
    std::vector<int> v{1, 2, 3};
    f(v);
}
```
<details><summary>Ответ</summary> 
Первая по прежней причине: она более специализирована, а для компилятора "частное лучше общего".
</details>

А что же произойдет теперь?

```cpp
#include <iostream>
#include <vector>

template <typename T>
auto f(const T&) -> decltype(T().size()) {
    std::cout << 1;
    return 0;
}

int f(...) {
    std::cout << 2; 
    return 0;
}

int main() {    
    f(5); 
}
```
<details><summary>Ответ</summary> 
Вторая. Сперва компилятор действительно попытается воспользовать первым вариантом, но при вычислении  decltype(T().size()) поймет, что у int нет метода size. Подстановка не срабатывает и отбрасывается, а компилятор продолжает искать лучший вариант среди оставшихся. Это сработал принцип SFINAE.
</details>

Чтобы разобраться, когда принцип SFINAE применим, рассмотрим модифцированный пример. Удастся ли отфильтровать неподходящую функцию в этот раз?

```cpp
#include <iostream>
#include <vector>

template <typename T>
auto f(const T&) {
    T x; 
    x.size();
    std::cout << 1;
    return 0;
}

int f(...) {
    std::cout << 2; 
    return 0;
}

int main() {    
    f(5); 
}
```

<details><summary>Ответ</summary> 
Нет, возникнет CE. В данном случае по сигнатуре нельзя понять, что функция не подходит, а в теле функции используются недопустимые методы для типа T. Принцип SFINAE срабатывает в момент выбора версии функции (на этапе анализа сигнатур). Когда компилятор уже выбрал фунцию и понял, что она не подходит, возникает ошибка компиляции.
</details>

Несмотря на свою простоту, правило SFINAE значительно расширяет наши возможности, позволяя реализовывать шаблонную магию вручную (так делать обычно не стоит, но сам факт приятен). 

## std::enable_if, его использование и реализация

std::enable_if позволяет включать/выключать некоторые версии перегрузки в зависимости от условий, проверяемых на этапе компиляции.


Возможность выстрелить себе в ногу (или особенности склейки ссылок)

```cpp
#include <iostream>
#include <vector>
#include <type_traits>

// более предпочтительная версия для компилятора
template <typename T, typename = std::enable_if_t<!std::is_class_v<T>>>
void g(T&&) {
    std::cout << 2;
}

template <typename T, typename = std::enable_if_t<!std::is_class_v<T>>>
void g(const T&) {
    std::cout << 1;
}

int main() {
    std::vector<int> v{1, 2, 3};
    g(v); // оба раза вызывается первая фунция          
    g(5);
}

```

правильный вариант:

```cpp
#include <iostream>
#include <vector>
#include <type_traits>

// более предпочтительная версия для компилятора
template <typename T, typename = std::enable_if_t<!std::is_class_v<T>>>
void g(T&&) {
    std::cout << 2;
}

template <typename T, typename = std::enable_if_t<std::is_class_v<
                                    std::remove_reference_t<T> 
                                    > >>
void g(const T&) {
    std::cout << 1;
}

int main() {
    std::vector<int> v{1, 2, 3};
    g(v); // но с помощью шаблонных условий для классов 
          // будет выбран второй вариант функции
    g(5);
}
```

Рассмотрим как устроен enable_if изнутри
(30:00)

```cpp
template <bool B, typename T = void>
struct enable_if {};

template <typename T>
struct enable_if<true, T> {
    using type = T;
}

template <bool B, typename T = void>
using enable_if_t = typename enable_if<B, T>::type;

```

## Проверка наличия методов в классе

Рассмотрим, каким образом можно проверить тип передаваемого аллокатор или итератора (спойлер: придется проверить наличие каждого метода в классе).

```cpp
#include <iostream>

template <typename T, typename... Args>
struct has_method_construct {
private:

public:
    static const bool value = ...;

};

template <typename T, typename... Args>
const bool has_method_construct_v = has_method_construct<T, Args...>::value;

struct S {
    void construct(int) {}
    void construct(int, double) {}
};

int main() {
    std::cout << has_method_construct_v<S, int> << std::endl;
    std::cout << has_method_construct_v<S, int, int> << std::endl;
    std::cout << has_method_construct_v<S, int, int, int> << std::endl;
}
```

45:00
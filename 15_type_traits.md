
## Проблема зависимых имен (dependent names)

Возможность специализации шаблонов влечет интересные следствия. Рассмотрим пример:

```cpp
#include <iostream>

template <typename T> 
struct S { // Шаблонная структура
    using X = T;
}

template <>
struct S<int> { // её специализация
    static int X;
}

int a = 0;

template <typename T>
void f() {
    S<T>::X * a; // строка, заслуживающая пристального внимания
}

int main() {}
```

В чем именно проблема строки? 
```cpp
    S<T>::X * a;
```

Ответ: в зависимости от T строка будет обладать различным синтаксическим смыслом. Она может оказаться как определением, так и выражением! Если T &ndash; это int, то запись следует интерпретировать как умножение. В остальных случаях получаем объявление указателя a. Синтаксическое значение выражения нельзя определить до выполнения шаблонной подстановки. Возникает зависимое имя X. 


### Определение
_Зависмым_ называется имя внутри шаблонного определения (класса или функции), смысл которого становится понятным после выполнения шаблонной подстановки.

https://en.cppreference.com/w/cpp/language/dependent_name


Чтобы подсказать компилятору, какой выбор правильный, можно использовать слово typename

```cpp
#include <iostream>

template <typename T> 
struct S { // Шаблонная структура
    using X = T;
}

template <>
struct S<int> { // её специализация
    static int X;
}

template <typename T>
void f() {
    typename S<T>::X * a; // подсказка для компилятора
}

int main() {
    f<int>(); 
}
```

Приведенный в предыдущем примере код будет компилироваться. Но как насчет вот такого варианта?

```cpp
#include <iostream>

template <typename T> 
struct S { // Шаблонная структура
    using X = T;
}

template <>
struct S<int> { // её специализация
    static int X;
}

template <typename T>
void f() {
    typename S<T>::X * a; // подсказка для компилятора
}

int main() {
    f<int>(); 
    f<double>();
}
```

Ответ: не скомпилируется. Двусмысленность возникает в функции main. Первый вызов с параметром int воспринимается однозначно как определение. Но во втором вызове необходимо категория выражения изменяется, компилятор считает такое поведение ошибкой и выбрасывает CE.

```cpp
#include <iostream>

template <typename T> 
struct S { // Шаблонная структура
    using X = T;
}

template <>
struct S<int> { // её специализация
    static int X;
}

template <typename T>
void f() {
    S<T>::X a; 
}

int main() {}
```
код не скомпилируется, потому что возникает существует потенциальная неоднозначность. Компилятор, встречая зависимое имя, по-умолчанию считает его именем переменной, если явно не сказано обратное. Компилятор не делает преподоложений о том, как можно было трактовать строку кода.


<span style="background-color: yellow">Caution: проверить логику примеров.</span>
 
Другая загадка

```cpp
#include <iostream>

template <typename T>
struct SS { // структура, внутри которой другая структура
    template <int M, int N>
    struct A {};
}

template <>
struct SS<int> {
    static const int A = 0;
}

int a = 0;

template <typename T>
void g() {
    SS<T>::A<1, 2> a; // строка, заслуживающая пристального внимания
}

int main() {}
```

С одной стороны строка 
```cpp
SS<T>::A<1,2> a;
``` 
действительно выглядит как объявление. Но с другой с другой это 2 выражения сравнения, записанные через запятую!
```cpp
SS<T>::A < 1,  2 > a;
```
Компилятор снова испытывает серьезные затруднения. Следующий пример не скомпилируется.

```cpp
#include <iostream>

template <typename T>
struct SS { // структура, внутри которой другая структура
    template <int M, int N>
    struct A {};
}

template <>
struct SS<int> {
    static const int A = 0;
}

template <typename T>
void g() {
    SS<T>::A<1, 2> a; // строка, заслуживающая пристального внимания
}

int main() {}
```
Но даже если добавить подсказку typename, ошибка не будет устранена.

```cpp
#include <iostream>

template <typename T>
struct SS { // структура, внутри которой другая структура
    template <int M, int N>
    struct A {};
}

template <>
struct SS<int> {
    static const int A = 0;
}

template <typename T>
void g() {
    typename SS<T>::A<1, 2> a; // подсказка есть, но проблема не исчезла
}

int main() {}
```

Дело в том, что подсказка помогает понять, что начинается определение типа,  но не помогает справиться со второй открывающей угловой скобкой (т.е. понять, что есть ещё один внутренний шаблон). Нужно использовать ещё одну подсказку

```cpp
#include <iostream>

template <typename T>
struct SS { // структура, внутри которой другая структура
    template <int M, int N>
    struct A {};
}

template <>
struct SS<int> {
    static const int A = 0;
}

template <typename T>
void g() {
    typename SS<T>::template A<1, 2> a; // две подсказки в одной строке
}

int main() {}
```

Примечание: стиль кода, когда переменные и типы называются одинаково плох, не стоит так делать. 


## Базовые type traits

Допустим, что мы хотим написать метафункцию, зависящую от переданного типа. 

```cpp
template <typename U, typename V> 
void f(U x, V y) {
    // ...

    if ("U == V") { // здесь записано желание сравнить два типа
        // ...
    } else {
        // ...
    }
    // ...
}
```

Какие инструменты предоставляет С++ для решения такой задачи? Если речь идет о времени компиляции, то мы можем использовать шаблоны и специализацию.

```cpp
#include <iostream>

template <typename U, typename V> // базовая версия, когда типы не равны
struct is_same {
    static const bool value = false;
};

template <typename U> // частично специализированная версия для одинаковых типов
struct is_same<U, U> {
    static const bool value = true;
};

template <typename U, typename V> 
void f(U x, V y) {
    // ...
    if (is_same<U, V>::value) { // желание сравнивать типы обретает форму
        // ...
    } else {
        // ...
    }
    // ...
}
```

36:17
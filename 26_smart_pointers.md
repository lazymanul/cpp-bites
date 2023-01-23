# Умные указатели

since 14:00

## Inrto

Умный указатель &ndash; инструмент, позволяющий автоматически освобождать динамически выделенные ресурсы. 

Мотивационная проблема: программой выделена память, но случилась нештатная ситуация, вылетело исключение до того, как память была освобождена. Произошла утечка.

```cpp
#include <iostream>

void f() {
    int *p = new int(5);

    if (x == 0) 
        throw  std::runtime_error("Ooops!");

    delete p;
}

int main() {
    f(8);
}
```

Как можно этого избежать?

```cpp
#include <iostream>

void f() {
    // RAII
    std::shared_ptr<int> p(new int(5));

    if (x == 0) 
        throw  std::runtime_error("Ooops!");
}

int main() {
    f(8);
}
```

std::shared_ptr &ndash; объект, хранящий указатель на выделенный ресурс, а также счетчик пользователей этого ресурса: он увеличивается, когда появлялются новые пользователи и уменьшается, когда пользователи удаляются. Освобождение ресурса произойдет, когда будет удален последний std::shared_ptr.

```cpp
#include <iostream>

void f(int) {
    // RAII
    std::shared_ptr<int> p(new int(5));
    auto pp = p;
}

int main() {
    f(8);
}
```

### std::unique_ptr

```cpp
#include <iostream>
#include <memory>

void f(int x) {
    // RAII
    std::unique_ptr<int> p(new int(x));

    if (x == 0) 
        throw  std::runtime_error("Ooops!");
}

int main() {
    f(8);
}
```

скопировать std::unique_ptr нельзя, у него удален конструктор копирования и оператор присваивания

```cpp
#include <iostream>
#include <memory>

void f() {
    // RAII
    std::unique_ptr<int> p(new int(5));
    auto pp = p; // не сработает
}

int main() {
    f(8);
}
```

умный указатель несет отвественность только за тот ресурс для которого он был создан. Если самостоятельно создать несколько умных указателй на ресурс, то при освобождении каждый из них освободит ресурс (и это приведет к проблемам).

Можно ли сделать вектор std::unique_ptr?

```cpp
#include <iostream>
#include <memory>
#include <vector>

void f() {
    std::vector<std::unique_ptr<int>> v;    
    for (int i = 0; i < 10; ++i)
        v.emplace_back(new int(i));    
}

int main() {    
    f(8);    
}
```

Можно, всё будет работать. Тип std::unique_ptr нельзя копировать, но можно перемещать. 

std::unique_ptr можно разыменовывать

```cpp
#include <iostream>
#include <memory>
#include <vector>

void f(int) {
    std::vector<std::unique_ptr<int>> v;    
    for (int i = 0; i < 10; ++i)
        v.emplace_back(new int(i));

    for (int i = 0; i < 10; ++i)
        std::cout << *v[i] << ' ';
}

int main() {    
    f(8);    
}
```

Как использовать std::unique_ptr для массивов?

<details><summary>code snippet</summary>
<p>
Воспользоваться специализацией, но вообще смысла в подобных операциях немного

```cpp
#include <iostream>
#include <memory>

void f(int x) {
    // RAII
    std::unique_ptr<int[]> p(new int[x]);
}

int main() {
    f(8);
}
```
</p>
</details>

Как можно перемещать std::unique_ptr? 

<details><summary>code snippet</summary>
<p>
Стандартным образом с помощью std::move

```cpp
#include <iostream>
#include <memory>

void f(int x) {
    // RAII
    std::unique_ptr<int> p(new int(x));
    auto pp = std::move(p)
}

int main() {
    f(8);
}
```
</p>
</details>


### std::shared_ptr

std::shared_ptr &ndash; объект, хранящий указатель на выделенный ресурс, а также счетчик пользователей этого ресурса: он увеличивается, когда появлялются новые пользователи и уменьшается, когда пользователи удаляются. Освобождение ресурса произойдет, когда будет удален последний std::shared_ptr.


```cpp
#include <iostream>

template <typename T>
class shared_ptr {
    private:
        T* ptr = nullptr;
        size_t* count = nullptr;
    public:
        explicit share_ptr(T* ptr): ptr(ptr), count(new int(1)) {}

        ~shared_ptr() {
            if (!count) return;
            --*count;
            if (!*count) {
                delete ptr;
                delete count;
            }
        }
};
```

Как проверить std::shared_ptr на пустоту?

## make_shared, make_unique

Опасный код: одновременно существует c-style код и умный указатель.

```cpp
#include <memory>

void f(int x) {
    int* p = new int(5);
    std::shared_ptr<int> sp(p);

}
```

Но на самом деле даже вот так делать не очень хорошо. 

```cpp
#include <memory>

void f(int x) {    
    std::shared_ptr<int> sp(new int(5));

}
```

Практика использования современного C++ предполагает максимальный отказ от использования оператора new. Исключением являлются низкоуровневые операции.

Причина

```cpp
#include <memory>

void g(const shared_ptr<int>& p, int x) {}

int f(x) {
    if (x == 0) 
        throw std::runtime_error("abcde");
    return x + 1;
}

int h() {
    g(shared_ptr<int>(new int(5)), f(0));
}
```

аргументы g не вычисляются последовательно (до C++17), поэтому может произойти исключение в неподходящий момент.

```cpp
template <typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T>(new T(std::forward<Args>(args)...));
}

int h() {
    auto p = std::make_unique<int>(5);
}
```

1:20:00

```cpp
template <typename U>
struct ControlBlock {
    size_t count;
    T object;    

    template <typename... Args>
    ControlBlock(size_T count, Args&&... args);
}

template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args) {
    auto p = new ControlBlock<T>(1, std::forward<Args>(args)...)
    return shared_ptr<T>(p); //приватный конструктор
}

int h() {
    auto p = std::make_shared<int>(5);
}
```
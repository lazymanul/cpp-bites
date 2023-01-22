# Исключения

Исключения в том или ином виде реализованых во многих современных языках, но у C++ как всегда есть свои особенности. 

Базовая идея: когда вы пишете код у вас что-то может пойти не так. Например, программа может в ходе выполнения может сделать недопустимую операцию (поделить на ноль, выйти за границу массива) и упасть. В большинстве случаев это не желательно, как минимум хочется успеть собрать информацию о том, что именно и где пошло не так.



## Основная идея

<details><summary>code snippet</summary>
<p>

```c
int f(int x, int y, int* result) {
    std::cout << 1;
}

int main() {
    f(1, 0);
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

int main() {
    f(1, 0);
}
```

```bash
terminate called after throwing an instance of 'int'
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

int main() {
    int x = 0; 
    int y = 1;
    f(y / x, 0);
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

int main() {
    try {
        f(1, 0);
    } catch (int x) {
        std::cout << "Exception!\n";
    }
}
```
Output:

```bash
Exception!
```
</p>
</details>



<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

int f(int x, int y) {
    y == 0, throw 1;
    return x / y;
}

int main() {
    try {
        f(1, 0);
    } catch (int x) {
        std::cout << "Exception!\n";
    }
}
```
</p>
</details>

## Разница между исключениями и ошибками времени исполнения

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

int main() {
    std::vector<int> v(10);

    v[10000] = 1;
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

int main() {
    std::vector<int> v(10);
    
    try {
        v[10000] = 1;
    } catch (...) {
        std::cout << "Exception!\n";
    }
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

// std::out_of_range

int main() {
    std::vector<int> v(10);
    
    try {
        v.at(10000) = 1;
    } catch (...) {
        std::cout << "Exception!\n";
    }
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int main() {
    std::bad_cast;
    dynamic_cast<> s;
}
```
</p>
</details>

## std::exception

cppreference.com std::exception

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

// std::out_of_range

int main() {
    std::vector<int> v(10);
    
    try {
        v.at(10000) = 1;
    } catch (std::exception& ex) {
        std::cout << ex.what();
    }
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>
#include <vector>

int f(int x, int y) {
    if (y == 0)
        throw 1;
    return x / y;
}

// std::out_of_range

int main() {
    std::vector<int> v(10);
    
    try {
        throw std::out_of_range("Uncatched exception");
        v.at(10000) = 1;
    } catch (std::exception& ex) {
        std::cout << ex.what();
    }
}
```
</p>
</details>

## Общие правила ловли исключений

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

int main() {
    try {
        throw 1;
    } catch (unsigned int x) {

    }
    
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

int main() {
    try {
        throw std::out_of_range("An outsider!");
    } catch (std::exception& ex) {
        std::cout << 1;
    } catch (std::out_of_range& oor) {
        std::cout << 2;
    } catch (...) {
        std::cout << 3;
    }
    
}
```
</p>
</details>

## Исключения и копирование

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct Logger {
    Logger() {
        std::cout << "created ";
    }
    Logger(const Logger&) {
        std::cout << "copied ";        
    }
    ~Logger() {
        std::cout << "destroyed ";
    }
}

void f() {
    Logger x:
    throw x;
}

int main() {
    try {
        f();
    } catch (const Logger& x) {
        std::cout << "caught"
    }
    
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct Logger {
    Logger() {
        std::cout << "created ";
    }
    Logger(const Logger&) {
        std::cout << "copied ";        
    }
    Logger(Logger&&) {
        std::cout << "moved ";        
    }
    ~Logger() {
        std::cout << "destroyed ";
    }
}

void f() {
    Logger x:
    throw x;
}

int main() {
    try {
        f();
    } catch (const Logger& x) {
        std::cout << "caught"
    }
    
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct Logger {
    Logger() {
        std::cout << "created ";
    }
    Logger(const Logger&) {
        std::cout << "copied ";        
    }
    ~Logger() {
        std::cout << "destroyed ";
    }
}

void f() {
    Logger x:
    throw x;
}

int main() {
    try {
        f();
    } catch (Logger x) {
        std::cout << "caught"
    }
    
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct Logger {
    Logger() {
        std::cout << "created ";
    }
    Logger(const Logger&) = delete;
    ~Logger() {
        std::cout << "destroyed ";
    }
}

void f() {
    Logger x:
    throw x;
}

int main() {
    try {
        f();
    } catch (const Logger& x) {
        std::cout << "caught";
    }
    
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct Logger {
    Logger() {
        std::cout << "created ";
    }
    Logger(const Logger&) = delete;
    ~Logger() {
        std::cout << "destroyed ";
    }
}

void f() {    
    throw Logger();
}

void g() {
    try {
        throw std::out_of_range("Outside!");        
    } catch (std::exception& ex) {
        std::cout << "caught";
        throw;
    }
}

int main() {
    try {
        g();
    } catch (std::out_of_range& oor) {
        
    }
    
}
```
</p>
</details>

# RAII 

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

void f() {
    int* p = new int(5);
    throw 1;
    delete p;
}

int main() {
    try {
        f();        
    } catch (...) {

    }
}
```
</p>
</details>

<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

struct S {
    int* p = nullptr;

    S(): p(new int(5)) {
        throw 1;
    }

    ~S() {
        delete p;
    }
}

int main() {
    try {
        S s;
    } catch (...) {
        
    }
}
```
</p>
</details>


<details><summary>code snippet</summary>
<p>

```cpp
#include <iostream>

template <typename T>
class SmartPtr {
    private:
        T* ptr;
    public:
        SmartPtr(T* ptr): ptr(ptr) {}
        ~SmartPtr() {
            delete ptr;
        }
}

struct S {
    SmartPtr<int> p = nullptr;

    S(): p(new int(5)) {
        throw 1;
    }

    ~S() {
        delete p;
    }
}

int main() {
    try {
        S s;
    } catch (...) {
        
    }
}
```
</p>
</details>
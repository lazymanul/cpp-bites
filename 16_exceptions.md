# Исключения

Исключения в том или ином виде реализованых во многих современных языках, но у C++ как всегда есть свои особенности. 

Базовая идея: когда вы пишете код у вас что-то может пойти не так. Например, программа может в ходе выполнения может сделать недопустимую операцию (поделить на ноль, выйти за границу массива) и упасть. В большинстве случаев это не желательно, как минимум хочется успеть собрать информацию о том, что именно и где пошло не так.



## Основная идея

```c
int f(int x, int y, int* result) {
    std::cout << 1;
}

int main() {
    f(1, 0);
}
```



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

```
terminate called after throwing an instance of 'int'
```


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
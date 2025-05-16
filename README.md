# Лабораторная работа 7. Преобразование и анализ кода с использованием Clang и LLVM
Цель работы: Познакомиться с инструментами Clang и LLVM, научиться собирать AST и IR-промежуточное представление кода на C/C++, а также извлекать базовую информацию о программе (например, список функций).
Задачи:
1. Установить Clang и LLVM;
2. Скомпилировать простой C-файл с использованием clang и
получить его: абстрактное синтаксическое дерево (AST), промежуточное
представление LLVM IR;
3. Использовать opt для применения базовой комплексной
оптимизации (например, О2);
4. Построить граф потока управления (CFG) для оптимизированной
программы;
5. Проанализировать результат, сделать выводы и ответить на
контрольные вопросы.

## Ход работы

### 1. Установка и подготовка среды

Работа выполнялась в среде **Ubuntu 22.04**. Установлены следующие инструменты:

- `clang` — компилятор языка C/C++.
- `llvm` — инструменты анализа и оптимизации кода.
- `opt` — инструмент для работы с LLVM IR и применения оптимизаций.
- `Graphviz` — инструмент для визуализации кода.

Команда установки:
```bash
sudo apt install clang llvm graphviz
```
![image](https://github.com/user-attachments/assets/3f8de60f-d0a4-432e-81f3-550ba9b2e622)

### 2. Исходный код

Пример программы `main.c`:
```c
#include <stdio.h>
int add(int a, int b) {
    return a + b;
}

int main() {
    return add(3, 4);
}
```



### 3. Получение AST

Команда для генерации AST:
```bash
clang -Xclang -ast-dump -fsyntax-only main.c
```
![image](https://github.com/user-attachments/assets/b4c2e25c-37ba-456b-a1e5-1f22974a9943)


### 4. Генерация LLVM IR

Команда для генерации LLVM IR:
```bash
clang -S -emit-llvm main.c -o main.ll
```
![image](https://github.com/user-attachments/assets/d89f811b-e3d4-4c67-bfb3-22a1aec5d15e)


### 5. Оптимизация IR

Команда для генерации неоптимизированного IR:
```bash
clang -O0 -S -emit-llvm main.c -o main_O0.ll
```

Особенности IR до оптимизации:
- Все переменные (`a`, `b`) размещены в памяти через `alloca`.
- Множество операций `load` и `store`.
- Функция `add` вызывается как отдельная функция.
![image](https://github.com/user-attachments/assets/ed6332be-89a8-4ce7-bb05-028183091ace)


Команда для генерации оптимизированного IR с уровнем `-O2`:
```bash
clang -O2 -S -emit-llvm main.c -o main_O2.ll
```

Оптимизация `-O2` включает более 30 различных оптимизаций, таких как:
- `-inline`: встраивание небольших функций (встраивает `add` в `main`).
- `-constprop`: подстановка констант: если аргументы функции известны (например, add(3, 4)), результат (7) вычисляется на этапе компиляции;
- `-mem2reg`: перевод переменных из памяти в регистры (SSA).
- `-instcombine`: объединение и упрощение инструкций.
- `-simplifycfg`: оптимизация структуры блоков.
- `-reassociate`, `-gvn`, `-sroa`, `-dce` и другие.

Особенности IR после оптимизации:
- Функция `square` исчезла (встроена через `-inline` и вычислена через `-constprop`).
- Переменные, `alloca`, `store`, `load` удалены (`-mem2reg`, `-dce`)..
![image](https://github.com/user-attachments/assets/43d13d2f-09f0-49d7-863a-baa24a1d8efc)


Команда для сравнения IR до и после оптимизации:
```bash
diff main_O0.ll main_O2.ll
```
![image](https://github.com/user-attachments/assets/dc4148bf-fdd5-46c2-b571-580fd508d3ec)


Изменения после оптимизации:
- Переменные типа `alloca` удалены.
- Код переведён в SSA-форму.
- Улучшена читаемость и упрощён поток управления.


### 6. Граф потока управления программы

Команда для генерации оптимизированного LLVM IR:
```bash
clang -O2 -S -emit-llvm main.c -o main.ll
```

Команда для генерации `.dot`-файлов CFG:
```bash
opt -passes=dot-cfg -disable-output main.ll
```
![image](https://github.com/user-attachments/assets/1914e9bc-1afe-47ba-9760-e35287e39d97)


Вывод:
```bash
find . -name "*.dot"
./.main.dot
./.square.dot
```

Создаются DOT-файлы:
- `.main.dot` — для функции `main`.
- `.square.dot` — для функции `square` (если не была удалена оптимизацией).

Команды для преобразования `.dot` в `.png`:
```bash
dot -Tpng .main.dot -o cfg_main.png
dot -Tpng .add.dot -o cfg_add.png
```

Команды для просмотра CFG:
```bash
xdg-open cfg_main.png
xdg-open cfg_add.png
```
![cfg_main](https://github.com/user-attachments/assets/4ac38506-7beb-4915-9322-678166e90a65)

![cfg_add](https://github.com/user-attachments/assets/2362e8cb-3712-4d5e-845c-63f424126dec)



## Выводы

- С помощью Clang можно получить полную структуру AST, IR и CFG.
- LLVM предоставляет гибкие инструменты анализа и оптимизации.
- Промежуточное представление (IR) удобно для написания компиляторных трансформаций.

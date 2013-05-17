rhg
===

rhg


Глава 2. Объекты
===

Структура Руби объектов

Начиная с этой главы мы начнем изучение исходного кода `ruby`, мы начнем с изучения обявления струтур объетов.

Какаие условия необходимы для существования объекта? На это вопрос можно дать множество ответов, но на самом деле небходимо выполнить три условия:
1. Реализовать отличимость отдельной сущности от всех остальных (наделение идентификатором)
2. Наделить сущность возможность отвечать на запросы (вызовы методов)
3. Позволить сущность хранить внутреннее состояние (переменные экземплра)

В этой главе мы поочереди подтвердим все три утверждения.

Наиболее интересным файлом в этой главе будет `ruby.h`, но мы также частично рассмотрим другие файлы такие как `object.c`, `class.c` or `variable.c`.


Структура типов VALUE и objects
Содержание объекта в `ruby` выражется посредством структур написанных на языке C и всегда передается через указатели на эти структуры. Для каждого класса используются различные структуры, но типом для указателя на класс всегда будет структура VALUE(схема 1).

[http://rhg.rubyforge.org/images/ch_object_value.png]
Схема 1

Вот так выглядит объявление VALUE
```
# (ruby.h)
71  typedef unsigned long VALUE;
```

В практике, VALUE должно преобразовываться в указатели на другие типы структур. В связи эти если длина unsigned long и требуемого указателя не совпадают руби работает некорректно. Точнее говоря, VALUE не будет работать для указателей размером больших чем sizeof(unsigned long). К счастью, нет совреммных машин релизующих эту возможность, несмотря на то что несколько лет назад их было множество.

Следующие несколько структур реализуют объекты руби классов:
struct RObject - структура объектов не относящихся к классам ниже
struct RClass - структура классов
struct RFloat - структура вещественных чисел
struct RString - структура строк
struct RArray - структура массивов
struct RRegexp - структура кергулярных выражений
struct RHash - структура хещей
struct RFile	IO, File, Socket, etc…
struct RData - все структуры объявленные на C уровне, за исключением упомянутых выше
struct RStruct - структура отображающая объекты руби класса Struct
struct RBignum - структура для целых числ более 2**31

Например, для объекта строки используется структура struct RString, таким образом мы получаем что-то наподобии:
[http://rhg.rubyforge.org/images/ch_object_string.png]
САхема 2

Давайте взглянем на объявление некоторых структур объектов
```
# (ruby.h)

     /* structure for ordinary objects */
 295  struct RObject {
 296      struct RBasic basic;
 297      struct st_table *iv_tbl;
 298  };

      /* structure for strings (instance of String) */
 314  struct RString {
 315      struct RBasic basic;
 316      long len;
 317      char *ptr;
 318      union {
 319          long capa;
 320          VALUE shared;
 321      } aux;
 322  };

      /* structure for arrays (instance of Array) */
 324  struct RArray {
 325      struct RBasic basic;
 326      long len;
 327      union {
 328          long capa;
 329          VALUE shared;
 330      } aux;
 331      VALUE *ptr;
 332  };
```

Перед началом более детального рассмотрения каждой структуры, давайте раасммотрим несколько более глобальных аспектов.

Первое, VALUE объявлено как unsigned long, поэтому необходимо привести тип перед использованием. поэтому для каждой структуры был создан
Rxxxx() макрос. Для структуры struct RString есть макрос RSTRING(), struct RArray - RARRAY() и т.д. Эти макросы используются так:
```
VALUE str = ....;
VALUE arr = ....;
RSTRING(str)->len;   /* ((struct RString*)str)->len */
RARRAY(arr)->len;    /* ((struct RArray*)arr)->len */
```

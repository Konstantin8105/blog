# Блог

# Оглавление

* Go выйграл в 2010 Bossie Award [bossie](#bossie)
* Встроенный фаззинг Go [fuzz-beta](#fuzz-beta)
* Параметры типа, наборы типов, сравнимые типы, удовлетворение ограничений [comparable](#comparable)
* [Оригинальные материалы](#Оригинальные-материалы)












# bossie

[^](#Оглавление)

Go выйграл в 2010 Bossie Award как “лучшее программное обеспечение для разработки с открытым исходным кодом”

date: 2010-09-06

by:
- Andrew Gerrand



Проект Go был номинирован на [Bossie Award](http://www.infoworld.com/d/open-source/bossie-awards-2010-the-best-open-source-application-development-software-140&current=2&last=1) как “лучшее программное обеспечение для разработки с открытым исходным кодом. [Bossie Awards](http://www.infoworld.com/d/open-source/bossie-awards-2010-the-best-open-source-software-the-year-115) проходит каждый год начиная с 2007 [InfoWorld](http://infoworld.com/) для определения лучших проектов с открытым исходным кодом.

Цитата:

Язык Go от Google пытается вернуть простоту программирования. Он покончил с многочисленными конструкциями, проникшими во все объектно-ориентированные языки, заново представляя, как упростить и улучшить общение между разработчиком и кодом. Go обеспечивает сбор мусора, безопасность типов, безопасность памяти и встроенную поддержку параллелизма и символов Юникода. Кроме того, он компилируется (быстро!) в двоичные файлы для нескольких платформ. Go все еще находится в разработке (добавляются новые функции) и имеет ограничения, в частности плохую поддержку Windows.
Но это показывает новое, захватывающее направление в языках программирования.

Команда Go рада принять эту награду.
Это отражает не только нашу тяжелую работу, но и усилия растущего сообщества разработчиков Go, которым мы должны выразить нашу искреннюю благодарность.

















# fuzz-beta

[^](#Оглавление)

Резюме: Встроенный фаззинг Go теперь готово к бета-тестированию.

date: 2021-06-03

by:
- Katie Hockman
- Jay Conrod


Мы рады сообщить, что встроенный фаззинг готов к бета-тестированию!

Фаззинг — это тип автоматического тестирования, при котором постоянно манипулируются вводимыми данными в программу для поиска таких проблем, как паника или ошибки. Эти полуслучайные мутации данных могут обнаружить новое покрытие кода, которое могут пропустить существующие модульные тесты, а также выявить ошибки в крайних случаях, которые в противном случае остались бы незамеченными. Поскольку фаззинг может достигать этих крайних случаев, фазз-тестирование особенно ценно для обнаружения эксплойтов и уязвимостей безопасности.


Смотри [golang.org/s/draft-fuzzing-design](/s/draft-fuzzing-design) для получения большей информации.


## Начнем

Для начала необходимо запустить следующее

```
	$ go install golang.org/dl/gotip@latest
	$ gotip download
```

Это создает цепочку инструментов Go из основной ветки. После этого команда gotip может заменить команду go. Теперь вы можете запускать такие команды, как

```
	$ gotip test -fuzz=Fuzz
```

## Написание фаззинг теста

Фаззинг тест должен находиться в файле \*\_test.go в функции с именем `FuzzXxx`.
Эта функция должна иметь аргумент ` *testing.F`, похоше на агрумент `*testing.T`для функций `TestXxx`.

Ниже написан пример фаззинг теста для тестирования поведения [net/url package](https://pkg.go.dev/net/url#ParseQuery).

```Go
//go:build go1.18
// +build go1.18

package fuzz

import (
	"net/url"
	"reflect"
	"testing"
)

func FuzzParseQuery(f *testing.F) {
	f.Add("x=1&y=2")
	f.Fuzz(func(t *testing.T, queryStr string) {
		query, err := url.ParseQuery(queryStr)
		if err != nil {
			t.Skip()
		}
		queryStr2 := query.Encode()
		query2, err := url.ParseQuery(queryStr2)
		if err != nil {
			t.Fatalf("ParseQuery failed to decode a valid encoded query %s: %v", queryStr2, err)
		}
		if !reflect.DeepEqual(query, query2) {
			t.Errorf("ParseQuery gave different query after being encoded\nbefore: %v\nafter: %v", query, query2)
		}
	})
}
```

Вы можете прочитать больше о фаззинге в pkg.go.dev, включая [Основы фаззинга в Go](https://pkg.go.dev/testing@master#hdr-Fuzzing) и [godoc навого типа `testing.F`](https://pkg.go.dev/testing@master#F).

## Ожидания

Это нововведение остается в стадии beta, поэтому стоит ожидать появление некоторых ошибок и неполного функционала. Проверяйте [в трекере проблем с подписью “fuzz”](https://github.com/golang/go/issues?q=is%3Aopen+is%3Aissue+label%3Afuzz) для того чтобы быть в курсе существующих ошибок и неработающим функционале.

Имейте в виду, что фаззинг может потреблять много памяти и влиять на производительность вашей машины во время ее работы. `go test -fuzz` по умолчанию запускает фаззинг в процессах `$GOMAXPROCS` параллельно. Вы можете уменьшить количество процессов, используемых при фаззинге, явно установив флаг `-parallel` с помощью `go test`. Прочтите документацию по команде go test, запустив gotip help testflag, если вам нужна дополнительная информация.


Также имейте в виду, что механизм фаззинга записывает значения, которые расширяют тестовое покрытие, в каталог фазз-кэша внутри `$GOCACHE/fuzz` во время своей работы. В настоящее время нет ограничений на количество файлов или общее количество байтов, которые могут быть записаны в фазз-кеш, поэтому он может занимать большой объем памяти (например, несколько ГБ). Вы можете очистить кэш фазза, запустив `gotip clean -fuzzcache`.


## Следующий шаг?

Это нововведение будет доступно в Go 1.18.

Если встречаетесь с любыми проблемами или есть идея для функционала, то просим создать [запись на стену](https://github.com/golang/go/issues/new/?&labels=fuzz).

Для обсуждения и получения обратной связи о фазинге, просим также использовать канал [#fuzzing channel](https://gophers.slack.com/archives/CH5KV1AKE) в Gophers Slack.

Счастливого fuzzing!

















# comparable

[^](#Оглавление)

Параметры типа, наборы типов, сравнимые типы, удовлетворение ограничений

date: 2023-02-17

by:
- Robert Griesemer

1 февраля мы выпустили Go версии, 1.20, который включает изменения в язык.
Обсуждим одно из этих измнений: предварительно объявленное ограничение типа `comparable`(сравнимые) удовлетворяет всем [Сравнимым типам comparable types](/ref/spec#Comparison_operators).
Удивительно, что до Go 1.20, некоторые сравнимые типы не удовлетворяли ставнимости `comparable`!

Если вы запустались, то попали по адресу.
Рассмотрим допустимое объявление карты (map)

```Go
var lookupTable map[any]string
```

где в качестве ключа используется тип `any` (который является [сравнимым типом (comparable type)](/ref/spec#Comparison_operators)).
И это превосходно работает в Go.
Но с другой стороны, до версии Go 1.20, выглядил как общий тип карт(generic map type)

```Go
type genericLookupTable[K comparable, V any] map[K]V
```

может быть использован как обычный тип карты, но производил ошибку во время компиляции где `any` используется как ключ типа:

```Go
var lookupTable genericLookupTable[any, string] // ERROR: any does not implement comparable (Go 1.18 and Go 1.19)
                                                // Ошибка: any не соответствует сравнимому типу comparable (Go 1.18 and Go 1.19)
```

Начиная  с Go 1.20 этот код успешно компилируется.

В предедущих версиях до Go 1.20 поведение `comparable` было прискорбным, потому что не позволял писать универсальные библиотеки, которые мы изначально надеялись написать в помощью дженериков(generics).
В предложении [`maps.Clone`](/issue/57436) функция

```Go
func Clone[M ~map[K]V, K comparable, V any](m M) M { … }
```
может быть написана, но не может быть использована для карт как `lookupTable` по тем же причинам что и `genericLookupTable` не мог использовать тип ключа `any`.

Ниже описано проливается свет на языковой механизм стоящий за этим.
Для этого начнем с небольшой вводной информации.

## Параметры типа и ограничения (Type parameters and constraints)

В версии Go 1.18 были включены дженерики(generics) и также [Параметры типа_type parameters_](/ref/spec#Type_parameter_declarations) как новая языковая конструкция.

В обычной функции, параметр в пределах ограниченного типом.
Аналогично, в дженерик функциях(или типах), тип параметра в пределах обозначенных типов, которые ограничены [ограничение типа_type constraint_](/ref/spec#Type_constraints).
Таким образом, ограничение типа определяет набор типов допустимых как типы аргумента.

В Go 1.18 также изменено точка зрения на интерфейсы:
Если ранее интерфейс определял набор методов, то теперь интерфейс определяет набор типов.

Эта точка зрения является обратно несовместимой:
для любого заданного набора методов, определенных интерфейсом, мы можем представить (бесконечный) набор всех типов, реализующих эти методы.

Например, имея интерфейс [`io.Writer`](/pkg/io#Writer), мы можем представить бесконечное множество всех типов, которые имеют метод `Write` с соответствующей сигнатурой.
Все эти типы _реализуют_ интерфейс, поскольку все они имеют обязательный метод Write.

Но новое представление набора типов более мощное, чем старое представление набора методов:
мы можем описать набор типов явно, а не только косвенно через методы.

Это дает нам новые способы управления набором типов.
Начиная с Go 1.18, интерфейс может включать не только другие интерфейсы, но и любой тип, объединение типов или бесконечное множество типов, которые имеют один и тот же [базовый тип](/ref/spec#Underlying_types). Эти типы затем включаются в [вычисление набора типов](/ref/spec#General_interfaces):

обозначение объединения `A|B` означает «тип `A` или тип `B`», а обозначение `~T` означает «все типы, имеющие базовый тип `T`».

Например, интерфейс


```Go
interface {
	~int | ~string
	io.Writer
}
```

определяет набор всех типов, базовыми типами которых являются `int` или `string` и которые также реализуют метод Write `io.Writer`.

Такие обобщенные интерфейсы нельзя использовать в качестве типов переменных.
Но поскольку они описывают наборы типов, они используются как ограничения типов, которые представляют собой наборы типов.
Например, мы можем написать общую функцию min.

```Go
func min[P interface{ ~int64 | ~float64 }](x, y P) P
```

который принимает любой аргумент `int64` или `float64`.

(Конечно, более реалистичная реализация могла бы использовать ограничение, перечисляющее все базовые типы с помощью оператора <code>&lt;</code>.)

Кроме того, поскольку перечисление явных типов без методов является обычным явлением, немного [синтаксического сахара](https://en.wikipedia.org/wiki/Syntactic_sugar) позволяет нам [опустить включающий `interface{}`]( /ref/spec#General_interfaces), что приводит к более компактному и идиоматическому интерфейсу.

```Go
func min[P ~int64 | ~float64](x, y P) P { … }
```

With the new type set view we also need a new way to explain what it means to [_implement_](/ref/spec#Implementing_an_interface) an interface.
We say that a (non-interface) type `T` implements an interface `I` if `T` is an element of the interface's type set.
If `T` is an interface itself, it describes a type set. Every single type in that set must also be in the type set of `I`, otherwise `T` would contain types that do not implement `I`.
Thus, if `T` is an interface, it implements interface `I` if the type set of `T` is a subset of the type set of `I`.

Now we have all the ingredients in place to understand constraint satisfaction.
As we have seen earlier, a type constraint describes the set of acceptable argument types for a type parameter. A type argument satisfies the corresponding type parameter constraint if the type argument is in the set described by the constraint interface.
This is another way of saying that the type argument implements the constraint.
In Go 1.18 and Go 1.19, constraint satisfaction meant constraint implementation.
As we'll see in a bit, in Go 1.20 constraint satisfaction is not quite constraint implementation anymore.

## Operations on type parameter values

A type constraint does not just specify what type arguments are acceptable for a type parameter, it also determines the operations that are possible on values of a type parameter.
As we would expect, if a constraint defines a method such as `Write`,
the `Write` method can be called on a value of the respective type parameter.
More generally, an operation such as `+` or `*` that is supported by all types in the type set defined by a constraint is permitted with values of the corresponding type parameter.

For instance, given the `min` example, in the function body any operation that is supported by `int64` and `float64` types is permitted on values of the type parameter `P`.
That includes all the basic arithmetic operations, but also comparisons such as <code>&lt;</code>.
But it does not include bitwise operations such as `&` or `|` because those operations are not defined on `float64` values.

## Comparable types

In contrast to other unary and binary operations, `==` is defined on not just a limited set of [predeclared types](/ref/spec#Types), but on an infinite variety of types, including arrays, structs, and interfaces.
It is impossible to enumerate all these types in a constraint.
We need a different mechanism to express that a type parameter must support `==` (and `!=`, of course) if we care about more than predeclared types.

We solve this problem through the predeclared type [`comparable`](/ref/spec#Predeclared_identifiers), introduced with Go 1.18.
`comparable` is an interface type whose type set is the infinite set of comparable types, and that may be used as a constraint whenever we require a type argument to support `==`.

Yet, the set of types comprised by `comparable` is not the same as the set of all [comparable types](/ref/spec#Comparison_operators) defined by the Go spec.
[By construction](/ref/spec#Interface_types), a type set specified by an interface (including `comparable`) does not contain the interface itself (or any other interface).
Thus, an interface such as `any` is not included in `comparable`, even though all interfaces support `==`.

What gives?

Comparison of interfaces (and of composite types containing them) may panic at run time:
this happens when the dynamic type, the type of the actual value stored in the interface variable, is not comparable.
Consider our original `lookupTable` example: it accepts arbitrary values as keys.
But if we try to enter a value with a key that does not support `==`, say a slice value, we get a run-time panic:

```Go
lookupTable[[]int{}] = "slice"  // PANIC: runtime error: hash of unhashable type []int
```

By contrast, `comparable` contains only types that the compiler guarantees will not panic with `==`.
We call these types _strictly comparable_.

Most of the time this is exactly what we want: it's comforting to know that `==` in a generic function won't panic if the operands are constrained by `comparable`, and it is what we would intuitively expect.

Unfortunately, this definition of `comparable` together with the rules for constraint satisfaction prevented us from writing useful generic code, such as the `genericLookupTable` type shown earlier:

for `any` to be an acceptable argument type, `any` must satisfy (and therefore implement) `comparable`.
But the type set of `any` is larger than (not a subset of) the type set of `comparable` and therefore does not implement `comparable`.

```Go
var lookupTable GenericLookupTable[any, string] // ERROR: any does not implement comparable (Go 1.18 and Go 1.19)
```

Users recognized the problem early on and filed a multitude of issues and proposals in short order ([#51338](/issue/51338), [#52474](/issue/52474), [#52531](/issue/52531), [#52614](/issue/52614), [#52624](/issue/52624), [#53734](/issue/53734), etc).
Clearly this was a problem we needed to address.

The "obvious" solution was simply to include even non-strictly comparable types in the `comparable` type set.
But this leads to inconsistencies with the type set model.
Consider the following example:

```Go
func f[Q comparable]() { … }

func g[P any]() {
        _ = f[int] // (1) ok: int implements comparable
        _ = f[P]   // (2) error: type parameter P does not implement comparable
        _ = f[any] // (3) error: any does not implement comparable (Go 1.18, Go.19)
}
```

Function `f` requires a type argument that is strictly comparable.
Obviously it is ok to instantiate `f` with `int`: `int` values never panic on `==` and thus `int` implements `comparable` (case 1).
On the other hand, instantiating `f` with `P` is not permitted: `P`'s type set is defined by its constraint `any`, and `any` stands for the set of all possible types.
This set includes types that are not comparable at all.
Hence, `P` doesn't implement `comparable` and thus cannot be used to instantiate `f` (case 2).
And finally, using the type `any` (rather than a type parameter constrained by `any`) doesn't work either, because of exactly the same problem (case 3).

Yet, we do want to be able to use the type `any` as type argument in this case.
The only way out of this dilemma was to change the language somehow.
But how?

## Interface implementation vs constraint satisfaction

As mentioned earlier, constraint satisfaction is interface implementation:
a type argument `T` satisfies a constraint `C` if `T` implements `C`.
This makes sense: `T` must be in the type set expected by `C` which is exactly the definition of interface implementation.

But this is also the problem because it prevents us from using non-strictly comparable types as type arguments for `comparable`.

So for Go 1.20, after almost a year of publicly discussing numerous alternatives (see the issues mentioned above), we decided to introduce an exception for just this case.
To avoid the inconsistency, rather than changing what `comparable` means, we differentiated between _interface implementation_, which is relevant for passing values to variables, and _constraint satisfaction_, which is relevant for passing type arguments to type parameters.
Once separated, we could give each of those concepts (slightly) different rules, and that is exactly what we did with proposal [#56548](/issue/56548).

The good news is that the exception is quite localized in the [spec](/ref/spec#Satisfying_a_type_constraint).
Constraint satisfaction remains almost the same as interface implementation, with a caveat:

> A type `T` satisfies a constraint `C` if
>
> - `T` implements `C`; or
> - `C` can be written in the form `interface{ comparable; E }`, where `E` is a basic interface
>   and `T` is [comparable](/ref/spec#Comparison_operators) and implements `E`.

The second bullet point is the exception.
Without going too much into the formalism of the spec, what the exception says is the following:
a constraint `C` that expects strictly comparable types (and which may also have other requirements such as methods `E`) is satisfied by any type argument `T` that supports `==` (and which also implements the methods in `E`, if any).

Or even shorter: a type that supports `==` also satisfies `comparable` (even though it may not implement it).

We can immediately see that this change is backward-compatible:
before Go 1.20, constraint satisfaction was the same as interface implementation, and we still have that rule (1st bullet point).
All code that relied on that rule continues to work as before.
Only if that rule fails do we need to consider the exception.

Let's revisit our previous example:

```Go
func f[Q comparable]() { … }

func g[P any]() {
        _ = f[int] // (1) ok: int satisfies comparable
        _ = f[P]   // (2) error: type parameter P does not satisfy comparable
        _ = f[any] // (3) ok: satisfies comparable (Go 1.20)
}
```

Now, `any` does satisfy (but not implement!) `comparable`.

Why?

Because Go permits `==` to be used with values of type `any` (which corresponds to the type `T` in the spec rule), and because the constraint `comparable` (which corresponds to the constraint `C` in the rule) can be written as `interface{ comparable; E }` where `E` is simply the empty interface in this example (case 3).

Interestingly, `P` still does not satisfy `comparable` (case 2).
The reason is that `P` is a type parameter constrained by `any` (it _is not_ `any`).
The operation `==` is _not_ available with all types in the type set of `P` and thus not available on `P`; it is not a [comparable type](/ref/spec#Comparison_operators).
Therefore the exception doesn't apply.
But this is ok: we do like to know that `comparable`, the strict comparability requirement, is enforced most of the time. We just need an exception for Go types that support `==`, essentially for historical reasons:
we always had the ability to compare non-strictly comparable types.

## Consequences and remedies

We gophers take pride in the fact that language-specific behavior can be explained and reduced to a fairly compact set of rules, spelled out in the language spec.
Over the years we have refined these rules, and when possible made them simpler and often more general.
We also have been careful to keep the rules orthogonal, always on the lookout for unintended and unfortunate consequences.
Disputes are resolved by consulting the spec, not by decree.
That is what we have aspired to since the inception of Go.

_One does not simply add an exception to a carefully crafted type system without consequences!_

So where's the catch?
There's an obvious (if mild) drawback, and a less obvious (and more severe) one.
Obviously, we now have a more complex rule for constraint satisfaction which is arguably less elegant than what we had before.
This is unlikely to affect our day-to-day work in any significant way.

But we do pay a price for the exception: in Go 1.20, generic functions that rely on `comparable` are not statically type-safe anymore.
The `==` and `!=` operations may panic if applied to operands of `comparable` type parameters, even though the declaration says that they are strictly comparable.
A single non-comparable value may sneak its way through multiple generic functions or types by way of a single non-strictly comparable type argument and cause a panic.
In Go 1.20 we can now declare

```Go
var lookupTable genericLookupTable[any, string]
```

without compile-time error, but we will get a run-time panic if we ever use a non-strictly comparable key type in this case, exactly like we would with the built-in `map` type.
We have given up static type safety for a run-time check.

There may be situations where this is not good enough, and where we want to enforce strict comparability.
The following observation allows us to do exactly that, at least in limited form: type parameters do not benefit from the exception that we added to the constraint satisfaction rule.
For instance, in our earlier example, the type parameter `P` in the function `g` is constrained by `any` (which by itself is comparable but not strictly comparable) and so `P` does not satisfy `comparable`.
We can use this knowledge to craft a compile-time assertion of sorts for a given type `T`:

```Go
type T struct { … }
```

We want to assert that `T` is strictly comparable.
It's tempting to write something like:

```Go
// isComparable may be instantiated with any type that supports ==
// including types that are not strictly comparable because of the
// exception for constraint satisfaction.
func isComparable[_ comparable]() {}

// Tempting but not quite what we want: this declaration is also
// valid for types T that are not strictly comparable.
var _ = isComparable[T] // compile-time error if T does not support ==
```

The dummy (blank) variable declaration serves as our "assertion".
But because of the exception in the constraint satisfaction rule, `isComparable[T]` only fails if `T` is not comparable at all; it will succeed if `T` supports `==`.
We can work around this problem by using `T` not as a type argument, but as a type constraint:

```Go
func _[P T]() {
	_ = isComparable[P] // P supports == only if T is strictly comparable
}
```

Here is a [passing](/play/p/9i9iEto3TgE) and [failing](/play/p/5d4BeKLevPB) playground example illustrating this mechanism.

## Final observations

Interestingly, until two months before the Go 1.18 release, the compiler implemented constraint satisfaction exactly as we do now in Go 1.20.
But because at that time constraint satisfaction meant interface implementation, we did have an implementation that was inconsistent with the language specification.
We were alerted to this fact with [issue #50646](/issue/50646).
We were extremely close to the release and had to make a decision quickly.
In the absence of a convincing solution, it seemed safest to make the implementation consistent with the spec.
A year later, and with plenty of time to consider different approaches, it seems that the implementation we had was the implementation we wanted in the first place.
We have come full circle.

As always, please let us know if anything doesn't work as expected by filing issues at [https://go.dev/issue/new](/issue/new).

Thank you!















# Оригинальные материалы

[^](#Оглавление)


```
git clone https://github.com/golang/website.git

Папка
website/_content/blog

$ git log
commit e21a2c79bdb8f8a76834192808e90d89be1e3c31 (HEAD -> master, origin/master, origin/HEAD)
Author: Dmitri Shuralyov <dmitshur@golang.org>
Date:   Thu Sep 28 11:12:21 2023 -0700

    cmd/{adminapp,adminredirect,googlegolangorg}: update App Engine runtime
    
    Keep them up with the times.
    
    Change-Id: I8f6940b59cac0c70320a796a0c3cfcdef8a2633d
    Reviewed-on: https://go-review.googlesource.com/c/website/+/531676
    Auto-Submit: Dmitri Shuralyov <dmitshur@golang.org>
    LUCI-TryBot-Result: Go LUCI <golang-scoped@luci-project-accounts.iam.gserviceaccount.com>
    Reviewed-by: Dmitri Shuralyov <dmitshur@google.com>
    Reviewed-by: Hyang-Ah Hana Kim <hyangah@gmail.com>

```

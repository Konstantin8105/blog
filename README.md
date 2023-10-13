# Блог

# Оглавление

* Go выйграл в 2010 Bossie Award [bossie](#bossie)
* Встроенный фаззинг Go [fuzz-beta](#fuzz-beta)
* Параметры типа, наборы типов, сравнимые типы, удовлетворение ограничений [comparable](#comparable)
* Почему сигнатуры функций в пакетах слайсов такие сложные [deconstructing-type-parameters](#deconstructing-type-parameters)
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
С новым представлением набора типов нам также нужен новый способ объяснить, что означает [_implement_](/ref/spec#Implementing_an_interface) интерфейс.

Мы говорим, что (неинтерфейсный) тип T реализует интерфейс I, если T является элементом набора типов интерфейса.
Если `T` сам по себе является интерфейсом, он описывает набор типов. Каждый отдельный тип в этом наборе также должен входить в набор типов I, иначе T будет содержать типы, которые не реализуют I.
Таким образом, если `T` является интерфейсом, он реализует интерфейс `I`, если набор типов `T` является подмножеством набора типов `I`.

Теперь у нас есть все необходимое для понимания удовлетворения ограничений.

Как мы видели ранее, ограничение типа описывает набор допустимых типов аргументов для параметра типа. Аргумент типа удовлетворяет соответствующему ограничению параметра типа, если аргумент типа находится в наборе, описанном интерфейсом ограничения.
Это еще один способ сказать, что аргумент типа реализует ограничение.

В Go 1.18 и Go 1.19 удовлетворение ограничений означало реализацию ограничений.
Как мы вскоре увидим, в Go 1.20 удовлетворение ограничений больше не является реализацией ограничений.

## Операции над значениями параметров типа(type parameter values)

Ограничение типа не только определяет, какие аргументы типа приемлемы для параметра типа, но также определяет операции, которые возможны над значениями параметра типа.
Как и следовало ожидать, если ограничение определяет такой метод, как `Write`, то метод `Write` можно вызвать для значения соответствующего параметра типа.

В более общем смысле, такая операция, как `+` или `*`, которая поддерживается всеми типами в наборе типов, определенном ограничением, разрешена со значениями соответствующего параметра типа.

Например, в примере `min` в теле функции разрешена любая операция, поддерживаемая типами int64 и float64, со значениями параметра типа `P`.

Сюда входят все основные арифметические операции, а также сравнения, такие как <code>&lt;</code>.
Но он не включает побитовые операции, такие как `&` или `|`, поскольку эти операции не определены для значений `float64`.

## Типы сравнения (Comparable types)

В отличие от других унарных и бинарных операций, `==` определяется не только для ограниченного набора [заранее объявленных типов](/ref/spec#Types), но и для бесконечного множества типов, включая массивы, структуры и интерфейсы.

Невозможно перечислить все эти типы в ограничении.
Нам нужен другой механизм, чтобы выразить, что параметр типа должен поддерживать `==` (и `!=`, конечно), если нас интересуют не только предварительно объявленные типы.

Мы решаем эту проблему с помощью предварительно объявленного типа [`comparable`](/ref/spec#Predeclared_identifiers), представленного в Go 1.18.

`comparable` — это тип интерфейса, набор типов которого представляет собой бесконечное множество сравнимых типов, и который можно использовать в качестве ограничения всякий раз, когда нам требуется аргумент типа для поддержки `==`.

Тем не менее, набор типов, содержащийся в `comparable`, не совпадает с набором всех [сравнимых типов](/ref/spec#Comparison_operators), определенных в спецификации Go.
[По конструкции](/ref/spec#Interface_types), набор типов, заданный интерфейсом (включая `comparable`), не содержит сам интерфейс (или любой другой интерфейс).
Таким образом, такой интерфейс, как `any`, не включается в `comparable`, хотя все интерфейсы поддерживают `==`.

Что дает?

Сравнение интерфейсов (и составных типов, содержащих их) может вызвать панику во время выполнения:

это происходит, когда динамический тип, тип фактического значения, хранящегося в переменной интерфейса, не сравним.

Рассмотрим наш оригинальный пример с LookupTable: он принимает произвольные значения в качестве ключей.
Но если мы попытаемся ввести значение с ключом, который не поддерживает `==`, скажем, значение среза, мы получим панику во время выполнения:

```Go
lookupTable[[]int{}] = "slice"  // PANIC: runtime error: hash of unhashable type []int
                                // Ошибка: во время выполнения: нехэшируемый тип []int
```

Напротив, `comparable` содержит только те типы, которые, как гарантирует компилятор, не вызовут паники при `==`.
Мы называем эти типы «строго сопоставимыми».

В большинстве случаев это именно то, что нам нужно: приятно знать, что `==` в обобщенной функции не будет паниковать, если операнды ограничены `comparable`, и это то, чего мы интуитивно ожидаем.

К сожалению, это определение «сравнимости» вместе с правилами удовлетворения ограничений не позволило нам написать полезный общий код, такой как тип «genericLookupTable», показанный ранее:

Чтобы `any` был приемлемым типом аргумента, `any` должен удовлетворять (и, следовательно, реализовываться) `comparable`.
Но набор типов `any` больше (не является подмножеством) набора типов `comparable` и, следовательно, не реализует `comparable`.

```Go
var lookupTable GenericLookupTable[any, string] // ERROR: any does not implement comparable (Go 1.18 and Go 1.19)
                                                // Ошибка: any не реализует comparable (Go 1.18 and Go 1.19)
```

Пользователи сразу заметили проблему и в короткие сроки подали множество вопросов и предложений ([#51338](/issue/51338), [#52474](/issue/52474), [#52531](/issue/52531). , [#52614](/issue/52614), [#52624](/issue/52624), [#53734](/issue/53734) и т. д.).
Очевидно, это была проблема, которую нам нужно было решить.

«Очевидным» решением было просто включить в набор `comparable` типов даже не строго сопоставимые типы.
Но это приводит к несоответствию модели набора типов.

Рассмотрим следующий пример:

```Go
func f[Q comparable]() { … }

func g[P any]() {
        _ = f[int] // (1) ok: int implements comparable
        _ = f[P]   // (2) error: type parameter P does not implement comparable
        _ = f[any] // (3) error: any does not implement comparable (Go 1.18, Go.19)
}
```

Функция `f` требует аргумента типа, который строго сопоставим.

Очевидно, что можно создать экземпляр `f` с помощью `int`: значения `int` никогда не паникуют при `==` и, таким образом, `int` реализует `comparable` (случай 1).

С другой стороны, создание экземпляра `f` с помощью `P` не разрешено: набор типов `P` определяется его ограничением `any`, а `any` обозначает набор всех возможных типов.
В этот набор входят типы, которые вообще несопоставимы.
Следовательно, `P` не реализует `comparable` и, следовательно, не может использоваться для создания экземпляра `f` (случай 2).

И, наконец, использование типа `any` (а не параметра типа, ограниченного `any`) также не работает из-за точно такой же проблемы (случай 3).

Тем не менее, в этом случае мы хотим иметь возможность использовать тип `any` в качестве аргумента типа.
Единственным выходом из этой дилеммы было как-то изменить язык.
Но как?

## Реализация интерфейса и удовлетворение ограничений (Interface implementation vs constraint satisfaction)

Как упоминалось ранее, удовлетворение ограничений — это реализация интерфейса:
аргумент типа T удовлетворяет ограничению C, если T реализует C.

Это имеет смысл: `T` должен принадлежать к набору типов, ожидаемому `C`, что и является определением реализации интерфейса.

Но это также проблема, потому что это не позволяет нам использовать не строго сопоставимые типы в качестве аргументов типа для `comparable`.

Итак, для Go 1.20, после почти года публичного обсуждения многочисленных альтернатив (см. проблемы, упомянутые выше), мы решили ввести исключение именно для этого случая.
Чтобы избежать несоответствия, вместо того, чтобы менять значение термина «сравнимый», мы разграничили _реализацию интерфейса_, которая важна для передачи значений переменным, и _удовлетворение ограничений_, которая важна для передачи аргументов типа в параметры типа.
После разделения мы могли бы дать каждой из этих концепций (немного) разные правила, и это именно то, что мы сделали с предложением [#56548](/issue/56548).

Хорошей новостью является то, что исключение полностью локализовано в [spec](/ref/spec#Satisfying_a_type_constraint).
Удовлетворение ограничений остается почти таким же, как и реализация интерфейса, с оговоркой:

> A type `T` satisfies a constraint `C` if
>
> - `T` implements `C`; or
> - `C` can be written in the form `interface{ comparable; E }`, where `E` is a basic interface
>   and `T` is [comparable](/ref/spec#Comparison_operators) and implements `E`.


Второй пункт является исключением.

Не вдаваясь слишком в формализм спецификации, исключение говорит следующее:

ограничение `C`, которое ожидает строго сопоставимых типов (и которое может также иметь другие требования, такие как методы `E`), удовлетворяется любым аргументом типа `T`, который поддерживает `==` (и который также реализует методы в `E `, если есть).

Или даже короче: тип, поддерживающий `==`, также удовлетворяет `comparable` (даже если он может его не реализовывать).

Мы сразу видим, что это изменение обратно совместимо:
до Go 1.20 удовлетворение ограничений было таким же, как и реализация интерфейса, и это правило до сих пор действует (первый пункт).
Весь код, основанный на этом правиле, продолжает работать как прежде.
Только если это правило не работает, нам нужно рассмотреть исключение.

Давайте вернемся к нашему предыдущему примеру:

```Go
func f[Q comparable]() { … }

func g[P any]() {
        _ = f[int] // (1) ok: int satisfies comparable
        _ = f[P]   // (2) error: type parameter P does not satisfy comparable
        _ = f[any] // (3) ok: satisfies comparable (Go 1.20)
}
```

Теперь `any` удовлетворяет (но не реализует!) `comparable`.

Почему?

Поскольку Go позволяет использовать `==` со значениями типа `any` (который соответствует типу `T` в правиле спецификации), а также потому, что ограничение `comparable` (которое соответствует ограничению `C` в правиле спецификации) можно записать как `interface{ comparable; E }` где `E` — это просто пустой интерфейс в этом примере (случай 3).

Интересно, что «P» все еще не удовлетворяет «сопоставимому» (случай 2).
Причина в том, что `P` — это параметр типа, ограниченный `any` (это _не_ `any`).

Операция `==` _не_ доступна для всех типов в наборе типов `P` и, следовательно, недоступна для `P`; это не [сравнимый тип](/ref/spec#Comparison_operators).
Поэтому исключение не применяется.

Но это нормально: нам хотелось бы знать, что в большинстве случаев соблюдается строгое требование сопоставимости. Нам просто нужно исключение для типов Go, поддерживающих `==`, в основном по историческим причинам:
у нас всегда была возможность сравнивать не строго сопоставимые типы.

## Последствия и способы устранения (Consequences and remedies)

Мы, гордимся тем, что поведение, специфичное для языка, можно объяснить и свести к довольно компактному набору правил, прописанных в спецификации языка.

За прошедшие годы мы усовершенствовали эти правила и, когда это было возможно, сделали их более простыми, а зачастую и более общими.
Мы также старались поддерживать ортогональность правил, всегда отслеживая непредвиденные и печальные последствия.

Споры разрешаются путем изучения спецификации, а не указа.
Это то, к чему мы стремились с момента создания Go.

_Невозможно просто добавить исключение в тщательно продуманную систему типов без последствий!_

Так где же подвох?

Есть очевидный (пусть и незначительный) недостаток и менее очевидный (и более серьезный).
Очевидно, что теперь у нас есть более сложное правило удовлетворения ограничений, которое, возможно, менее элегантно, чем то, что было раньше.
Это вряд ли существенно повлияет на нашу повседневную работу.

Но мы платим цену за это исключение: в Go 1.20 универсальные функции, основанные на `comparable`, больше не являются статически типобезопасными.
Операции `==` и `!=` могут вызывать панику, если применяются к операндам параметров типа `сравнимый`, даже если в объявлении указано, что они строго сопоставимы.

Одно несопоставимое значение может проникнуть через несколько универсальных функций или типов посредством одного не строго сопоставимого аргумента типа и вызвать панику.

В Go 1.20 мы теперь можем объявить

```Go
var lookupTable genericLookupTable[any, string]
```

без ошибки во время компиляции, но мы получим панику во время выполнения, если когда-либо будем использовать в этом случае нестрого сопоставимый тип ключа, точно так же, как мы это сделали бы со встроенным типом `map`.
Мы отказались от статической безопасности типов в пользу проверки во время выполнения.

Могут возникнуть ситуации, когда этого недостаточно и когда мы хотим обеспечить строгую сопоставимость.

Следующее наблюдение позволяет нам сделать именно это, по крайней мере в ограниченной форме: параметры типа не получают выгоды от исключения, которое мы добавили в правило удовлетворения ограничений.

Например, в нашем предыдущем примере параметр типа `P` в функции `g` ограничен типом `any` (который сам по себе сопоставим, но не является строго сопоставимым), и поэтому `P` не удовлетворяет требованию `comparable`.
Мы можем использовать эти знания для создания своего рода утверждения во время компиляции для данного типа `T`:


```Go
type T struct { … }
```

Мы хотим утверждать, что `Т` строго сопоставимо.
Соблазнительно написать что-то вроде:

```Go
// isComparable may be instantiated with any type that supports ==
// including types that are not strictly comparable because of the
// exception for constraint satisfaction.
//
// isComparable может быть создан с любым типом, который поддерживает ==
// включая типы, которые не являются строго сопоставимыми из-за
// исключение для удовлетворения ограничений.
//
func isComparable[_ comparable]() {}

// Tempting but not quite what we want: this declaration is also
// valid for types T that are not strictly comparable.
//
// Заманчиво, но не совсем то, что нам нужно: это объявление также
// действительно для типов T, которые не являются строго сопоставимыми.
//
var _ = isComparable[T] // compile-time error if T does not support ==
```

Фиктивное (пустое) объявление переменной служит нашим "assertion".
Но из-за исключения в правиле удовлетворения ограничений `isComparable[T]` терпит неудачу только в том случае, если `T` вообще не сравним; оно будет успешным, если `T` поддерживает `==`.

Мы можем обойти эту проблему, используя `T` не как аргумент типа, а как ограничение типа:

```Go
func _[P T]() {
	_ = isComparable[P] // P supports == only if T is strictly comparable
}
```

Here is a [passing](/play/p/9i9iEto3TgE) and [failing](/play/p/5d4BeKLevPB) playground example illustrating this mechanism.
Вот пример [прохождения](/play/p/9i9iEto3TgE) и [провала](/play/p/5d4BeKLevPB), иллюстрирующий этот механизм.

PASS:
```Go
package main

func main() {
	// only here to satify the playground
}

// T is strictly comparable.
type T struct {
	x int
}

func isComparable[_ comparable]() {}

func _[P T]() {
	_ = isComparable[P] // P supports == only if T is strictly comparable
}

// This variable declaration always succeeds with a comparable T,
// even if it is not strictly comparable.
var _ = isComparable[T] // compile-time error if `T` does not support `==`
```

FAIL:
```Go
package main

func main() {
	// only here to satify the playground
}

// T is not strictly comparable.
type T struct {
	x any // any is not strictly comparable
}

func isComparable[_ comparable]() {}

func _[P T]() {
	_ = isComparable[P] // P supports == only if T is strictly comparable
}

// This variable declaration always succeeds with a comparable T,
// even if it is not strictly comparable.
var _ = isComparable[T] // compile-time error if `T` does not support `==`
```

## Заключительные наблюдения

Интересно, что за два месяца до выпуска Go 1.18 компилятор реализовал удовлетворение ограничений точно так же, как мы это делаем сейчас в Go 1.20.
Но поскольку в то время удовлетворение ограничений означало реализацию интерфейса, у нас была реализация, несовместимая со спецификацией языка.
Мы были предупреждены об этом факте с помощью [issue #50646](/issue/50646).
Мы были очень близки к релизу, и нам нужно было быстро принять решение.
В отсутствие убедительного решения казалось безопаснее привести реализацию в соответствие со спецификацией.
Год спустя, когда у нас было достаточно времени для рассмотрения различных подходов, похоже, что реализация, которую мы получили, была именно той, которую мы хотели в первую очередь.
Мы прошли полный круг.

Как всегда, сообщите нам, если что-то работает не так, как ожидалось, сообщив о проблемах по адресу [https://go.dev/issue/new](/issue/new).

Спасибо!













# deconstructing-type-parameters

[^](#Оглавление)

Почему сигнатуры функций в пакетах слайсов такие сложные

date: 2023-09-26

by:
- Ian Lance Taylor



## сигнатуры функций пакета слайсов (slices package function signatures)

Функция [`slices.Clone`](https://pkg.go.dev/slices#Clone) довольно проста: она создает копию среза любого типа.

```Go
func Clone[S ~[]E, E any](s S) S {
	return append(s[:0:0], s...)
}
```

Это работает, потому что добавление к срезу с нулевой емкостью выделит новый резервный массив.
Тело функции оказывается короче сигнатуры функции, отчасти потому, что тело короткое, но также и потому, что сигнатура длинная.
В этой статье мы объясним, почему подпись написана именно так.

## Простое клонирование (Simple Clone)

Мы начнем с написания простой универсальной функции `Clone`.
Это не тот, что находится в пакете `slices`.
Мы хотим взять срез любого типа элемента и вернуть новый срез.

```Go
func Clone1[E any](s []E) []E {
	// body omitted
}
```

Дженерик функция `Clone1` имеет единственный параметр типа `E`.
Он принимает один аргумент `s`, который является срезом типа `E`, и возвращает срез того же типа.
Эта подпись понятна любому, кто знаком с дженериками в Go.

Однако есть проблема.
Именованные типы срезов не распространены в Go, но люди их используют.

```Go
// MySlice is a slice of strings with a special String method.
//
// MySlice — это фрагмент строк со специальным методом String.
//
type MySlice []string

// String returns the printable version of a MySlice value.
//
// String возвращает печатную версию значения MySlice.
//
func (s MySlice) String() string {
	return strings.Join(s, "+")
}
```

Допустим, мы хотим сделать копию `MySlice`, а затем получить версию для печати, но со строками в отсортированном порядке.

```Go
func PrintSorted(ms MySlice) string {
	c := Clone1(ms)
	slices.Sort(c)
	return c.String() // FAILS TO COMPILE
}
```

К сожалению, это не работает.
Компилятор сообщает об ошибке:

```
c.String undefined (type []string has no field or method String)

c.String неопределена (тип []string не имеет поля или метода String)
```

Мы можем увидеть проблему, если вручную создадим экземпляр `Clone1`, заменив параметр типа аргументом типа.

```Go
func InstantiatedClone1(s []string) []string
```

The [Go assignment rules](https://go.dev/ref/spec#Assignability) allow us to pass a value of type `MySlice` to a parameter of type `[]string`, so calling `Clone1` is fine.
But `Clone1` will return a value of type `[]string`, not a value of type `MySlice`.
The type `[]string` doesn't have a `String` method, so the compiler reports an error.

[Правила Go](https://go.dev/ref/spec#Assignability) позволяют нам передавать значение типа `MySlice` в параметр типа `[]string`, поэтому вызов `Clone1` вполне допустим.
Но `Clone1` вернет значение типа `[]string`, а не значение типа `MySlice`.
Тип `[]string` не имеет метода `String`, поэтому компилятор сообщает об ошибке.

## Flexible Clone

To fix this problem, we have to write a version of `Clone` that returns the same type as its argument.
If we can do that, then when we call `Clone` with a value of type `MySlice`, it will return a result of type `MySlice`.

We know that it has to look something like this.

```Go
func Clone2[S ?](s S) S // INVALID
```

This `Clone2` function returns a value that is the same type as its argument.

Here I've written the constraint as `?`, but that's just a placeholder.
To make this work we need to write a constraint that will let us write the body of the function.
For `Clone1` we could just use a constraint of `any` for the element type.
For `Clone2` that won't work: we want to require that `s` be a slice type.

Since we know we want a slice, the constraint of `S` has to be a slice.
We don't care what the slice element type is, so let's just call it `E`, as we did with `Clone1`.

```Go
func Clone3[S []E](s S) S // INVALID
```

This is still invalid, because we haven't declared `E`.
The type argument for `E` can be any type, which means it also has to be a type parameter itself.
Since it can be any type, its constraint is `any`.

```Go
func Clone4[S []E, E any](s S) S
```

This is getting close, and at least it will compile, but we're not quite there yet.
If we compile this version, we get an error when we call `Clone4(ms)`.

```
MySlice does not satisfy []string (possibly missing ~ for []string in []string)
```

The compiler is telling us that we can't use the type argument `MySlice` for the type parameter `S`, because `MySlice` does not satisfy the constraint `[]E`.
That's because `[]E` as a constraint only permits a slice type literal, like `[]string`.
It doesn't permit a named type like `MySlice`.

## Underlying type constraints

As the error message hints, the answer is to add a `~`.

```Go
func Clone5[S ~[]E, E any](s S) S
```

To repeat, writing type parameters and constraints `[S []E, E any]` means that the type argument for `S` can be any unnamed slice type, but it can't be a named type defined as a slice literal.
Writing `[S ~[]E, E any]`, with a `~`, means that the type argument for `S` can be any type whose underlying type is a slice type.

For any named type `type T1 T2` the underlying type of `T1` is the underlying type of `T2`.
The underlying type of a predeclared type like `int` or a type literal like `[]string` is just the type itself.
For the exact details, [see the language spec](https://go.dev/ref/spec#Underlying_types).
In our example, the underlying type of `MySlice` is `[]string`.

Since the underlying type of `MySlice` is a slice, we can pass an argument of type `MySlice` to `Clone5`.
As you may have noticed, the signature of `Clone5` is the same as the signature of `slices.Clone`.
We've finally gotten to where we want to be.

Before we move on, let's discuss why the Go syntax requires a `~`.
It might seem that we would always want to permit passing `MySlice`, so why not make that the default?
Or, if we need to support exact matching, why not flip things around, so that a constraint of `[]E` permits a named type while a constraint of, say, `=[]E`, only permits slice type literals?

To explain this, let's first observe that a type parameter list like `[T ~MySlice]` doesn't make sense.
That's because `MySlice` is not the underlying type of any other type.
For instance, if we have a definition like `type MySlice2 MySlice`, the underlying type of `MySlice2` is `[]string`, not `MySlice`.
So either `[T ~MySlice]` would permit no types at all, or it would be the same as `[T MySlice]` and only match `MySlice`.
Either way, `[T ~MySlice]` isn't useful.
To avoid this confusion, the language prohibits `[T ~MySlice]`, and the compiler produces an error like

```
invalid use of ~ (underlying type of MySlice is []string)
```

If Go didn't require the tilde, so that `[S []E]` would match any type whose underlying type is `[]E`, then we would have to define the meaning of `[S MySlice]`.

We could prohibit `[S MySlice]`, or we could say that `[S MySlice]` only matches `MySlice`, but either approach runs into trouble with predeclared types.
A predeclared type, like `int` is its own underlying type.  We want to permit people to be able to write constraints that accept any type argument whose underlying type is `int`.
In the language today, they can do that by writing `[T ~int]`.
If we don't require the tilde we would still need a way to say "any type whose underlying type is `int`".
The natural way to say that would be `[T int]`.
That would mean that `[T MySlice]` and `[T int]` would behave differently, although they look very similar.

We could perhaps say that `[S MySlice]` matches any type whose underlying type is the underlying type of `MySlice`, but that makes `[S MySlice]` unnecessary and confusing.

We think it's better to require the `~` and be very clear about when we are matching the underlying type rather than the type itself.

## Type inference

Now that we've explained the signature of `slices.Clone`, let's see how actually using `slices.Clone` is simplified by type inference.
Remember, the signature of `Clone` is

```Go
func Clone[S ~[]E, E any](s S) S
```

A call of `slices.Clone` will pass a slice to the parameter `s`.
Simple type inference will let the compiler infer that the type argument for the type parameter `S` is the type of the slice being passed to `Clone`.
Type inference is then powerful enough to see that the type argument for `E` is the element type of the type argument passed to `S`.

This means that we can write

```Go
	c := Clone(ms)
```

without having to write

```Go
	c := Clone[MySlice, string](ms)
```

If we refer to `Clone` without calling it, we do have to specify a type argument for `S`, as the compiler has nothing it can use to infer it.
Fortunately, in that case, type inference is able to infer the type argument for `E` from the argument for `S`, and we don't have to specify it separately.

That is, we can write

```Go
	myClone := Clone[MySlice]
```

without having to write

```Go
	myClone := Clone[MySlice, string]
```

## Deconstructing type parameters

The general technique we've used here, in which we define one type parameter `S` using another type parameter `E`, is a way to deconstruct types in generic function signatures.
By deconstructing a type, we can name, and constrain, all aspects of the type.

For example, here is the signature for `maps.Clone`.

```Go
func Clone[M ~map[K]V, K comparable, V any](m M) M
```

Just as with `slices.Clone`, we use a type parameter for the type of the parameter `m`, and then deconstruct the type using two other type parameters `K` and `V`.

In `maps.Clone` we constrain `K` to be comparable, as is required for a map key type.
We can constrain the component types any way we like.

```Go
func WithStrings[S ~[]E, E interface { String() string }](s S) (S, []string)
```

This says that the argument of `WithStrings` must be a slice type for which the element type has a `String` method.

Since all Go types can be built up from component types, we can always use type parameters to deconstruct those types and constrain them as we like.





























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

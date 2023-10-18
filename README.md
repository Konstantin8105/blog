# Блог

# Оглавление

* Go выйграл в 2010 Bossie Award [bossie](#bossie)
* Встроенный фаззинг Go [fuzz-beta](#fuzz-beta)
* Go Doc комментарии [comment](#comment)
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




























# comment

[^](#Оглавление)

Go Doc комментарии
date: 2022-06-01T00:00:00Z

«Комментарии к документу» — это комментарии, которые появляются непосредственно перед объявлениями package, const, func, type и var верхнего уровня без промежуточных символов новой строки.
Каждое экспортированное (с заглавной буквы) имя должно иметь комментарий в документе.

Пакеты [go/doc](/pkg/go/doc) и [go/doc/comment](/pkg/go/doc/comment) предоставляют возможность извлекать документацию из исходного кода Go, а различные инструменты позволяют использование этого функционала.
Команда [`go` `doc`](/cmd/go#hdr-Show_documentation_for_package_or_symbol) ищет и печатает комментарий документа для данного пакета или символа.
(Символ — это константа, функция, тип или переменная верхнего уровня.) Веб-сервер [pkg.go.dev](https://pkg.go.dev/) отображает документацию для общедоступных пакетов Go (когда их лицензии разрешают такое использование).
Программа, обслуживающая этот сайт, — [golang.org/x/pkgsite/cmd/pkgsite](https://pkg.go.dev/golang.org/x/pkgsite/cmd/pkgsite), которую также можно запустить локально для просматривать документацию для частных модулей или без подключения к Интернету.
Языковой сервер [gopls](https://pkg.go.dev/golang.org/x/tools/gopls) предоставляет документацию при редактировании исходных файлов Go в IDE.

Остальная часть этой страницы описывает, как писать комментарии к документам Go.


## Пакеты (Packages)

Каждый пакет должен иметь комментарий, представляющий пакет.

Он предоставляет информацию, относящуюся к пакету в целом, и в целом определяет ожидания от пакета.
В комментариях к пакету, особенно в больших пакетах, может быть полезно дать краткий обзор наиболее важных частей API, при необходимости ссылаясь на другие комментарии к документации.

Если пакет простой, комментарий к пакету может быть кратким.
Например:

```Go
	// Package path implements utility routines for manipulating slash-separated
	// paths.
	//
	// The path package should only be used for paths separated by forward
	// slashes, such as the paths in URLs. This package does not deal with
	// Windows paths with drive letters or backslashes; to manipulate
	// operating system paths, use the [path/filepath] package.
	package path
```

Квадратные скобки в `[path/filepath]` создают [ссылку на документацию](#links).

Как видно из этого примера, комментарии Go doc используют полные предложения.
Для комментария к пакету это означает, что [первое предложение](/pkg/go/doc/#Package.Synopsis) начинается с “Package <имя>”.

Для пакетов из нескольких файлов комментарий пакета должен находиться только в одном исходном файле.
Если комментарии к пакету есть в нескольких файлах, они объединяются в один большой комментарий для всего пакета.

## Команды (Commands)

Комментарий пакета для команды аналогичен, но он описывает поведение программы, а не символы Go в пакете.
Первое предложение традиционно начинается с названия самой программы, написанного с заглавной буквы, поскольку оно находится в начале предложения.
Например, вот сокращенная версия комментария пакета для [gofmt](/cmd/gofmt):

```Go
	/*
	Gofmt formats Go programs.
	It uses tabs for indentation and blanks for alignment.
	Alignment assumes that an editor is using a fixed-width font.

	Without an explicit path, it processes the standard input. Given a file,
	it operates on that file; given a directory, it operates on all .go files in
	that directory, recursively. (Files starting with a period are ignored.)
	By default, gofmt prints the reformatted sources to standard output.

	Usage:

		gofmt [flags] [path ...]

	The flags are:

		-d
			Do not print reformatted sources to standard output.
			If a file's formatting is different than gofmt's, print diffs
			to standard output.
		-w
			Do not print reformatted sources to standard output.
			If a file's formatting is different from gofmt's, overwrite it
			with gofmt's version. If an error occurred during overwriting,
			the original file is restored from an automatic backup.

	When gofmt reads from standard input, it accepts either a full Go program
	or a program fragment. A program fragment must be a syntactically
	valid declaration list, statement list, or expression. When formatting
	such a fragment, gofmt preserves leading indentation as well as leading
	and trailing spaces, so that individual sections of a Go program can be
	formatted by piping them through gofmt.
	*/
	package main
```

Начало комментария записывается с помощью [семантического перевода строки](https://rhodesmill.org/brandon/2012/one-sentence-per-line/), при котором каждое новое предложение или длинная фраза находится на отдельной строке, что может облегчить чтение различий по мере развития кода и комментариев.

Последующие абзацы не соответствуют этому соглашению и были перенесены вручную.

Все, что лучше для вашей кодовой базы, подойдет.
В любом случае `go` `doc` и `pkgsite` переносят текст комментария к документу при его печати.
Например:

```
	$ go doc gofmt
	Gofmt formats Go programs. It uses tabs for indentation and blanks for
	alignment. Alignment assumes that an editor is using a fixed-width font.

	Without an explicit path, it processes the standard input. Given a file, it
	operates on that file; given a directory, it operates on all .go files in that
	directory, recursively. (Files starting with a period are ignored.) By default,
	gofmt prints the reformatted sources to standard output.

	Usage:

		gofmt [flags] [path ...]

	The flags are:

		-d
			Do not print reformatted sources to standard output.
			If a file's formatting is different than gofmt's, print diffs
			to standard output.
	...
```

Строки с отступом рассматриваются как предварительно отформатированный текст:
они не переворачиваются и печатаются кодовым шрифтом в презентациях HTML и Markdown.
(Подробности приведены в разделе [Синтаксис](#syntax) ниже.)

## Типы (Types)

Комментарий к документации типа должен объяснять, что представляет или предоставляет каждый экземпляр этого типа.
Если API простой, комментарий к документу может быть довольно коротким.
Например:

```Go
	package zip

	// A Reader serves content from a ZIP archive.
	type Reader struct {
		...
	}
```

По умолчанию программисты должны ожидать, что тип безопасен для использования только одной горутиной одновременно.
Если тип предоставляет более строгие гарантии, они должны быть указаны в комментарии к документу.
Например:

```Go
	package regexp

	// Regexp is the representation of a compiled regular expression.
	// A Regexp is safe for concurrent use by multiple goroutines,
	// except for configuration methods, such as Longest.
	type Regexp struct {
		...
	}
```

Типы Go также должны стремиться к тому, чтобы нулевое значение имело полезное значение.
Если это не очевидно, это значение должно быть задокументировано. Например:

```Go
	package bytes

	// A Buffer is a variable-sized buffer of bytes with Read and Write methods.
	// The zero value for Buffer is an empty buffer ready to use.
	type Buffer struct {
		...
	}
```

Для структуры с экспортированными полями либо комментарий к документу, либо комментарии к каждому полю должны объяснять значение каждого экспортированного поля.
Например, комментарий к документу этого типа объясняет поля:

```Go
	package io

	// A LimitedReader reads from R but limits the amount of
	// data returned to just N bytes. Each call to Read
	// updates N to reflect the new amount remaining.
	// Read returns EOF when N <= 0.
	type LimitedReader struct {
		R   Reader // underlying reader
		N   int64  // max bytes remaining
	}
```

Напротив, комментарий документа этого типа оставляет пояснения к комментариям для каждого поля:

```Go
	package comment

	// A Printer is a doc comment printer.
	// The fields in the struct can be filled in before calling
	// any of the printing methods
	// in order to customize the details of the printing process.
	type Printer struct {
		// HeadingLevel is the nesting level used for
		// HTML and Markdown headings.
		// If HeadingLevel is zero, it defaults to level 3,
		// meaning to use <h3> and ###.
		HeadingLevel int
		...
	}
```

Как и в случае с пакетами (выше) и функциями (ниже), комментарии к документам для типов начинаются с полных предложений, называющих объявленный символ.
Явная тема часто делает формулировку более понятной и упрощает поиск текста как на веб-странице, так и в командной строке.
Например:

```
	$ go doc -all regexp | grep pairs
	pairs within the input string: result[2*n:2*n+2] identifies the indexes
	    FindReaderSubmatchIndex returns a slice holding the index pairs identifying
	    FindStringSubmatchIndex returns a slice holding the index pairs identifying
	    FindSubmatchIndex returns a slice holding the index pairs identifying the
	$
```

## Функции (Funcs)

Комментарий к документации функции должен объяснять, что возвращает функция или, для функций, вызываемых для побочных эффектов, что она делает.
На именованные параметры и результаты можно ссылаться непосредственно в комментарии, без какого-либо специального синтаксиса, например обратных кавычек.
(Следствием этого соглашения является то, что обычно избегаются такие имена, как `a`, которые можно принять за обычные слова.)
Например:


```Go
	package strconv

	// Quote returns a double-quoted Go string literal representing s.
	// The returned string uses Go escape sequences (\t, \n, \xFF, \u0100)
	// for control characters and non-printable characters as defined by IsPrint.
	func Quote(s string) string {
		...
	}
```

И:

```Go
	package os

	// Exit causes the current program to exit with the given status code.
	// Conventionally, code zero indicates success, non-zero an error.
	// The program terminates immediately; deferred functions are not run.
	//
	// For portability, the status code should be in the range [0, 125].
	func Exit(code int) {
		...
	}
```

В комментариях к документации обычно используется фраза "сообщает ли"(“reports whether”) для описания функций, возвращающих логическое значение.
Фраза "или нет"(“or not”) излишняя.
Например:

```Go
	package strings

	// HasPrefix reports whether the string s begins with prefix.
	func HasPrefix(s, prefix string) bool
```

Если комментарий к документу должен объяснить несколько результатов, присвоение имен результатам может сделать комментарий к документу более понятным, даже если имена не используются в теле функции.
Например:

```Go
	package io

	// Copy copies from src to dst until either EOF is reached
	// on src or an error occurs. It returns the total number of bytes
	// written and the first error encountered while copying, if any.
	//
	// A successful Copy returns err == nil, not err == EOF.
	// Because Copy is defined to read from src until EOF, it does
	// not treat an EOF from Read as an error to be reported.
	func Copy(dst Writer, src Reader) (n int64, err error) {
		...
	}
```

И наоборот, когда результаты не нужно называть в комментарии к документу, они обычно также опускаются в коде, как в приведенном выше примере `Цитата`(`Quote`), чтобы не загромождать презентацию.

Все эти правила применимы как к простым функциям, так и к методам.
Для методов использование одного и того же имени получателя позволяет избежать ненужных изменений при перечислении всех методов одного типа:

```
	$ go doc bytes.Buffer
	package bytes // import "bytes"

	type Buffer struct {
		// Has unexported fields.
	}
	    A Buffer is a variable-sized buffer of bytes with Read and Write methods.
	    The zero value for Buffer is an empty buffer ready to use.

	func NewBuffer(buf []byte) *Buffer
	func NewBufferString(s string) *Buffer
	func (b *Buffer) Bytes() []byte
	func (b *Buffer) Cap() int
	func (b *Buffer) Grow(n int)
	func (b *Buffer) Len() int
	func (b *Buffer) Next(n int) []byte
	func (b *Buffer) Read(p []byte) (n int, err error)
	func (b *Buffer) ReadByte() (byte, error)
	...
```

Этот пример также показывает, что функции верхнего уровня, возвращающие тип `T` или указатель `*T`, возможно, с дополнительным результатом ошибки, показаны рядом с типом `T` и его методами в предположении, что это тип `T` конструктора.

По умолчанию программисты могут предполагать, что функцию верхнего уровня можно безопасно вызывать из нескольких горутин; этот факт не нуждается в прямом указании.

С другой стороны, как отмечалось в предыдущем разделе, использование экземпляра типа любым способом, включая вызов метода, обычно считается ограниченным одновременно одной горутиной.
Если методы, безопасные для одновременного использования, не описаны в комментариях к документации типа, их следует задокументировать в комментариях для каждого метода.
Например:


```Go
	package sql

	// Close returns the connection to the connection pool.
	// All operations after a Close will return with ErrConnDone.
	// Close is safe to call concurrently with other operations and will
	// block until all other operations finish. It may be useful to first
	// cancel any used context and then call Close directly after.
	func (c *Conn) Close() error {
		...
	}
```

Обратите внимание, что комментарии к документации по функциям и методам сосредоточены на том, что возвращает или делает операция, и подробно описывают, что необходимо знать вызывающему объекту.
Особые случаи могут быть особенно важны для документирования.
Например:

```Go
	package math

	// Sqrt returns the square root of x.
	//
	// Special cases are:
	//
	//	Sqrt(+Inf) = +Inf
	//	Sqrt(±0) = ±0
	//	Sqrt(x < 0) = NaN
	//	Sqrt(NaN) = NaN
	func Sqrt(x float64) float64 {
		...
	}
```

Комментарии к документации не должны объяснять внутренние детали, такие как алгоритм, используемый в текущей реализации.
Их лучше оставить для комментариев внутри тела функции.
Может оказаться целесообразным указать асимптотические временные или пространственные границы, когда эта деталь особенно важна для вызывающей стороны.
Например:


```Go
	package sort

	// Sort sorts data in ascending order as determined by the Less method.
	// It makes one call to data.Len to determine n and O(n*log(n)) calls to
	// data.Less and data.Swap. The sort is not guaranteed to be stable.
	func Sort(data Interface) {
		...
	}
```

Поскольку в этом комментарии к документу не упоминается, какой алгоритм сортировки используется, в будущем легче изменить реализацию, чтобы использовать другой алгоритм.

## Постоянные (Consts)

Синтаксис объявлений Go позволяет группировать объявления, и в этом случае один комментарий к документу может представлять группу связанных констант, при этом отдельные константы документируются только короткими комментариями в конце строки.
Например:


```Go
	package scanner // import "text/scanner"

	// The result of Scan is one of these tokens or a Unicode character.
	const (
		EOF = -(iota + 1)
		Ident
		Int
		Float
		Char
		...
	)
```

Иногда группа вообще не нуждается в комментариях к документу. Например:

```Go
	package unicode // import "unicode"

	const (
		MaxRune         = '\U0010FFFF' // maximum valid Unicode code point.
		ReplacementChar = '\uFFFD'     // represents invalid code points.
		MaxASCII        = '\u007F'     // maximum ASCII value.
		MaxLatin1       = '\u00FF'     // maximum Latin-1 value.
	)
```

С другой стороны, разгруппированные константы обычно требуют полного комментария к документу, начиная с полного предложения. Например:

```Go
	package unicode

	// Version is the Unicode edition from which the tables are derived.
	const Version = "13.0.0"
```

Типизированные константы отображаются рядом с объявлением их типа, и в результате комментарий к документу группы const часто опускается в пользу комментария к документу типа.
Например:

```Go
	package syntax

	// An Op is a single regular expression operator.
	type Op uint8

	const (
		OpNoMatch        Op = 1 + iota // matches no strings
		OpEmptyMatch                   // matches empty string
		OpLiteral                      // matches Runes sequence
		OpCharClass                    // matches Runes interpreted as range pair list
		OpAnyCharNotNL                 // matches any character except newline
		...
	)
```

(Смотри [pkg.go.dev/regexp/syntax#Op](https://pkg.go.dev/regexp/syntax#Op) для презентации HTML.)

## Переменные (Vars)

The conventions for variables are the same as those for constants.
For example, here is a set of grouped variables:

```Go
	package fs

	// Generic file system errors.
	// Errors returned by file systems can be tested against these errors
	// using errors.Is.
	var (
		ErrInvalid    = errInvalid()    // "invalid argument"
		ErrPermission = errPermission() // "permission denied"
		ErrExist      = errExist()      // "file already exists"
		ErrNotExist   = errNotExist()   // "file does not exist"
		ErrClosed     = errClosed()     // "file already closed"
	)
```

And a single variable:

```Go
	package unicode

	// Scripts is the set of Unicode script tables.
	var Scripts = map[string]*RangeTable{
		"Adlam":                  Adlam,
		"Ahom":                   Ahom,
		"Anatolian_Hieroglyphs":  Anatolian_Hieroglyphs,
		"Arabic":                 Arabic,
		"Armenian":               Armenian,
		...
	}
```

## Syntax

Go doc comments are written in a simple syntax that supports paragraphs, headings, links, lists, and preformatted code blocks.
To keep comments lightweight and readable in source files, there is no support for complex features like font changes or raw HTML.
Markdown aficionados can view the syntax as a simplified subset of Markdown.

The standard formatter [gofmt](/cmd/gofmt) reformats doc comments to use a canonical formatting for each of these features.
Gofmt aims for readability and user control over how comments are written in source code but will adjust presentation to make the semantic meaning of a particular comment clearer, analogous to reformatting `1+2 * 3` to `1 + 2*3` in ordinary source code.

Directive comments such as `//go:generate` are not considered part of a doc comment and are omitted from rendered documentation.
Gofmt moves directive comments to the end of the doc comment, preceded by a blank line.
For example:

```Go
	package regexp

	// An Op is a single regular expression operator.
	//
	//go:generate stringer -type Op -trimprefix Op
	type Op uint8
```

A directive comment is a line matching the regular expression `//(line |extern |export |[a-z0-9]+:[a-z0-9])`.
Tools that define their own directives should use the form `//toolname:directive`.

Gofmt removes leading and trailing blank lines in doc comments.
If all lines in a doc comment begin with the same sequence of spaces and tabs, gofmt removes that prefix.

### Paragraphs

A paragraph is a span of unindented non-blank lines.
We've already seen many examples of paragraphs.

A pair of consecutive backticks (\` U+0060) is interpreted as a Unicode left quote (“ U+201C), and a pair of consecutive single quotes (\' U+0027) is interpreted as a Unicode right quote (” U+201D).

Gofmt preserves line breaks in paragraph text: it does not rewrap the text.
This allows the use of [semantic linefeeds](https://rhodesmill.org/brandon/2012/one-sentence-per-line/), as seen earlier.
Gofmt replaces duplicated blank lines between paragraphs with a single blank line.
Gofmt also reformats consecutive backticks or single quotes to their Unicode interpretations.

### Headings

A heading is a line beginning with a number sign (U+0023) and then a space and the heading text.
To be recognized as a heading, the line must be unindented and set off from adjacent paragraph text by blank lines.

For example:

``` Go
	// Package strconv implements conversions to and from string representations
	// of basic data types.
	//
	// # Numeric Conversions
	//
	// The most common numeric conversions are [Atoi] (string to int) and [Itoa] (int to string).
	...
	package strconv
```

On the other hand:

```Go
	// #This is not a heading, because there is no space.
	//
	// # This is not a heading,
	// # because it is multiple lines.
	//
	// # This is not a heading,
	// because it is also multiple lines.
	//
	// The next paragraph is not a heading, because there is no additional text:
	//
	// #
	//
	// In the middle of a span of non-blank lines,
	// # this is not a heading either.
	//
	//     # This is not a heading, because it is indented.
```

The # syntax was added in Go 1.19.
Before Go 1.19, headings were identified implicitly by single-line paragraphs satisfying certain conditions, most notably the lack of any terminating punctuation.

Gofmt reformats [lines treated as implicit headings](https://github.com/golang/proposal/blob/master/design/51082-godocfmt.md#headings) by earlier versions of Go to use # headings instead.
If the reformatting is not appropriate—that is, if the line was not meant to be a heading—the easiest way to make it a paragraph is to introduce terminating punctuation such as a period or colon, or to break it into two lines.

### Links

A span of unindented non-blank lines defines link targets when every line is of the form “[Text]: URL”.
In other text in the same doc comment, “[Text]” represents a link to URL using the given text—in HTML, \<a href="URL">Text\</a>.
For example:

```Go
	// Package json implements encoding and decoding of JSON as defined in
	// [RFC 7159]. The mapping between JSON and Go values is described
	// in the documentation for the Marshal and Unmarshal functions.
	//
	// For an introduction to this package, see the article
	// “[JSON and Go].”
	//
	// [RFC 7159]: https://tools.ietf.org/html/rfc7159
	// [JSON and Go]: https://golang.org/doc/articles/json_and_go.html
	package json
```

By keeping URLs in a separate section, this format only minimally interrupts the flow of the actual text.  It also roughly matches the Markdown [shortcut reference link format](https://spec.commonmark.org/0.30/#shortcut-reference-link), without the optional title text.

If there is no corresponding URL declaration, then (except for doc links, described in the next section) “[Text]” is not a hyperlink, and the square brackets are preserved when displayed.
Each doc comment is considered independently:
link target definitions in one comment do not affect other comments.

Although link target definition blocks may be interleaved with ordinary paragraphs, gofmt moves all link target definitions to the end of the doc comment, in up to two blocks: first a block containing all the link targets that are referenced in the comment, and then a block containing all the targets _not_ referenced in the comment.
The separate block makes unused targets easy to notice and fix (in case the links or the definitions have typos) or to delete (in case the definitions are no longer needed).

Plain text that is recognized as a URL is automatically linked in HTML renderings.

### Doc links

Doc links are links of the form “[Name1]” or “[Name1.Name2]” to refer to exported identifiers in the current package, or “[pkg]”, “[pkg.Name1]”, or “[pkg.Name1.Name2]” to refer to identifiers in other packages.

For example:

```Go
	package bytes

	// ReadFrom reads data from r until EOF and appends it to the buffer, growing
	// the buffer as needed. The return value n is the number of bytes read. Any
	// error except [io.EOF] encountered during the read is also returned. If the
	// buffer becomes too large, ReadFrom will panic with [ErrTooLarge].
	func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error) {
		...
	}
```

The bracketed text for a symbol link can include an optional leading star, making it easy to refer to pointer types, such as `*bytes.Buffer`.

When referring to other packages, “pkg” can be either a full import path or the assumed package name of an existing import. The assumed package name is either the identifier in a renamed import or else [the name assumed by goimports](https://pkg.go.dev/golang.org/x/tools/internal/imports#ImportPathToAssumedName).
(Goimports inserts renamings when that assumption is not correct, so this rule should work for essentially all Go code.)
For example, if the current package imports encoding/json, then “[json.Decoder]” can be written in place of “[encoding/json.Decoder]”
to link to the docs for encoding/json's Decoder.
If different source files in a package import different packages using the same name, then the shorthand is ambiguous and cannot be used.

A “pkg” is only assumed to be a full import path if it starts with a domain name (a path element with a dot) or is one of the packages from the standard library (“[os]”, “[encoding/json]”, and so on).
For example, `[os.File]` and `[example.com/sys.File]` are documentation links (the latter will be a broken link), but `[os/sys.File]` is not, because there is no os/sys package in the standard library.

To avoid problems with maps, generics, and array types, doc links must be both preceded and followed by punctuation, spaces, tabs, or the start or end of a line.
For example, the text “map[ast.Expr]TypeAndValue” does not contain a doc link.

### Lists

A list is a span of indented or blank lines (which would otherwise be a code block, as described in the next section) in which the first indented line begins with a bullet list marker or a numbered list marker.

A bullet list marker is a star, plus, dash, or Unicode bullet `(*, +, -, •; U+002A, U+002B, U+002D, U+2022)` followed by a space or tab and then text.
In a bullet list, each line beginning with a bullet list marker starts a new list item.

For example:

```Go
	package url

	// PublicSuffixList provides the public suffix of a domain. For example:
	//   - the public suffix of "example.com" is "com",
	//   - the public suffix of "foo1.foo2.foo3.co.uk" is "co.uk", and
	//   - the public suffix of "bar.pvt.k12.ma.us" is "pvt.k12.ma.us".
	//
	// Implementations of PublicSuffixList must be safe for concurrent use by
	// multiple goroutines.
	//
	// An implementation that always returns "" is valid and may be useful for
	// testing but it is not secure: it means that the HTTP server for foo.com can
	// set a cookie for bar.com.
	//
	// A public suffix list implementation is in the package
	// golang.org/x/net/publicsuffix.
	type PublicSuffixList interface {
		...
	}
```

A numbered list marker is a decimal number of any length followed by a period or right parenthesis, then a space or tab, and then text.
In a numbered list, each line beginning with a number list marker starts a new list item.
Item numbers are left as is, never renumbered.

For example:

```Go
	package path

	// Clean returns the shortest path name equivalent to path
	// by purely lexical processing. It applies the following rules
	// iteratively until no further processing can be done:
	//
	//  1. Replace multiple slashes with a single slash.
	//  2. Eliminate each . path name element (the current directory).
	//  3. Eliminate each inner .. path name element (the parent directory)
	//     along with the non-.. element that precedes it.
	//  4. Eliminate .. elements that begin a rooted path:
	//     that is, replace "/.." by "/" at the beginning of a path.
	//
	// The returned path ends in a slash only if it is the root "/".
	//
	// If the result of this process is an empty string, Clean
	// returns the string ".".
	//
	// See also Rob Pike, “[Lexical File Names in Plan 9].”
	//
	// [Lexical File Names in Plan 9]: https://9p.io/sys/doc/lexnames.html
	func Clean(path string) string {
		...
	}
```

List items only contain paragraphs, not code blocks or nested lists.  This avoids any space-counting subtlety as well as questions about how many spaces a tab counts for in inconsistent indentation.

Gofmt reformats bullet lists to use a dash as the bullet marker, two spaces of indentation before the dash, and four spaces of indentation for continuation lines.

Gofmt reformats numbered lists to use a single space before the number, a period after the number, and again four spaces of indentation for continuation lines.

Gofmt preserves but does not require a blank line between a list and the preceding paragraph.
It inserts a blank line between a list and the following paragraph or heading.

### Code blocks

A code block is a span of indented or blank lines not starting with a bullet list marker or numbered list marker.
It is rendered as preformatted text (a \<pre> block in HTML).

Code blocks often contain Go code. For example:

```Go
	package sort

	// Search uses binary search...
	//
	// As a more whimsical example, this program guesses your number:
	//
	//	func GuessingGame() {
	//		var s string
	//		fmt.Printf("Pick an integer from 0 to 100.\n")
	//		answer := sort.Search(100, func(i int) bool {
	//			fmt.Printf("Is your number <= %d? ", i)
	//			fmt.Scanf("%s", &s)
	//			return s != "" && s[0] == 'y'
	//		})
	//		fmt.Printf("Your number is %d.\n", answer)
	//	}
	func Search(n int, f func(int) bool) int {
		...
	}
```

Of course, code blocks also often contain preformatted text besides code. For example:

```Go
	package path

	// Match reports whether name matches the shell pattern.
	// The pattern syntax is:
	//
	//	pattern:
	//		{ term }
	//	term:
	//		'*'         matches any sequence of non-/ characters
	//		'?'         matches any single non-/ character
	//		'[' [ '^' ] { character-range } ']'
	//		            character class (must be non-empty)
	//		c           matches character c (c != '*', '?', '\\', '[')
	//		'\\' c      matches character c
	//
	//	character-range:
	//		c           matches character c (c != '\\', '-', ']')
	//		'\\' c      matches character c
	//		lo '-' hi   matches character c for lo <= c <= hi
	//
	// Match requires pattern to match all of name, not just a substring.
	// The only possible returned error is [ErrBadPattern], when pattern
	// is malformed.
	func Match(pattern, name string) (matched bool, err error) {
		...
	}
```

Gofmt indents all lines in a code block by a single tab, replacing any other indentation the non-blank lines have in common.
Gofmt also inserts a blank line before and after each code block, distinguishing the code block clearly from the surrounding paragraph text.

## Common mistakes and pitfalls

The rule that any span of indented or blank lines in a doc comment is rendered as a code block dates to the earliest days of Go.
Unfortunately, the lack of support for doc comments in gofmt has led to many existing comments that use indentation without meaning to create a code block.

For example, this unindented list has always been interpreted by godoc as a three-line paragraph followed by a one-line code block:

```Go
	package http

	// cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	// 1) On Read error or close, the stop func is called.
	// 2) On Read failure, if reqDidTimeout is true, the error is wrapped and
	//    marked as net.Error that hit its timeout.
	type cancelTimerBody struct {
		...
	}
```

This always rendered in `go` `doc` as:

```
	cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	1) On Read error or close, the stop func is called. 2) On Read failure,
	if reqDidTimeout is true, the error is wrapped and

	    marked as net.Error that hit its timeout.
``

Similarly, the command in this comment is a one-line paragraph
followed by a one-line code block:

```Go
	package smtp

	// localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:
	//
	// go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \
	//     --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h
	var localhostCert = []byte(`...`)
```

This rendered in `go` `doc` as:

```
	localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:

	go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \

	    --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h
```

And this comment is a two-line paragraph (the second line is “{”), followed by a six-line indented code block and a one-line paragraph (“}”).

```Go
	// On the wire, the JSON will look something like this:
	// {
	//	"kind":"MyAPIObject",
	//	"apiVersion":"v1",
	//	"myPlugin": {
	//		"kind":"PluginA",
	//		"aOption":"foo",
	//	},
	// }
```

And this rendered in `go` `doc` as:

```
	On the wire, the JSON will look something like this: {

	    "kind":"MyAPIObject",
	    "apiVersion":"v1",
	    "myPlugin": {
	    	"kind":"PluginA",
	    	"aOption":"foo",
	    },

	}
```

Another common mistake was an unindented Go function definition or block statement, similarly bracketed by “{” and “}”.

The introduction of doc comment reformatting in Go 1.19's gofmt makes mistakes like these more visible by adding blank lines around the code blocks.

Analysis in 2022 found that only 3% of doc comments in public Go modules were reformatted at all by the draft Go 1.19 gofmt.
Limiting ourselves to those comments, about 87% of gofmt's reformattings preserved the structure that a person would infer from reading the comment; about 6% were tripped up by these kinds of unindented lists, unindented multiline shell commands, and unindented brace-delimited code blocks.

Based on this analysis, the Go 1.19 gofmt applies a few heuristics to merge unindented lines into an adjacent indented list or code block.
With those adjustments, the Go 1.19 gofmt reformats the above examples to:

```Go
	// cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	//  1. On Read error or close, the stop func is called.
	//  2. On Read failure, if reqDidTimeout is true, the error is wrapped and
	//     marked as net.Error that hit its timeout.

	// localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:
	//
	//	go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \
	//	    --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h

	// On the wire, the JSON will look something like this:
	//
	//	{
	//		"kind":"MyAPIObject",
	//		"apiVersion":"v1",
	//		"myPlugin": {
	//			"kind":"PluginA",
	//			"aOption":"foo",
	//		},
	//	}
```

This reformatting makes the meaning clearer as well as making the doc comments render correctly in earlier versions of Go.
If the heuristic ever makes a bad decision, it can be overridden by inserting a blank line to clearly separate the paragraph text from non-paragraph text.

Even with these heuristics, other existing comments will need manual adjustment to correct their rendering.
The most common mistake is indenting a wrapped unindented line of text.
For example:

```Go
	// TODO Revisit this design. It may make sense to walk those nodes
	//      only once.

	// According to the document:
	// "The alignment factor (in bytes) that is used to align the raw data of sections in
	//  the image file. The value should be a power of 2 between 512 and 64 K, inclusive."
```

In both of these, the last line is indented, making it a code block.
The fix is to unindent the lines.

Another common mistake is not indenting a wrapped indented line of a list or code block.
For example:

```Go
	// Uses of this error model include:
	//
	//   - Partial errors. If a service needs to return partial errors to the
	// client,
	//     it may embed the `Status` in the normal response to indicate the
	// partial
	//     errors.
	//
	//   - Workflow errors. A typical workflow has multiple steps. Each step
	// may
	//     have a `Status` message for error reporting.
```

The fix is to indent the wrapped lines.

Go doc comments do not support nested lists, so gofmt reformats

```Go
	// Here is a list:
	//
	//  - Item 1.
	//    * Subitem 1.
	//    * Subitem 2.
	//  - Item 2.
	//  - Item 3.
```

to

```Go
	// Here is a list:
	//
	//  - Item 1.
	//  - Subitem 1.
	//  - Subitem 2.
	//  - Item 2.
	//  - Item 3.
```

Rewriting the text to avoid nested lists usually improves the documentation and is the best solution.
Another potential workaround is to mix list markers, since bullet markers do not introduce list items in a numbered list, nor vice versa.
For example:

```Go
	// Here is a list:
	//
	//  1. Item 1.
	//
	//     - Subitem 1.
	//
	//     - Subitem 2.
	//
	//  2. Item 2.
	//
	//  3. Item 3.
```
















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

[Правила Go](https://go.dev/ref/spec#Assignability) позволяют нам передавать значение типа `MySlice` в параметр типа `[]string`, поэтому вызов `Clone1` вполне допустим.
Но `Clone1` вернет значение типа `[]string`, а не значение типа `MySlice`.
Тип `[]string` не имеет метода `String`, поэтому компилятор сообщает об ошибке.

## Гибкий клон (Flexible Clone)

Чтобы решить эту проблему, нам нужно написать версию `Clone`, которая возвращает тот же тип, что и его аргумент.
Если мы сможем это сделать, то когда мы вызовем `Clone` со значением типа `MySlice`, он вернет результат типа `MySlice`.

Мы знаем, что это должно выглядеть примерно так.

```Go
func Clone2[S ?](s S) S // INVALID 
                        // НЕДОПУСТИМО
```

Эта функция `Clone2` возвращает значение того же типа, что и ее аргумент.

Здесь я записал ограничение как `?`, но это всего лишь заполнитель.
Чтобы это работало, нам нужно написать ограничение, которое позволит нам записать тело функции.
Для `Clone1` мы могли бы просто использовать ограничение `any` для типа элемента.
Для `Clone2` это не сработает: мы хотим, чтобы `s` был типом среза.

Поскольку мы знаем, что нам нужен срез, ограничение `S` должно быть срезом.
Нас не волнует тип элемента среза, поэтому давайте просто назовем его «E», как мы это сделали с `Clone1`.

```Go
func Clone3[S []E](s S) S // INVALID
                          // НЕДОПУСТИМО
```

Это по-прежнему недействительно, поскольку мы не объявили `E`.
Аргумент типа для `E` может быть любого типа, что означает, что он сам должен быть параметром типа.
Поскольку он может быть любого типа, его ограничением является `any`.

```Go
func Clone4[S []E, E any](s S) S
```

Это приближается, и, по крайней мере, оно скомпилируется, но мы еще не совсем там.
Если мы скомпилируем эту версию, мы получим ошибку при вызове `Clone4(ms)`.

```
MySlice does not satisfy []string (possibly missing ~ for []string in []string)
MySlice не соответствует []string (возможно, отсутствует ~ для []string в []string)
```

Компилятор сообщает нам, что мы не можем использовать аргумент типа `MySlice` для параметра типа `S`, поскольку `MySlice` не удовлетворяет ограничению `[]E`.
Это потому, что `[]E` в качестве ограничения допускает только литерал типа среза, например `[]string`.
Он не допускает именованный тип, такой как `MySlice`.

## Базовые ограничения типа(Underlying type constraints)

Как подсказывает сообщение об ошибке, ответом будет добавление `~`.

```Go
func Clone5[S ~[]E, E any](s S) S
```

Повторим, запись параметров и ограничений типа `[S []E, E Any]` означает, что аргумент типа для `S` может быть любым безымянным типом среза, но не может быть именованным типом, определенным как литерал среза.
Написание `[S ~[]E, E Any]` с `~` означает, что аргумент типа для `S` может быть любым типом, базовым типом которого является тип среза.

Для любого именованного типа `type T1 T2` базовым типом `T1` является базовый тип `T2`.
Базовый тип предварительно объявленного типа, такого как `int`, или литерала типа, такого как `[]string`, — это просто сам тип.
Точные сведения см. в [см. спецификацию языка](https://go.dev/ref/spec#Underlying_types).
В нашем примере базовым типом `MySlice` является `[]string`.

Поскольку базовым типом `MySlice` является срез, мы можем передать аргумент типа `MySlice` в `Clone5`.
Как вы могли заметить, подпись `Clone5` такая же, как подпись `slices.Clone`.
Мы наконец-то добрались туда, где хотим быть.

Прежде чем двигаться дальше, давайте обсудим, почему синтаксис Go требует `~`.
Может показаться, что мы всегда хотим разрешить передачу `MySlice`, так почему бы не сделать это значением по умолчанию?
Или, если нам нужно поддерживать точное соответствие, почему бы не перевернуть ситуацию так, чтобы ограничение `[]E` разрешало именованный тип, а ограничение, скажем, `=[]E`, допускало только литералы типа среза?

Чтобы объяснить это, давайте сначала заметим, что список параметров типа, такой как `[T ~MySlice]`, не имеет смысла.

Это потому, что `MySlice` не является базовым типом какого-либо другого типа.
Например, если у нас есть такое определение, как `type MySlice2 MySlice`, базовым типом `MySlice2` будет `[]string`, а не `MySlice`.
Таким образом, либо `[T ~MySlice]` вообще не допускает никаких типов, либо он будет таким же, как `[T MySlice]` и будет соответствовать только `MySlice`.
В любом случае `[T ~MySlice]` бесполезен.
Чтобы избежать этой путаницы, язык запрещает `[T ~MySlice]`, и компилятор выдает ошибку типа


```
invalid use of ~ (underlying type of MySlice is []string)
недопустимое использование ~ (основной тип MySlice — []string)
```

Если бы Go не требовал тильду, чтобы `[S []E]` соответствовал любому типу, базовым типом которого является `[]E`, тогда нам пришлось бы определить значение `[S MySlice]`.

Мы могли бы запретить `[S MySlice]` или сказать, что `[S MySlice]` соответствует только `MySlice`, но любой подход сталкивается с проблемами с предварительно объявленными типами.
Предварительно объявленный тип, например `int`, является собственным базовым типом. Мы хотим дать людям возможность писать ограничения, которые принимают любой аргумент типа, базовым типом которого является `int`.
В современном языке это можно сделать, написав `[T ~int]`.
Если бы нам не требовалась тильда, нам все равно нужен был бы способ сказать «любой тип, базовым типом которого является `int`».
Естественным способом сказать это было бы `[T int]`.
Это означало бы, что `[T MySlice]` и `[T int]` будут вести себя по-разному, хотя они выглядят очень похоже.

Возможно, мы могли бы сказать, что `[S MySlice]` соответствует любому типу, базовым типом которого является базовый тип `MySlice`, но это делает `[S MySlice]` ненужным и запутанным.

Мы считаем, что лучше требовать `~` и четко понимать, когда мы сопоставляем базовый тип, а не сам тип.



## Вывод типа (Type inference)

Теперь, когда мы объяснили сигнатуру `slices.Clone`, давайте посмотрим, как на самом деле упрощается использование `slices.Clone` за счет вывода типа.
Помните, что запись функции `Clone`

```Go
func Clone[S ~[]E, E any](s S) S
```

Вызов `slices.Clone` передаст срез в параметр `S`.
Простой вывод типа позволит компилятору сделать вывод, что аргумент типа для параметра типа `S` является типом среза, передаваемого в `Clone`.
В этом случае вывод типа является достаточно мощным, чтобы увидеть, что аргумент типа для `E` является типом элемента аргумента типа, переданного в `S`.

Это означает, что мы можем написать

```Go
	c := Clone(ms)
```

без необходимости переписывания следующим образом

```Go
	c := Clone[MySlice, string](ms)
```

Если мы ссылаемся на `Clone`, не вызывая его, нам нужно указать аргумент типа для `S`, поскольку компилятору нечего использовать для его вывода.
К счастью, в этом случае вывод типа способен вывести аргумент типа для `E` из аргумента для `S`, и нам не нужно указывать его отдельно.

То есть мы можем написать

```Go
	myClone := Clone[MySlice]
```

без необходимости переписывания следующим образом

```Go
	myClone := Clone[MySlice, string]
```

## Деконструкция параметров типа (Deconstructing type parameters)

Общий метод, который мы использовали здесь, в котором мы определяем один параметр типа `S`, используя другой параметр типа `E`, представляет собой способ деконструкции типов в сигнатурах общих функций.
Деконструируя тип, мы можем назвать и ограничить все аспекты типа.

Например, вот подпись для `maps.Clone`.

```Go
func Clone[M ~map[K]V, K comparable, V any](m M) M
```

Как и в случае с `slices.Clone`, мы используем параметр типа для типа параметра `m`, а затем деконструируем тип, используя два других параметра типа `K` и `V`.

В `maps.Clone` мы ограничиваем `K` сопоставимостью, как это требуется для типа ключа карты.
Мы можем ограничить типы компонентов любым удобным для нас способом.

```Go
func WithStrings[S ~[]E, E interface { String() string }](s S) (S, []string)
```

Это говорит о том, что аргумент `WithStrings` должен быть типом среза, для которого тип элемента имеет метод `String`.

Поскольку все типы Go могут быть созданы из типов компонентов, мы всегда можем использовать параметры типа, чтобы деконструировать эти типы и ограничить их по своему усмотрению.




























# Оригинальные материалы

[^](#Оглавление)


```
git clone https://github.com/golang/website.git

Папка
website/_content/blog
website/_content/doc

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

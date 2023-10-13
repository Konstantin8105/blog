# Блог

# Оглавление

* [fuzz-beta.md](#fuzz-beta.md)
* [Оригинальные материалы](#Оригинальные-материалы)


# fuzz-beta.md

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

```
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


Также имейте в виду, что механизм фаззинга записывает значения, которые расширяют тестовое покрытие, в каталог фазз-кэша внутри `$GOCACHE/fuzz` во время своей работы. В настоящее время нет ограничений на количество файлов или общее количество байтов, которые могут быть записаны в фазз-кеш, поэтому он может занимать большой объем памяти (например, несколько ГБ). Вы можете очистить кэш фазза, запустив gotip clean -fuzzcache.


## Следующий шаг?

Это нововведение будет доступно в Go 1.18.

Если встречаетесь с любыми проблемами или есть идея для функционала, то просим создать [запись на стену](https://github.com/golang/go/issues/new/?&labels=fuzz).

Для обсуждения и получения обратной связи о фазинге, просим ьакже использовать канал [#fuzzing channel](https://gophers.slack.com/archives/CH5KV1AKE) в Gophers Slack.

Счастливого fuzzing!




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

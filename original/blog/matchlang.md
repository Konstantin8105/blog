---
title: Language and Locale Matching in Go
date: 2016-02-09
by:
- Marcel van Lohuizen
tags:
- language
- locale
- tag
- BCP
- 47
- matching
summary: How to internationalize your web site with Go's language and locale matching.
---

## Introduction

Consider an application, such as a web site, with support for multiple languages
in its user interface.
When a user arrives with a list of preferred languages, the application must
decide which language it should use in its presentation to the user.
This requires finding the best match between the languages the application supports
and those the user prefers.
This post explains why this is a difficult decision and how Go can help.

## Language Tags

Language tags, also known as locale identifiers, are machine-readable
identifiers for the language and/or dialect being used.
The most common reference for them is the IETF BCP 47 standard, and that is the
standard the Go libraries follow.
Here are some examples of BCP 47 language tags and the language or dialect they
represent.

{{raw (file "matchlang/tags.html")}}

The general form of the language tag is
a language code (“en”, “cmn”, “zh”, “nl”, “az” above)
followed by an optional subtag for script (“-Arab”),
region (“-US”, “-BE”, “-419”),
variants (“-oxendict” for Oxford English Dictionary spelling),
and extensions (“-u-co-phonebk” for phone-book sorting).
The most common form is assumed if a subtag is omitted, for instance
“az-Latn-AZ” for “az”.

The most common use of language tags is to select from a set of system-supported
languages according to a list of the user's language preferences, for example
deciding that a user who prefers Afrikaans would be best served (assuming
Afrikaans is not available) by the system showing Dutch. Resolving such matches
involves consulting data on mutual language comprehensibility.

The tag resulting from this match is subsequently used to obtain
language-specific resources such as translations, sorting order,
and casing algorithms.
This involves a different kind of matching. For example, as there is no specific
sorting order for Portuguese, a collate package may fall back to the sorting
order for the default, or “root”, language.

## The Messy Nature of Matching Languages

Handling language tags is tricky.
This is partly because the boundaries of human languages are not well defined
and partly because of the legacy of evolving language tag standards.
In this section we will show some of the messy aspects of handling language tags.

_Tags with different language codes can indicate the same language_

For historical and political reasons, many language codes have changed over
time, leaving languages with an older legacy code as well as a new one.
But even two current codes may refer to the same language.
For example, the official language code for Mandarin is “cmn”, but “zh” is by
far the most commonly used designator for this language.
The code “zh” is officially reserved for a so called macro language, identifying
the group of Chinese languages.
Tags for macro languages are often used interchangeably with the most-spoken
language in the group.

_Matching language code alone is not sufficient_

Azerbaijani (“az”), for example, is written in different scripts depending on
the country in which it is spoken: "az-Latn" for Latin (the default script),
"az-Arab" for Arabic, and “az-Cyrl” for Cyrillic.
If you replace "az-Arab" with just "az", the result will be in Latin script and
may not be understandable to a user who only knows the Arabic form.

Also different regions may imply different scripts.
For example: “zh-TW” and “zh-SG” respectively imply the use of Traditional and
Simplified Han. As another example, “sr” (Serbian) defaults to Cyrillic script,
but “sr-RU” (Serbian as written in Russia) implies the Latin script!
A similar thing can be said for Kyrgyz and other languages.

If you ignore subtags, you might as well present Greek to the user.

_The best match might be a language not listed by the user_

The most common written form of Norwegian (“nb”) looks an awful lot like Danish.
If Norwegian is not available, Danish may be a good second choice.
Similarly, a user requesting Swiss German (“gsw”) will likely be happy to be
presented German (“de”), though the converse is far from true.
A user requesting Uygur may be happier to fall back to Chinese than to English.
Other examples abound.
If a user-requested language is not supported, falling back to English is often
not the best thing to do.

_The choice of language decides more than translation_

Suppose a user asks for Danish, with German as a second choice.
If an application chooses German, it must not only use German translations
but also use German (not Danish) collation.
Otherwise, for example, a list of animals might sort “Bär” before “Äffin”.

Selecting a supported language given the user’s preferred languages is like a
handshaking algorithm: first you determine which protocol to communicate in (the
language) and then you stick with this protocol for all communication for the
duration of a session.

_Using a “parent” of a language as fallback is non-trivial_

Suppose your application supports Angolan Portuguese (“pt-AO”).
Packages in [golang.org/x/text](https://golang.org/x/text), like collation and display, may not
have specific support for this dialect.
The correct course of action in such cases is to match the closest parent dialect.
Languages are arranged in a hierarchy, with each specific language having a more
general parent.
For example, the parent of “en-GB-oxendict” is “en-GB”, whose parent is “en”,
whose parent is the undefined language “und”, also known as the root language.
In the case of collation, there is no specific collation order for Portuguese,
so the collate package will select the sorting order of the root language.
The closest parent to Angolan Portuguese supported by the display package is
European Portuguese (“pt-PT”) and not the more obvious “pt”, which implies
Brazilian.

In general, parent relationships are non-trivial.
To give a few more examples, the parent of “es-CL” is “es-419”, the parent of
“zh-TW” is “zh-Hant”, and the parent of “zh-Hant” is “und”.
If you compute the parent by simply removing subtags, you may select a “dialect”
that is incomprehensible to the user.

## Language Matching in Go

The Go package [golang.org/x/text/language](https://golang.org/x/text/language) implements the BCP 47
standard for language tags and adds support for deciding which language to use
based on data published in the Unicode Common Locale Data Repository (CLDR).

Here is a sample program, explained below, matching a user's language
preferences against an application's supported languages:

{{code "matchlang/complete.go"}}

### Creating Language Tags

The simplest way to create a language.Tag from a user-given language code string
is with language.Make.
It extracts meaningful information even from malformed input.
For example, “en-USD” will result in “en” even though USD is not a valid subtag.

Make doesn’t return an error.
It is common practice to use the default language if an error occurs anyway so
this makes it more convenient. Use Parse to handle any error manually.

The HTTP Accept-Language header is often used to pass a user’s desired
languages.
The ParseAcceptLanguage function parses it into a slice of language tags,
ordered by preference.

By default, the language package does not canonicalize tags.
For example, it does not follow the BCP 47 recommendation of eliminating scripts
if it is the common choice in the “overwhelming majority”.
It similarly ignores CLDR recommendations: “cmn” is not replaced by “zh” and
“zh-Hant-HK” is not simplified to “zh-HK”.
Canonicalizing tags may throw away useful information about user intent.
Canonicalization is handled in the Matcher instead.
A full array of canonicalization options are available if the programmer still
desires to do so.

### Matching User-Preferred Languages to Supported Languages

A Matcher matches user-preferred languages to supported languages.
Users are strongly advised to use it if they don’t want to deal with all the
intricacies of matching languages.

The Match method may pass through user settings (from BCP 47 extensions) from
the preferred tags to the selected supported tag.
It is therefore important that the tag returned by Match is used to obtain
language-specific resources.
For example, “de-u-co-phonebk” requests phone-book ordering for German.
The extension is ignored for matching, but is used by the collate package to
select the respective sorting order variant.

A Matcher is initialized with the languages supported by an application, which
are usually the languages for which there are translations.
This set is typically fixed, allowing a matcher to be created at startup.
Matcher is optimized to improve the performance of Match at the expense of
initialization cost.

The language package provides a predefined set of the most commonly used
language tags that can be used for defining the supported set.
Users generally don’t have to worry about the exact tags to pick for supported
languages.
For example, AmericanEnglish (“en-US”) may be used interchangeably with the more
common English (“en”), which defaults to American.
It is all the same for the Matcher. An application may even add both, allowing
for more specific American slang for “en-US”.

### Matching Example

Consider the following Matcher and lists of supported languages:

	var supported = []language.Tag{
		language.AmericanEnglish,    // en-US: first language is fallback
		language.German,             // de
		language.Dutch,              // nl
		language.Portuguese          // pt (defaults to Brazilian)
		language.EuropeanPortuguese, // pt-pT
		language.Romanian            // ro
		language.Serbian,            // sr (defaults to Cyrillic script)
		language.SerbianLatin,       // sr-Latn
		language.SimplifiedChinese,  // zh-Hans
		language.TraditionalChinese, // zh-Hant
	}
	var matcher = language.NewMatcher(supported)

Let's look at the matches against this list of supported languages for various
user preferences.

For a user preference of "he" (Hebrew), the best match is "en-US" (American
English).
There is no good match, so the matcher uses the fallback language (the first in
the supported list).

For a user preference of "hr" (Croatian), the best match is "sr-Latn" (Serbian
with Latin script), because, once they are written in the same script, Serbian
and Croatian are mutually intelligible.

For a user preference of "ru, mo" (Russian, then Moldavian), the best match is
"ro" (Romanian), because Moldavian is now canonically classified as "ro-MD"
(Romanian in Moldova).

For a user preference of "zh-TW" (Mandarin in Taiwan), the best match is
"zh-Hant" (Mandarin written in Traditional Chinese), not "zh-Hans" (Mandarin
written in Simplified Chinese).

For a user preference of "af, ar" (Afrikaans, then Arabic), the best match is
"nl" (Dutch). Neither preference is supported directly, but Dutch is a
significantly closer match to Afrikaans than the fallback language English is to
either.

For a user preference of "pt-AO, id" (Angolan Portuguese, then Indonesian), the
best match is "pt-PT" (European Portuguese), not "pt" (Brazilian Portuguese).

For a user preference of "gsw-u-co-phonebk" (Swiss German with phone-book
collation order), the best match is "de-u-co-phonebk" (German with phone-book
collation order).
German is the best match for Swiss German in the server's language list, and the
option for phone-book collation order has been carried over.

### Confidence Scores

Go uses coarse-grained confidence scoring with rule-based elimination.
A match is classified as Exact, High (not exact, but no known ambiguity), Low
(probably the correct match, but maybe not), or No.
In case of multiple matches, there is a set of tie-breaking rules that are
executed in order.
The first match is returned in the case of multiple equal matches.
These confidence scores may be useful, for example, to reject relatively weak
matches.
They are also used to score, for example, the most likely region or script from
a language tag.

Implementations in other languages often use more fine-grained, variable-scale
scoring.
We found that using coarse-grained scoring in the Go implementation ended up
simpler to implement, more maintainable, and faster, meaning that we could
handle more rules.

### Displaying Supported Languages

The [golang.org/x/text/language/display](https://golang.org/x/text/language/display) package allows naming language
tags in many languages.
It also contains a “Self” namer for displaying a tag in its own language.

For example:

{{code "matchlang/display.go" `/START/` `/END/`}}

prints

	English              (English)
	French               (français)
	Dutch                (Nederlands)
	Flemish              (Vlaams)
	Simplified Chinese   (简体中文)
	Traditional Chinese  (繁體中文)
	Russian              (русский)

In the second column, note the differences in capitalization, reflecting the
rules of the respective language.

## Conclusion

At first glance, language tags look like nicely structured data, but because
they describe human languages, the structure of relationships between language
tags is actually quite complex.
It is often tempting, especially for English-speaking programmers, to write
ad-hoc language matching using nothing other than string manipulation of the
language tags.
As described above, this can produce awful results.

Go's [golang.org/x/text/language](https://golang.org/x/text/language) package solves this complex problem
while still presenting a simple, easy-to-use API. Enjoy.

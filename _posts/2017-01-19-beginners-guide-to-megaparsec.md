---
toc: false
layout: post
description: Explore how to build a simple markdown parser with Haskell and megaparsec
categories: []
title: Beginner's guide to Megaparsec
---
### What is Megaparsec?

[Megaparsec][megaparsec-hackage] is a parsing library written in Haskell. It's a fork of the popular [Parsec][parsec-hackage] and is more up to date.

This post came into being because while there are tutorials for Parsec, there aren't many for Megaparsec. As a Haskell noob, I found myself discouraged to depend on Parsec tutorials too much, given that Megaparsec may have diverged over time.

Without further ado, let's get right to it.

### Environment and Setup

This post is written against `stack`, `ghc` version 8.0.1 and `megaparsec` version 5.0.1.

We can follow the official docs for info on how to install `stack`. Once we have it up and running, installing `megaparsec` is a piece of cake:

```bash
$ stack install megaparsec
```

Note that in a real world project, we'll be adding `megaparsec` to our `.cabal` file, instead of installing it manually like this. However, for the purpose of this tutorial, we'll be working with a single file instead of a full blown project.

Next, create a Haskell source file; let's call it `myparser.hs`.

```bash
$ touch myparser.hs
```

With that done, we have everything we need to begin. All the code examples below should be written in `myparser.hs`.

### Parsing a single letter

Let's start with the simplest of parsers. We'll parse a single letter, and output results, based on whether it passed or failed. This should serve a secondary purpose of verifying that we've set up everything correctly.

```haskell
-- myparser.hs

module MyParser (main) where

import Text.Megaparsec
import Text.Megaparsec.String
import System.Environment

main = do
    input <- fmap head getArgs
    parseTest singleLetterP input

singleLetterP :: Parser Char
singleLetterP = char 'h'
```

Let's dissect this bit of code. `Text.Megaparsec` module is the umbrella for all the basic functions from this package. It provides methods such as `parseTest`, `parse` etc. along with a host of parsers and combinators.

`Text.Megaparsec.String` module provides the `Parser` datatype synonym. In short, this means that we're dealing with string inputs. There are corresponding modules for `ByteString` and `Text` types for when they're needed.

In the first line of `main` function, we're getting the first command line argument. We're using a partial function `head`, which is not advised in production code. However, here, it serves our purpose, since the code will now give error if we fail to provide an argument.

The second line of the `main` function is calling `parseTest` function. This function takes a parser and input string as arguments. It runs the parser on the input, and prints the results/error to stdout.

Finally, `singleLetterP` is the actual parser we wrote. Take a note of its datatype. `Char` in `Parser Char` tells us what this parser will output.

In the function definition, `char` function is a utility parser provided by `Text.Megaparsec.Char` module, which is already included in the `Text.Megaparsec` import. This parser matches a single letter provided to it as argument.

Let's run this code and see what happens:

```bash
$ stack runghc myparser.hs haskell
'h'

$ stack runghc myparser.hs javascript
1:1:
unexpected 'j'
expecting 'h'
```

So we get our first working parser, which gives us the result or helpful error messages to boot.

### How parsing works

When we run a parser, it begins parsing from the very beginning of input. The parser we write should match the input continuously, or it'll fail. Also, the parser will match/consume only as much as we'll tell it to.

For instance, in our example above, when we supplied 'haskell' as the input string, it matched `h` from the very beginning, because that's all we coded. It stopped matching after that.

If we had provided `jhaskell` or something else, it'll still fail. That's because the parser `char 'h'` will not arbitrarily match 'h' from anywhere in the string. It'll consume a single letter from beginning and pass or fail depending on whether it's 'h'.

That said, we'll see how to tie together multiple parsers which let us skip letters or run multiple parsers over same inputs to see what fits.

### Parsing this OR that

What if we have a scenario where we want to be able to run more than one parser on same input, in case the previous one fails. We have a handy infix function `<|>` for exactly that.

Observe:

```haskell
-- myparser.hs

-- ... imports and module declaration ...

main = do
    input <- fmap head getArgs
    parseTest singleLetterP input

singleLetterP :: Parser Char
singleLetterP = char 'h' <|> char 'j'
```

Now, this parser will match anything starting with 'h' or 'j'. Here's our code in action:

```bash
$ stack runghc myparser.hs haskell
'h'

$ stack runghc myparser.hs javascript
'j'

$ stack runghc myparser.hs ruby
1:1:
unexpected 'a'
expecting 'h' or 'j'
```

Even though we've only combined two single letter parsers, this infix method `<|>` works on arbitrarily complex parsers.

### Parsing this AND that

As we've seen, `<|>` basically acts as boolean OR function for parsers. But we also want to be able to run more than one parser one after another sequentially, each parser consuming a part of input string, and forwarding the rest onwards.

Fortunately, the datatype `Parser a` belongs to Monad typeclass. Which means we get the `>>=` (bind function) to compose multiple parsers.

Let's modify our code so that our parser will match two letters, one after another.

```haskell
-- myparser.hs

-- ... imports and module declaration ...

main = do
    input <- fmap head getArgs
    parseTest doubleLetterP input

doubleLetterP :: Parser Char
doubleLetterP = char 'h' >>= (\ _ -> char 'a')
```

Now, our `doubleLetterP` parser will consume a single letter 'h', followed by a single letter 'a'.

```
$ stack runghc myparser.hs haskell
'a'

$ stack runghc myparser.hs hoogle
unexpected 'o'
expecting 'a'

$ stack runghc myparser.hs assembly
1:1:
unexpected 'a'
expecting 'h'
```

The syntax we've used to combine two parsers this way leaves something to be desired. Fortunately, by the virtue of being a member of Monad typeclass, we get the `do` syntax for free.

Following parser does the exact same thing as the previous example, only with an arguably more readable syntax:

```haskell
-- myparser.hs

-- ... imports and main function ...

doubleLetterP :: Parser Char
doubleLetterP = do
    char 'h'
    char 'a'
```

So far, we've only been using a single parser - `char`. There are many other useful parsers which we have in our arsenal out of the box. For the full list, check out the API docs for [Text.Megaparsec.Char][megaparsec-char-doc].

### Combinators

Combinators are functions that we use to combine one or more parsers in different ways. We've already seen one of them - the `<|>` function. But that's neither the only combinator available to us, nor is that that only way we can combine our parsers.

Let's write a parser to match a word. We define word as a continuous string of alphanumeric characters. Any spaces, tabs and/or special characters are not included.

```haskell
-- myparser.hs

-- ... imports ...

main = do
    input <- fmap head getArgs
    parseTest wordP input

wordP :: Parser String
wordP = some alphaNumChar
```

Firstly, note the change in datatype of our `wordP` function. Since we now expect our parser to match a string instead of single character, the datatype will reflect that. Feel free to check out what error we get if we change that back to `Parser Char`.

Secondly, we've introduced a combinator here, namely `some`. This is a function that takes a parser as argument. In our example, `alphaNumChar` is that argument. `some` will try to run the parser continuously one or more times as long as it keeps getting successful.

Here's our new parser in action

```bash
$ stack runghc myparser.hs haskell
"haskell"

$ stack runghc myparser.hs "haskell hoogle"
"haskell"

$ stack runghc myparser.hs ";haskell"
1:1:
unexpected ';'
expecting alphanumeric character

$ stack runghc myparser.hs ""
1:1:
unexpected end of input
expecting alphanumeric character
```

One of the major ups of using megaparsec is the helpful error messages we get without doing anything extra.

Note: There's also a `many` combinator which matches 0 or more occurrences of the supplied parser.

### Transforming parsed strings

More often than not, if we're parsing a string, it's because we want to extract some data out from it and do something with it. For example, we might be parsing an entries from account statement, and we want to convert the parsed balances to a number type.

For this example, we'll try to parse the bold notation of markdown and convert it to html tag. In markdown, if we have to make text bold, we enclose it in `**`. For instance, `**This is bold**` will show up as **This is bold** after conversion.

Behind the scenes, the markdown converter parses the text inside `**` delimiters, and puts it in `<strong>` tags. Let's try to write a parser which does this.

```haskell
-- myparser.hs

-- ... imports ...

main = do
    input <- fmap head getArgs
    parseTest boldP input

boldP :: Parser String
boldP = do
    _ <- count 2 (char '*')
    txt <- some (alphaNumChar <|> char ' ')
    _ <- count 2 (char '*')
    return $ concat [ "<strong>", txt, "</strong>" ]

```

Let's look into what's happening in our `boldP` parser. `count` is another combinator we have which allows us to consume a set number of matches for a parser.

In this instance, first, we consume two `*` characters. Then, we consume one or more alphanumeric characters, as well as spaces.
Lastly, we consume another two `*` characters.

The more interesting thing happening here is that we are able to extract the matched string into a variable. We can do that with any parser we write, because all `Parser`s are monads.

In the last line of our `boldP`, we simply wrap the extracted text from between the `**` with `<strong>` tags.

Let's see this parser in action:

```bash
$ stack runghc myparser.hs '** haskell **'
"<strong> haskell </strong>"

$ stack runghc myparser.hs "**haskell has functions**"
"<strong>haskell has functions</strong>"

$ stack runghc myparser.hs haskell
1:1:
unexpected 'h'
expecting '*'
```

Note that our parser only matches alphanumeric characters and spaces between the delimiters. It'll be more robust when it matches pretty much anything between the delimiters. Feel free to implement that.

We can similarly parse strings into our custom datatypes, convert between them and more. We just need to remember that the datatype of the final output of parser is reflected in the datatype of the parser itself.

### Conclusion

There are many more tools to discover in megaparsec itself, for various general and specific tasks. With this rather lengthy tutorial, I hope I have been able to get some idea of how we can get started with writing parsers. Feel free to share thoughts below or on the [reddit thread][comments].

[megaparsec-hackage]: https://hackage.haskell.org/package/megaparsec
[parsec-hackage]: https://hackage.haskell.org/package/parsec
[megaparsec-char-doc]: https://hackage.haskell.org/package/megaparsec-5.1.2/docs/Text-Megaparsec-Char.html
[comments]: https://www.reddit.com/r/haskell/comments/5ow2v5/beginners_guide_to_megaparsec/

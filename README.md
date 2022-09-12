# JinxScript

JinxScript is a JavaScript code generator that encodes any piece of text (which obviously includes JavaScript code) by using the many weird features of JavaScript, into heavily obfuscated code using only 32 symbol and punctuation characters.

It is the spiritual successor to [jjencode](https://utf-8.jp/public/jjencode.html), however using a selection of modern JavaScript features, such as template literals, array methods, anonymous functions, and so much more, all while keeping the output length to a minimum.

Decoding it is super easy, just paste the output and run it in a terminal.

The project is still in the making though it definitely needs some improvement.

## Features

- Generated code can be run even on a browser!
- Admire that build time (8s build time on entire _Lord of the Rings_ trilogy)
- Only 32 characters, nothing more, nothing less!

## Disclaimer

The code and command-line program in this repo is **NOT** intended and **SHALL NOT** be used for malicious purposes. A lot of web attacks use obfuscation to bypass content filters, even more so since the generated code is completely devoid of alphanumerics.

I strongly urge you **NOT** to inject ANY malicious JavaScript code, nor misuse the code generated by this program for anything malicious or illegal. This program shall be used **ONLY** for **EXPERIMENTAL** purposes.

**I am not held responsible for any damage caused by the program in this repo and your wilful indifference.**

## Implementation Requirements

- Output code should **only** contain the following 32 symbol and punctuation characters: `` !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~ ``, nothing more, nothing less
- Output code should be valid JavaScript that when run in a console or terminal evaluate to the original string.
- Output should be as short as possible.
- Script should also work when in strict mode; the only accepted tokens containing alphanumerics are the keywords `var` or `let`.
- No time for randomness: output should be the same when run and re-run again.
- Only two global variables should be used.

Usage:

```sh
js-obfuscate ./path/to/file.txt
```

- `--global`: Specify the name of the global variable (default `$`)
- `--let`: Use `let` rather than `var`
- `--help`: Display usage and available options
- `--out=path/to/dir/file.js`: Specify the output file (default is `./out`)
- `--prove`: Run the interpreter on the generated code with `node`
- `--log`: Include a `console.log` command

## Future enhancements

- Important: **Please, add the compiled Babel output!**
- Add unicase alphanumeric strings to the parser, as those can be gotten with a simple `.toString(36)` conversion on a number. The maximum allowed length is `10`.
- Add all possible substrings to existing strings defined on the global object.
- Limit the tokens to runs of 256 characters, though this run length can be specified.
- Improve the current lexer as it fails to the `default` `Base31/UTF16` encoder when there are multiple of the same string in a row adjacent to other sequences of strings not separated by a word boundary.
- Use an encoding/assignment scheme that derives new n-grams by combining or modifying strings. Default is 4 n-grams.

## How it works

Like JJEncode, JinxScript is a substitution encoding scheme that goes through three phases:

- Initialization, where characters and values are assigned to variables;
- Substitution, where the variables are used to construct strings;
- Execution, where the constructed code is evaluated and executed.

You may want to read [this article](https://blog.korelogic.com/blog/2015/01/12/javascript_deobfuscation) explaining how the original jjencode works, along with the main key differences between JinxScript and jjencode.

The entire file is generated with the same [program I made](https://github.com/nxltm/JinxScript/blob/main/obfuscator.js); do take note I used experimental JavaScript syntax, which is run using a special BabelJS configuration (see `package.json`)

### Initialization

We start with our global variable, `$`

JavaScript variables are pretty flexible (and weird) int he characters that can be used, so the above name of `$` is a valid name. Unlike other languages like Python or Ruby, this character is not reserved and can therefore can be used in any part of the variable name. This allows other variables names that we'll see in this JS, such as `$_`, `$$$`, and `_$_`.

The file starts out simple with `$=~[]` which consists of two parts: declaring a variable `$` and assigning the value of `~[]`, (bitwise not on an empty array), which evaluates to `-1`.

Now, looking on the source file, we see a cipher string.

We have 32 characters: `` ;.!:_-,?/'*+#%&^"|~$=<>`@\()[]{} `` and out of these, subtracting the bracket pairs, we now have 26 characters, `` ;.!:_-,?/'*+#%&^"|~$=<>`@\ `` enough to encode all the letters of the English alphabet.

So, using this cipher, we assign a pair of symbols to each letter of the English alphabet.

```js
const LETTERS = `abcdefghijklmnopqrstuvwxyz`
const CIPHER = `;.!:_-,?/'*+#%&^"|~$=<>\`@\\`
```

The first symbol, `$` or `_` determines if the symbol is uppercase or lowercase. The second is assigned an arbitrary symbol.

- `_` and `$`, which can form valid variable names in JavaScript, are reserved for the two most common letters `e` and `t`.
- `x` and `z` are rarely used and therefore get the escape sequences `\\` and `` \` `` which are slightly longer.

For digits, we use a different encoding scheme, we would convert to binary, pad the result to 3 digits, and then assign `_` to `0` and `$` to `1`.

```js
const encodeLetter = char =>
  (V.isUpperCase(char) ? "$" : "_") +
  (char.toLowerCase() |> LETTERS.indexOf(%) |> CIPHER[%])

const encodeDigit = number =>
  +number
  |> %.toString(2).padStart(3, 0)
  |> %.replace(/(?<_0>0)|(?<_1>1)/g, (_0, _1) => (_0 == 1 ? "$" : "_"))
```

The second line is a bit longer, but broken up we see:

```js
$ = {
  ___: `${++$}`,
  _$: `${!""}`[$],
  "_-": `${![]}`[$],
  "$/": `${!"" / ![]}`[$],
  "$%": `${+{}}`[$],
  __$: `${++$}`,
  "_|": `${!""}`[$],
  "_;": `${![]}`[$],
  "_%": `${[][[]]}`[$],
  "_&": `${{}}`[$],
  _$_: `${++$}`,
  "_=": `${!""}`[$],
  "_+": `${![]}`[$],
  "_:": `${[][[]]}`[$],
  "_.": `${{}}`[$],
  _$$: `${++$}`,
  __: `${!""}`[$],
  "_~": `${![]}`[$],
  "_'": `${{}}`[$],
  $__: `${++$}`,
  $_$: `${++$}`,
  "_/": `${[][[]]}`[$],
  "_!": `${{}}`[$],
  $$_: `${++$}`,
  $$$: `${++$}`,
  "_@": `${!"" / ![]}`[$],
  "-": `${{}}`[$],
  $___: `${++$}`,
  "$&": `${{}}`[$],
  $__$: `${++$}`,
}
```

In this line, `$` is being reassigned to a JavaScript object, as denoted by the curly braces. Properties of the object are defined within the braces in the form `key: value`, and individual properties are separated by commas.

The first property is `___` (3 underscores), or its full name, `$.___`. The value of this property is `++$`, which takes the value of `$` (currently `-1`), increments it (to `0`), and then assigns it to the property. So, in this statement, `$`is incremented by one to 0 and then assigned to`$.___`. Note that since the object is still being built, `$` is still a number and not an object yet.

The second property is `"_$"`. `$` is assigned to the letter `t`, while the `_` tells it is lowercase. `!`(logical-not)-ing an array, which evaluates to `true`, produces `false`; doing the same with an empty string, which evaluates to `false`, produces `true`.

Because of JavaScript's type coercion, `Infinity` is made by simply dividing `true` by `false`, as it innately converts them to floating-point numbers: `true` = `1` and `false` = `0`. There is no integer type in JavaScript.

`undefined` can be made by accessing an unknown property on an array _or_ object, `NaN` is made by attempting to convert anything into a number where it shouldn't.

The infamous string `[object Object]` can be made by converting an object to a string. Because this is JavaScript.

```js
const CONSTANTS = {
  true: '!""', // ''==false
  false: "![]", // []==true
  undefined: "[][[]]", // [][[]]
  Infinity: '!""/![]', // true/false==1/0
  NaN: "+{}", // It makes sense
  "[object Object]": "{}",
}
```

We don't even need to call toString, we just need to wrap the entire expression in a template string literal.

The rest of the object properties are constructed in a similar fashion: incrementing the `$` variable, constructing a string, and grabbing a character out of the string by specifying its index.

After decoding all of the object, the values look as such:

```js
$ = {
  ___: "0",
  _$: "t",
  "_-": "f",
  "$/": "I",
  "$%": "N",
  __$: "1",
  "_|": "r",
  "_;": "a",
  "_%": "n",
  "_&": "o",
  _$_: "2",
  "_=": "u",
  "_+": "l",
  "_:": "d",
  "_.": "b",
  _$$: "3",
  __: "e",
  "_~": "s",
  "_'": "j",
  $__: "4",
  $_$: "5",
  "_/": "i",
  "_!": "c",
  $$_: "6",
  $$$: "7",
  "_@": "y",
  "-": " ",
  $___: "8",
  "$&": "O",
  $__$: "9",
}
```

What do we have? A bunch of letters. The purpose of this whole statement was so that we have enough letters to begin forming our first words, such as `concat`, `call`, `join`, `return`, `slice`, and `constructor`. We also have a space character, which we assigned to the property of `-`.

```js
const CHARSET_1 = {}

for (const [constant, expression] of _.entries(CONSTANTS))
  for (const char of constant)
    if (/[\w\s]/.test(char) && !(char in CHARSET_1))
      CHARSET_1[char] = [expression, constant.indexOf(char)]

const RES_CHARSET_1 =
  _.range(0, 10)
  |> %.map(digit => [
    encodeDigit(digit) + ":`${++" + GLOBAL_VAR + "}`",
    _.entries(CHARSET_1)
    |> %.filter(([, [, val]]) => val == digit)
    |> %.map(([char, [lit]]) => {
      const key = quote(encodeLetter(char))
      return key + ":`${" + lit + "}`[" + GLOBAL_VAR + "]"
    }),
  ])
  |> %.flat().join`,`
  |> %.replace(/,$/, "").replace(/,+/g, ",")
  |> [GLOBAL_VAR + "={" + % + "}"][0]
  |> %.replace("_undefined", SPACE)

RESULT += RES_CHARSET_1
```

The next few lines construct more strings used for substitution, mainly for string manipulation. We would assign single-character symbols to these, so that they can be referenced without having to re-spell everything again.

```js
const IDENT_SET1 = {
  concat: "+",
  call: "!",
  join: "%",
  slice: "/",
  return: "_",
  constructor: "$",
}
```

We would use the constructors to retrieve some more letters, such as `v`, `g`, `m` and some capitals such as `A`, `S,` `N`, `R`, `E`, and `F`. The letter `v` is extracted from the word `native`, which is gotten from the stringified version of any one of these constructors, `Array` being the shortest of them all.

```js
Array.constructor.toString() == "function Array() { [native code] }"
```

These expressions or literals are the shortest possible expressions containing no alphanumeric characters.

```js
const LITERALS = {
  Array: "[]",
  Object: "{}",
  String: '""', // or ``, ''
  Number: "(~[])", // or +'', +{}
  Boolean: "(![])", // or !'', !{}
  RegExp: "/./", // or /!/, etc.
  Function: "(()=>{})",
}
```

This is an assignment to a new property in the `$` object, `$.$`. The value of the property is constructed by concatenating values together, as denoted by the plus operator. Each value is grabbing a character using an index, so this is a string being constructed. We can evaluate each of the values to get the entire string.

```js
const CHARSET_2 = { ...CHARSET_1 }

for (const [key, expression] of _.entries(LITERALS)) {
  const constructor = eval(key).toString()
  for (const char of constructor)
    if (/[\w\s]/.test(char) && !(char in CHARSET_2))
      CHARSET_2[char] = [expression, constructor.indexOf(char)]
}

for (const value of _.entries(CHARSET_2)) {
  const [char, [expression, index]] = value

  const expansion =
    "`${" +
    expression +
    "[" +
    GLOBAL_VAR +
    ".$]}`[" +
    [...`${index}`].map(digit => {
      return GLOBAL_VAR + "." + encodeDigit(digit)
    }).join`+` +
    "]"

  if (!(char in CHARSET_1)) {
    CHARSET_2[char] = [expression, index, expansion]
  }
}
```

This, corresponds to the lines in JavaScript:

<!-- WIP -->
# VoidScript

VoidScript is a JavaScript code generator that encodes any piece of text (which obviously includes JavaScript code) by using the many weird features of JavaScript, into heavily obfuscated code using only 32 symbol and punctuation characters.

It is the spiritual successor to Yosuke Hasegawa's [jjencode](https://utf-8.jp/public/jjencode.html), however using a selection of modern JavaScript features, such as template literals, array methods, anonymous functions, and so much more, all while keeping the output length to a minimum.

Decoding it is super easy, just paste the output and run it in a terminal.

The project is still in the making though it definitely needs some improvement.

## Features

- Generated code can be run even on a browser!
- Admire that build time (2s build/2s execution time on entire _Lord of the Rings_ trilogy)
- Only 32 characters, nothing more, nothing less!

## Disclaimer

The code and command-line program in this repo is **NOT** intended and **SHALL NOT** be used for malicious purposes. A lot of web attacks use obfuscation to bypass content filters, even more so since the generated code is completely devoid of alphanumerics.

This program is also hardware-intensive, as it uses complex, brain-busting methods _all the time_ to generate the code that evaluates back to the original string.

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

Like JJEncode, VoidScript is a substitution encoding scheme that goes through three phases:

- Initialization, where characters and values are assigned to variables;
- Substitution, where the variables are used to construct strings;
- Execution, where the constructed code is evaluated and executed.

You may want to read [this article](https://blog.korelogic.com/blog/2015/01/12/javascript_deobfuscation) explaining how the original jjencode works, along with the main key differences between VoidScript and jjencode.

The entire file is generated with the same [program I made](https://github.com/nxltm/VoidScript/blob/main/obfuscator.js); do take note I used experimental JavaScript syntax, which is run using a special BabelJS configuration (see `package.json`).

### How the obfuscator works

The obfuscator/encoder works by heavily abusing JavaScript's weird quirks, including its infamous type coercion feature. JavaScript is a weakly typed programming language, and it allows the evaluation of any expression as any type.

It's weird enough that you could write an entire valid JavaScript program or virtually any piece of text with as little as six characters. This project uses a substantial character pool of 32, which are the symbol and punctuation characters in ASCII, nothing more, nothing less.

This program is a superset of jjencode in that regard and many concepts behind jjencode have also been adopted in VoidScript. There is a single global variable (default `$`) which stores strings and functions, with a second storing the encoded input string as an obfuscated JavaScript expression.

Both are essentially a substitution encoding that goes through two phases: initialization, where characters and substrings are assigned to values, and substitution, where the variables are used to construct back the input string. There might be a third phase, execution, which is done by passing the code (evaluating to a string) to the Function constructor to be executed as JavaScript.

I assign the global variable `$` to `~[]`, made by getting the bitwise NOT of an empty array. We then increment this variable and begin assigning properties into an object by accessing single characters of simple boolean, numeric or primitive values as strings: `true`, `false`, `undefined`, `Infinity`, `NaN`, and `[object Object]`, making to cipher our properties so we can access them later. In the same statement, we then reassign that object to the global variable.

I assign the letters with two characters, the first which defines the case and the second the actual letter. The numbers have keys defined as binary.

I then use these letters to begin assembling strings such as `constructor`, and we can turn them into strings by converting its constructor into a string, to give us a string like `function Array() { [native code] }`, yielding us a few more letters. This includes the new Function literal, `()=>{}`.

In between every step, I then start forming the names of methods and functions with these single-letter strings, and along the way retrieving global functions such as `eval`, `escape` and `parseInt` with the Function constructor. I assign single symbol keys to these strings so I could use them to access properties and call methods easier.

Now we have almost the entire lowercase alphabet save for a few and about half of the uppercase letters. The other five lowercase letters (`hkqwz`) are made by converting a small number into a base 36 string which always yields lowercase. Two more uppercase letters `C` and `D` are created by escaping a URL with a single character, and uppercase `U` is gotten from the `[object Undefined]` string.

Using the strings I made, I define a pair of functions that turn arbitrary UTF-16 sequences into a sequence of symbol characters. (The decoder function is ciphered). The code points are converted into base 31 and its digits ciphered using the 31 symbol characters, leaving the comma to separate each UTF-16 code unit.

Words that repeat themselves throughout the program are captured, encoded and assigned new keys to the same global object, ranked based on decreasing frequency and given a predefined key based on a custom generator function so that they can be referenced in the string later on. Exceptions to this include the single character strings, stringified primitives and literals, and the property and method strings which have been defined already.

Sequences of ASCII symbols are included literally in the output. A regular expression splits and tokenizes the input string into these categories, and then joins them together with a `+` which is the string concatenation operator.

While the output does have some letters and symbols, I left them there mostly for debugging.

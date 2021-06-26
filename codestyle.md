# The James Livesey Official Coding Style
I'm very particular about the way I arrange the syntax in my code. I believe
that having a strict, distinct coding style is something that every programmer
should have. Otherwise, code will gradually get messier and more unreadable as
the codebase gets mutated in ways that do not conform to _any_ coding style,
_ever_.

The definitions in this coding style document have been formulated over many
years of my coding, and it's what I've been comfortable with for around 10 years
now!

If you're working on one of my projects, please please _please_ use this
document as a reference to how you should write your code ─ or otherwise I may
reject your pull request (especially when your code looks bonkers)! Since not
all of the definitions defined herein will work in _all_ programming languages,
you may need to adapt the styles shown below in a similar way.

## Indentation
Use 4 spaces per indent. No tabs. Don't ask why ─ it's what I'm used to. I don't
like looking at code that has 2 spaces per indent as code seems to mush all into
one line. Similarly, those who write with 8 spaces per indent would force me to
purchase an ultrawide monitor so I can see the sheer length of some of your
lines of code.

Please don't _not_ add idents. Python users can't _not_ do this, though!

**Example:**
```javascript
function isValidUnsignedInt(value, bits) {
    if (typeof(value) != "number") {
        return false;
    }

    if (value != Math.floor(value)) {
        return false;
    }

    if (value >= 0 && value < Math.pow(2, bits)) {
        return true;
    }

    return false;
}
```

## Commenting
Comments should either appear above or directly to the right of code. Inline
comments should not be indented to the next tab stop.

Text in comments should be written in sentence case, and rarely should have a
full stop (`.`) at the end of the sentence (unless a comment contains multiple
sentences).

**Example:**
```javascript
/*
    My lovely piece of code
    Notice how this block comment is indented
*/

calculateMean([1, 2, 4, 8]); // This calculates the mean of four integers
```

## Line spacing
Blank lines of code should be placed in-between adjacent statements and should
isolate statements from other code. They should also be used to differentiate
different subfunctionalities of a section of code. Otherwise, it'll feel like
the coding equivalent of reading one **absolute unit** of a paragraph.

Blank lines should isolate variable definitions from commands too.

**Example:**
```javascript
function getWeighting(boolean) {
    var weighting = 0;

    if (boolean === true) {
        weighting = 1;
    } else if (boolean === false) {
        weighting = -1;
    }

    return weighting;
}

// Scores in the form of [[number, boolean], ...]
function weightScores(scores) {
    var weightedScores = [];

    for (var i = 0; i < scores.length; i++) {
        if (scores[i][0] == 0 && scores[i][1] == null) {
            continue;
        }

        weightedScores.push(scores[i][0] * getWeighting(scores[i][1]));
    }

    return [...new Set(weightedScores)]; // Remove duplicate scores
}

console.log(weightScores([
    [5, true],
    [1, null],
    [0, false]
]));
```

## Token spacing
There should be one space (and one space only) between tokens (such as `+` and
`-` etc.). The only exception to this is when 'lining up' variable definitions
is absolutely necessary, or when expanding brackets onto multiple lines. If you
don't, it'llfeellikeyou'rereadingthecodingequivalentofthis!

There are a few exceptions to this rule:
* Colons (`:`) that are used outside of ternary operators should not have a
  space before them (when they're being used in ternary operations, both spaces
  should surround the `:`)
* Commas (`,`) and semicolons (`;`) should not have a space before them
* Unary operators (such as exclamation marks `!`and tildes `~`) should not have
  a space after them.

**Example:**
```javascript
function calculateMean(values) {
    if (values.length == 0) {
        throw TypeError("Value array is empty");
    }

    return values.reduce((a, b) => a + b) / values.length;
}
```

## Control flow brackets
Brackets that dictate what commands belong in a statement (brace brackets `{`
`}` are used in most programming languages) must open on the same line as the
statement itself. People who open brackets on a new line just don't make sense.

**Example:**
```javascript
function getSign(sign) {
    if (typeof(sign) != "number") {
        return NaN;
    } else if (sign >= 0) {
        return 1;
    } else {
        return -1;
    }
}
```

## Command brackets
Brackets that dictate what arguments belong in a command (brackets `(` `)` are
used in most programming languages) must open immediately after the identifer.
There should be no space between the command's identifier and the opening
bracket.

If it is beneficial to expand the brackets onto multiple lines for readability
reasons, then the contents inside those brackets should be indented. This should
present itself similarly to the way control flow brackets are arranged.

**Example:**
```javascript
console.log(calculateMean(weightScores([
    [4, false],
    [8, null],
    [7, true],
    [16, false]
])));
```

## Identifier naming
I am an avid user of camelCase naming. This is because camelCase is used
extensively throughout the JavaScript standard library. Other kinds of cases
such as flatcase, PascalCase, Snake_Case, SHOUTY_CASE and kebab-case should not
be used (in fact the latter case would not work in most programming languages).

There are two exceptions to this: constant identifiers and class identifiers.
Constant identifiers should use SHOUTY_CASE and class identifiers should use
PascalCase.

**Example:**
```javascript
const FREEFALL_ACCELERATION = 9.81;

var score = 0;
var amountOfWorldExplored = 0.2;

class UnidentifiedFlyingObject {
    constructor(origin, size) {
        this.origin = origin;
        this.mass = Math.pow(size, 3);
    }

    get weight() {
        return this.mass * FREEFALL_ACCELERATION;
    }

    set weight(value) {
        this.mass = value / FREEFALL_ACCELERATION;
    }
}

function createWorld() {
    var world = [];

    for (var i = 0; i < 100; i++) {
        world.append(new UnidentifiedFlyingObject("Mars", Math.random() * 5));
    }

    return world;
}
```

## String literals
Strings should be wrapped in quotation marks (`"`) instead of apostrophes (`'`).
Qutotation marks must be escaped if they are used in a string. I prefer it this
way because then I don't have to escape every single apostrophe in contractions.

Backticks can also wrap around strings in some cases (such as where a string is
multiline, or where templating in a string is particularly useful). Newline
escapes (`\n` in most programming languages) are preferred over using multiline
strings, though.

**Example:**
```javascript
console.log("The quick brown fox jumped over the lazy dog.");

sendMessage(`Hi, this is ${username} here!`);
```

## Method chaining
When chaining methods (such as with jQuery), such methods can be either inline
or in expanded form. When they are in expanded form, the 'shape' of the chain
should be similarly arranged as to how arguments are arranged in brackets.

To those who think this is weird: I agree ─ it's certainly not _too_ standard.
It's just that I like the indentation so it's much easier to read large trees of
method chains.

**Example:**
```javascript
$("h1").text("Hello!").addClass("greeting");

$("body").append(
    $("<h1>")
        .addClass("articleTitle")
        .text(article.title)
    ,
    $("<div>").append([
        $("<small>").text(article.date.toLocaleDateString("en-GB")),
        $("<p>")
            .text(article.excerpt)
            .attr("title", "Subscribe to read more!")
    ])
);
```

## Enumerators
Enumerators must be in camelCase. In JavaScript, enumerators must be emulated
using a constant object. I like enumerators ─ they're much nicer than using
strings to identify multiple cases!

```javascript
const cardinalDirections = {
    NORTH: 0,
    EAST: 1,
    SOUTH: 2,
    WEST: 3
};
```

## Miscellaneous
* Classes should use setters and getters wherever possible.
* If a subroutine is capable of returning a value, return it insetad of setting
  a global variable.
* Identifiers should be written in English (United Kingdom) wherever possible.
* If it looks bad, it _is_ bad!

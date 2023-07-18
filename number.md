Please create PrairieLearn questions according to the following rules.

There are three files to generate:
1. info.json
2. question.html
3. server.py

# info.json
The `info.json` file for each question defines properties of the question. These are all the possible properties:

| Property                 | Type    | Description                                                                                                                                                            |
| ------------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `uuid`                   | string  | [Unique identifier](https://prairielearn.readthedocs.io/en/latest/uuid/). (Required; no default)                                                                       |
| `type`                   | enum    | Type of the test. Must be `"v3"` for new-style questions. (Required; no default)                                                                                       |
| `title`                  | string  | The title of the question (e.g., `"Addition of vectors in Cartesian coordinates"`). (Required; no default)                                                             |
| `topic`                  | string  | The category of question (e.g., `"Vectors"`, `"Energy"`). Like the chapter in a textbook. (Required; no default)                                                       |
| `tags`                   | array   | Optional extra tags associated with the question (e.g., `["secret", "concept"]`). (Optional; default: no tags)                                                         |
| `gradingMethod`          | enum    | The grading method used for auto-grading this question. Valid values: `Internal`, `External`, or `Manual` (for manual-only questions). (Optional; default: `Internal`) |
| `singleVariant`          | boolean | Whether the question is not randomized and only generates a single variant. (Optional; default: `false`)                                                               |
| `showCorrectAnswer`      | boolean | Whether the question should display the answer panel. (Optional; default: `true`)                                                                                      |
| `partialCredit`          | boolean | Whether the question will give partial points for fractional scores. (Optional; default: `true`)                                                                       |
| `externalGradingOptions` | object  | Options for externally graded questions. See the [external grading docs](https://prairielearn.readthedocs.io/en/latest/externalGrading/). (Optional; default: none)    |
| `dependencies`           | object  | External JavaScript or CSS dependencies to load. See below. (Optional; default: `{}`)                                                                                  |

# question.html
The `question.html` is a template used to render the question to the student.
The `question.html` is regular HTML, with four special features:

1. Any text in double-curly-braces (like `{{params.m}}`) is substituted with variable values. If you use triple-braces (like `{{{params.html}}}`) then raw HTML is substituted (don't use this unless you know you need it). This is using [Mustache](https://mustache.github.io/mustache.5.html) templating.
2. Special HTML elements (like `<pl-number-input>`) enable input and formatted output. See the [list of PrairieLearn elements](https://prairielearn.readthedocs.io/en/latest/elements/). Note that that **all submission elements must have unique `answers-name` attributes.** This is is necessary for questions to be graded properly.
3. A special `<markdown>` tag allows you to write Markdown inline in questions. 
4. LaTeX equations are available within HTML by using `$x^2$` for inline equations, and `$$x^2$$` or `\[x^2\]` for display equations.

# server.py
The `server.py` file for each question creates randomized question variants by generating random parameters and the corresponding correct answer. The `server.py` functions are:

|Function|Return object|modifiable `data` keys|unmodifiable `data` keys|Description|
|---|---|---|---|---|
|`generate()`||`correct_answers`, `params`|`options`, `variant_seed`|Generate the parameter and true answers for a new random question variant. Set `data["params"][name]` and `data["correct_answers"][name]` for any variables as needed. Modify the `data` dictionary in-place.|
|`prepare()`||`answers_names`, `correct_answers`, `params`|`options`, `variant_seed`|Final question preparation after element code has run. Can modify data as necessary. Modify the `data` dictionary in-place.|
|`render()`|`html` (string)||`correct_answers`, `editable`, `feedback`, `format_errors`, `options`, `panel`, `params`, `partial_scores`, `raw_submitted_answers`, `score`, `submitted_answers`, `variant_seed`, `num_valid_submissions`|Render the HTML for one panel and return it as a string.|
|`parse()`||`format_errors`, `submitted_answers`|`correct_answers`, `options`, `params`, `raw_submitted_answers`, `variant_seed`|Parse the `data["submitted_answers"][var]` data entered by the student, modifying this variable. Modify the `data` dictionary in-place.|
|`grade()`||`correct_answers`, `feedback`, `format_errors`, `params`, `partial_scores`, `score`, `submitted_answers`|`options`, `raw_submitted_answers`, `variant_seed`|Grade `data["submitted_answers"][var]` to determine a score. Store the score and any feedback in `data["partial_scores"][var]["score"]` and `data["partial_scores"][var]["feedback"]`. Modify the `data` dictionary in-place.|
|`file()`|`object` (string, bytes-like, file-like)|

You are to generate integer and number input questions. Here are the specifications for pl-integer-input and pl-number-input:
`pl-integer-input` element: Fill in the blank field that requires an **integer** input.
## Customizations

|Attribute|Type|Default|Description|
|---|---|---|---|
|`answers-name`|string|—|Variable name to store data in. Note that this attribute has to be unique within a question, i.e., no value for this attribute should be repeated within a question.|
|`weight`|integer|1|Weight to use when computing a weighted average score over elements.|
|`correct-answer`|float|special|Correct answer for grading. Defaults to `data["correct_answers"][answers-name]`. If `base` is provided, then this answer must be given in the provided base.|
|`allow-blank`|boolean|false|Whether or not an empty input box is allowed. By default, empty input boxes will not be graded (invalid format).|
|`blank-value`|integer|0 (zero)|Value to be used as an answer if element is left blank. Only applied if `allow-blank` is `true`.|
|`label`|text|—|A prefix to display before the input box (e.g., `label="$x =$"`).|
|`suffix`|text|—|A suffix to display after the input box (e.g., `suffix="items"`).|
|`base`|integer|10|The base used to parse and represent the answer, or the special value 0 (see below).|
|`display`|"block" or "inline"|"inline"|How to display the input field.|
|`size`|integer|35|Size of the input box.|
|`show-help-text`|boolean|true|Show the question mark at the end of the input displaying required input parameters.|
|`placeholder`|string|-|Custom placeholder text. If not set, defaults to "integer" if `base` is 10, otherwise "integer in base `base`".|
|`show-score`|boolean|true|Whether to show the score badge next to this element.|

## Specifying a non-trivial base
By default, the values are interpreted in base 10. The `base` argument may also be used, with a value between 2 and 36, to indicate a different base to interpret the student input, as well as to print the final result.

The `base` argument can also accept a special value of 0. In this case, the values will by default be interpreted in base 10, however the student has the option of using different prefixes to indicate a value in a different format:

- The prefixes `0x` and `0X` can be used for base-16 values (e.g., `0x1a`);
- The prefixes `0b` and `0B` can be used for base-2 values (e.g., `0b1101`);
- The prefixes `0o` and `0O` can be used for base-8 values (e.g., `0o777`).

#### Integer range

pl-integer-input can accept integers of unbounded size, however the correct answer will only be stored as the Python `int` if it is between -9007199254740991 and +9007199254740991 (between -(2^53 - 1) and +(2^53 - 1)). Otherwise, the correct answer will be stored as a string. This distinction is important in `server.py` scripts for `parse()` and `grade()`, as well as downloaded assessment results.

Note that answers can include underscores which are ignored (i.e., `1_000` will be parsed as `1000`).

## `pl-number-input` element

Fill in the blank field that allows for **numeric** value input within specific tolerances.
## Customizations

|Attribute|Type|Default|Description|
|---|---|---|---|
|`answers-name`|string|—|Variable name to store data in. Note that this attribute has to be unique within a question, i.e., no value for this attribute should be repeated within a question.|
|`weight`|integer|1|Weight to use when computing a weighted average score over elements.|
|`correct-answer`|float|special|Correct answer for grading. Defaults to `data["correct_answers"][answers-name]`.|
|`label`|text|—|A prefix to display before the input box (e.g., `label="$F =$"`).|
|`suffix`|text|—|A suffix to display after the input box (e.g., `suffix="$\rm m/s^2$"`).|
|`display`|"block" or "inline"|"inline"|How to display the input field.|
|`comparison`|"relabs", "sigfig", or "decdig"|"relabs"|How to grade. "relabs" uses relative ("rtol") and absolute ("atol") tolerances. "sigfig" and "decdig" use "digits" significant or decimal digits.|
|`rtol`|number|1e-2|Relative tolerance for `comparison="relabs"`.|
|`atol`|number|1e-8|Absolute tolerance for `comparison="relabs"`.|
|`digits`|integer|2|number of digits that must be correct for `comparison="sigfig"` or `comparison="decdig"`.|
|`allow-complex`|boolean|false|Whether or not to allow complex numbers as answers. If the correct answer `ans` is a complex object, you should use `import prairielearn as pl` and `data["correct_answers"][answers-name] = pl.to_json(ans)`.|
|`allow-blank`|boolean|false|Whether or not an empty input box is allowed. By default, empty input boxes will not be graded (invalid format).|
|`blank-value`|string|0 (zero)|Value to be used as an answer if element is left blank. Only applied if `allow-blank` is `true`. Must follow the same format as an expected user input (e.g., fractions if allowed, complex numbers if allowed, etc.).|
|`show-help-text`|boolean|true|Show the question mark at the end of the input displaying required input parameters.|
|`show-placeholder`|boolean|true|Show the placeholder text that shows the default comparison.|
|`size`|integer|35|Size of the input box.|
|`show-correct-answer`|boolean|true|Whether to show the correct answer in the submitted answers panel.|
|`allow-fractions`|boolean|true|Whether to allow answers expressed as a rational number of the format `a/b`.|
|`custom-format`|string|special|Custom format to use when formatting the submitted or correct answer. By default, submitted answers are shown with the format `.12g`, while the correct answer uses `.12g` if the comparison method is `relabs`, or a custom format based on the number of digits if the comparison method is `sigfig` or `decdig`. A full description of the format can be found [on the Python documentation page](https://docs.python.org/3/library/string.html#format-specification-mini-language).|


# Examples
## Example 1 - Number Input
User input:
info.json:
title = Compute the correct number
topic = Example
tags = math, number

question.html:
Question Text = Given two numbers a = {{ a }} and b = {{ b }}, what is c = {{ equation }}?
Label = $c =$

server.py:
a = integer from 2 to 20
b = integer from 6 to 11
eq_options:
1. sqrt(a^2 + b^2),
2. a^2/b
3. 2(b-a)^2
4. (a^2 + b^2)/a
5. (a^2 + b^2)/b
equation = randomly pick from eq_options
ans = calculated equation value

Your Output:
info.json:
```json
{
    "uuid": "f24e0597--42aa-abef-6a96b6afdca8",
    "title": "Compute the correct number",
    "topic": "Example",
    "tags": ["math", "number"],
    "type": "v3"
}
```

question.html:
```html
<pl-question-panel>
<p>
Given two numbers $a = {{params.a}}$ and $b = {{params.b}}$, what is $c = {{params.equation}}$?
</p>
</pl-question-panel>

<p>
<pl-number-input answers-name="ans" label="$c =$"></pl-number-input>
</p>
```

server.py:
```python
import math
import random

def generate(data):
    # Sample an integer number between 2 and 20
    a = random.randint(2, 20)
    data["params"]["a"] = a

    # Sample an integer number between 6 and 11
    b = random.randint(6, 11)
    data["params"]["b"] = b

    # Options for equations
    eq_options = [
        {"equation": "\\sqrt{(a^2 + b^2)}", "solution": math.sqrt(a**2 + b**2)},
        {"equation": "a^2/b", "solution": a**2 / b},
        {"equation": "2(b-a)^2", "solution": 2 * (b - a) ** 2},
        {"equation": "(a^2 + b^2)/a", "solution": (a**2 + b**2) / a},
        {"equation": "(a^2 + b^2)/b", "solution": (a**2 + b**2) / b},
    ]
    choice = random.choice(eq_options)

    # Put the result into data['params']
    data["correct_answers"]["ans"] = choice["solution"]
    data["params"]["equation"] = choice["equation"]
```
## Example 2 - Integer Input
User Input:
info.json:
title = Compute the angle
topic = Example
tags = math, integer

question.html:
label =$\theta =$
suffix = $^o$
Question Text = Convert the angle theta = {{ angle_radians }} (in radians) to degrees:

server.py:
angle_radians = [pi/2, pi/4, pi/3, pi/6] (randomly pick one)
angle_degrees = [90, 45, 60, 30]

ans = corresponding degree in angle_degrees

Your Output:
info.json:
```json
{
    "uuid": "ac12d03a-c234-4123-82a3-e53889da6ec2",
    "title": "Compute the angle",
    "topic": "Example",
    "tags": ["math", "integer"],
    "type": "v3"
}
```

question.html:
```html
<pl-question-panel>
<p>Convert the angle $\theta = {{params.angle_radians}}$ (in radians) to degrees:</p>
</pl-question-panel>

<pl-integer-input answers-name="ans" label="$\theta =$" suffix="$^o$"></pl-integer-input>
```

server.py:
```python
import random


def generate(data):
    # Create a list of possible angles in radians
    angles_radians = ["\\pi/2", "\\pi/4", "\\pi/3", "\\pi/6"]
    angles_degrees = [90, 45, 60, 30]

    # Select one of the entries of the list
    option = random.randint(0, len(angles_radians) - 1)

    # Put the angle in radians into data['params']
    data["params"]["angle_radians"] = angles_radians[option]

    # Put the correct answer into data['correct_answers']
    data["correct_answers"]["ans"] = angles_degrees[option]
```

Do the examples make sense? Are you ready to create PrairieLearn questions?

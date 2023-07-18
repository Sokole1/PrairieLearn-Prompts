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

You are to generate multiple-choice type questions. Here are the specifications for pl-multiple-choice:
A `pl-multiple-choice` element selects **one** correct answer and zero or more incorrect answers and displays them in a random order as radio buttons. Duplicate answer choices (string equivalents) are not permitted in the `pl-multiple-choice` element, and an exception will be raised upon question generation if two (or more) choices are identical.

## Customizations

|Attribute|Type|Default|Description|
|---|---|---|---|
|`answers-name`|string|—|Variable name to store data in. Note that this attribute has to be unique within a question, i.e., no value for this attribute should be repeated within a question.|
|`weight`|integer|1|Weight to use when computing a weighted average score over elements.|
|`inline`|boolean|false|List answer choices on a single line instead of as separate paragraphs.|
|`number-answers`|integer|special|The total number of answer choices to display. Defaults to displaying one correct answer and all incorrect answers.|
|`fixed-order`|boolean|false|Disable the randomization of answer order.|
|`hide-letter-keys`|boolean|false|Hide the letter keys in the answer list, i.e., (a), (b), (c), etc.|
|`all-of-the-above`|string|`false`|Add "All of the above" choice. See below for details.|
|`none-of-the-above`|string|`false`|Add "None of the above" choice. See below for details.|
|`all-of-the-above-feedback`|string|—|Helper text to be displayed to the student next to the `all-of-the-above` option after question is graded if this option has been selected by the student.|
|`none-of-the-above-feedback`|string|—|Helper text to be displayed to the student next to the `none-of-the-above` option after question is graded if this option has been selected by the student.|
|`external-json`|string|special|Optional path to a JSON file to load external answer choices from. Answer choices are stored as lists under "correct" and "incorrect" key names.|
|`external-json-correct-key`|string|special|Optionally override default json "correct" attribute name when using `external-json` file.|
|`external-json-incorrect-key`|string|special|Optionally override default json "incorrect" attribute name when using `external-json` file.|
|`allow-blank`|boolean|false|Whether or not an empty submission is allowed. If `allow-blank` is set to `true`, a submission that does not select any option will be marked as incorrect instead of invalid.|

The attributes `none-of-the-above` and `all-of-the-above` can be set to one of these values:

- `false`: the corresponding choice will not be shown in the list of choices. This is the default.
- `random`: the corresponding choice will always be shown, and will be randomly correct, with probability proportional to the total number of correct choices. In other words, if there are `N` possible correct choices in total, this choice will be correct with probability `1/N`.
- `correct`: the corresponding choice will always be shown and will always be the correct answer.
- `incorrect`: the corresponding choice will always be shown and will always be an incorrect answer (i.e., a distractor).
- `true`: same as `random`, accepted for backwards compatibility.

Note that "All of the above" and "None of the above", if set, are bounded by the `number-answers` value above. Also, these two values are always shown as the last choices, regardless of the setting for `fixed-order`. If both choices are shown, then "All of the above" will be listed before "None of the above".

Inside the `pl-multiple-choice` element, each choice must be specified with a `pl-answer` that has attributes:

|Attribute|Type|Default|Description|
|---|---|---|---|
|`correct`|boolean|false|Is this a correct answer to the question?|
|`feedback`|string|—|Helper text (HTML) to be displayed to the student next to the option after question is graded if this option has been selected by the student.|
|`score`|float|-|Score given to answer choice if selected by student. Defaults to 1.0 for correct answers and 0.0 for incorrect answers.|

# Examples
## Example 1 - Multiple Choice Simple
User input:
info.json:
title = Simple Multiple Choice
topic = Example
tags = mc, simple

question.html:
Question Text = Which is a capital city in Canada?
Number Answers = 4
Correct Answers = Ottawa, Victoria, Edmonton, Winnipeg, Toronto, Quebec City, Fredericton, Halifax, Charlottetown, St. John's, Yellowknife, Iqaluit, Whitehorse
Incorrect Answers = Montreal, Vancouver, Calgary, Saskatoon, Regina, Hamilton, London, Moncton, Saint John, Sydney, Corner Brook, Thunder Bay

Your Output:
info.json:
```json
{
    "uuid": "f24e0597--42aa-abef-6a96b6afdca8",
    "title": "Simple Multiple Choice",
    "topic": "Example",
    "tags": ["mc", "simple"],
    "type": "v3"
}
```

question.html:
```html
<pl-question-panel>
Which is a capital city in Canada?
</pl-question-panel>

<pl-multiple-choice answers-name='answer' number-answers='4'>
  <pl-answer correct='true'>Ottawa</pl-answer>
  <pl-answer correct='true'>Victoria</pl-answer>
  <pl-answer correct='true'>Edmonton</pl-answer>
  <pl-answer correct='true'>Winnipeg</pl-answer>
  <pl-answer correct='true'>Toronto</pl-answer>
  <pl-answer correct='true'>Quebec City</pl-answer>
  <pl-answer correct='true'>Fredericton</pl-answer>
  <pl-answer correct='true'>Halifax</pl-answer>
  <pl-answer correct='true'>Charlottetown</pl-answer>
  <pl-answer correct='true'>St. John's</pl-answer>
  <pl-answer correct='true'>Yellowknife</pl-answer>
  <pl-answer correct='true'>Iqaluit</pl-answer>
  <pl-answer correct='true'>Whitehorse</pl-answer>

  <pl-answer>Montreal</pl-answer>
  <pl-answer>Vancouver</pl-answer>
  <pl-answer>Calgary</pl-answer>
  <pl-answer>Saskatoon</pl-answer>
  <pl-answer>Gold</pl-answer>
  <pl-answer>Regina</pl-answer>
  <pl-answer>Hamilton</pl-answer>
  <pl-answer>London</pl-answer>
  <pl-answer>Moncton</pl-answer>
  <pl-answer>Saint John</pl-answer>
  <pl-answer>Sydney</pl-answer>
  <pl-answer>Corner Brook</pl-answer>
  <pl-answer>Thunder Bay</pl-answer>
</pl-multiple-choice>
```

## Example 2 - Multiple Choice With Formula
User Input:
info.json:
title = Multiple Choice Formula
topic = Example
tags = mc, formula, physics

question.html:
none-of-the-above = random
Question Text = Under ideal conditions, a car accelerates uniformly from an initial velocity of {{ initial_velocity }} m/s at a rate of {{ acceleration }} m/s² for a duration of {{ time }} seconds. What is the final velocity of the car?

server.py:
All numbers are 1 decimal unless specified otherwise
initial_velocity = number from 5.0 to 35.0
acceleration = number from 1.0 to 5.0
time = number from 3.0 to 10.0

correct = initial_velocity + acceleration * time, units = m/s
w1 = acceleration * time 
w2 = initial_velocity - acceleration * time
w3 = random number from 0.0 to 100.0 

Your Output:
info.json:
```json
{
    "uuid": "ac12d03a-c234-4123-82a3-e53889da6ec2",
    "title": "Multiple Choice Formula",
    "topic": "Example",
    "tags": ["mc", "formula", "physics"],
    "type": "v3"
}
```

question.html:
```html
<pl-question-panel>
Under ideal conditions, a car accelerates uniformly from an initial velocity of ${{ params.initial_velocity }}\rm\ m/s$ at a rate of ${{ params.acceleration }}\rm\ m/s²$ for a duration of ${{ params.time }}\rm\ s$. What is the final velocity of the car?
</pl-question-panel>

<pl-multiple-choice answers-name="answer" none-of-the-above="random">
  <pl-answer correct="true">${{params.correct}}\rm\  m/s$</pl-answer>
  <pl-answer correct="false">${{params.w1}}\rm\ m/s$</pl-answer>
  <pl-answer correct="false">${{params.w2}}\rm\ m/s$</pl-answer>
  <pl-answer correct="false">${{params.w3}}\rm\ m/s$</pl-answer>
</pl-multiple-choice>
```

server.py:
```python
import random

def generate(data):
    # Generate question variables
    initial_velocity = round(random.uniform(5.0, 35.0), 1)
    acceleration = round(random.uniform(1.0, 5.0), 1)
    time = round(random.uniform(3.0, 10.0), 1)

    # Calculate the answer and distractor answers
    correct = round(initial_velocity + acceleration * time, 1)
    w1 = round(acceleration * time, 1)
    w2 = round(initial_velocity - acceleration * time, 1)
    w3 = round(random.uniform(0.0, 100.0), 1)
    
    # Put variables in data
    data["params"]["initial_velocity"] = initial_velocity
    data["params"]["acceleration"] = acceleration
    data["params"]["time"] = time
    
    data["params"]["correct"] = correct
    data["params"]["w1"] = w1
    data["params"]["w2"] = w2
    data["params"]["w3"] = w3
```

Do the examples make sense? Are you ready to create Multiple Choice Questions?

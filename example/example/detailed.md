<!-- markdownlint-disable MD033 -->

# A Demo Form

## Description

This is a detailed example demonstrating the usage of scriptedforms.

This file format is based upon markdown. There are however a few
extra custom html elements. The custom html elements come in two
types, either section elements `<section-something>` or
variable elements `<variable-something>`.

## Custom elements overview

### Sections overview

Whenever markdown code blocks are written within a section that code
is sent to the Python kernel when certain conditions are fulfilled.

There are four kinds of sections

* `<section-start>`,
* `<section-live>`,
* `<section-button>`,
* and `<section-output>`.

Code which is written inside of these defined sections is run
as python code according to specific rules.

### Variable overview

Veriable elements are attached to a specific python variable which update on
user input. All variable elements require at least one item placed between the
open and close braces. The first item between the open and close braces is
the Python variable definition. It doesn't strictly have to be a Python variable
it merely has to be valid Python code to exist on the left hand side of an assignment
equal sign.

There are six kinds of variable inputs:

* `<variable-number>`,
* `<variable-slider>`,
* `<variable-table>`,
* `<variable-tick>`,
* `<variable-toggle>`,
* `<variable-string>`,
* and `<variable-dropdown>`.

`<variable-number>` and `<variable-slider>` represent floats or integers.
`<variable-string>` represents a Python string. `<variable-tick>` and `<variable-toggle>`
are both booleans. The `<variable-table>` is a pandas dataframe with all of the values 
in the dataframe being floats or integers. `<variable-dropdown>` is a string with provided options.

## Usage of the scriptedforms elements

### Start sections

Whenever a jupyterlab services session is started
code within the start sections is run first.

If you reopen or update the form template without restarting the kernel
this code will not re-run however a button will appear that will allow you to
manually re-run the code if need be.

An example `<section-start>` is given following:

<section-start>

```python
data = np.ones(3) * np.nan
data[0] = 5

table = pd.DataFrame(
  columns=['Meas1', 'Meas2', 'Meas3', 'Avg'],
  index=['6MV', '10MV'],
  data=[[1,np.nan,np.nan,np.nan],[4,5,np.nan,np.nan]])

hello = False
world = False
bye = False

machine = None

submit_count = 0
output_count = 0
```

</section-start>

As can be seen from this code there are already a few namespaces included by
default within the Python session. Some of these are for convenience, some are
required for the proper running of the form. The code that is run at boot of
a new form kernel can be found within the
[source code](https://github.com/SimonBiggs/scriptedforms/blob/master/scriptedforms/src/services/session-start-code.ts).

### Live sections and demo of each of the variable types

The `<section-live>` element is designed to contain both code and variable inputs. Whenever
the user changes any variable within the live section all code that is also
contained within that live section is subsequently run.

Each of the usable variables are demoed below making use of `<section-live>`.

#### Number and slider variables

Both the number and the slider require at least one input, the python variable
name. They both have three remaining optional inputs, minimum, maximum, and
step size in that order. The inputs need to be separated by semi colons (`;`).
White space is ignored. The use of commas (`,`) to separate values is deprecated
and will be removed in version 0.6.0.

Should optional inputs not be given they are assigned the default values. In
both the slider and number variables step size defaults to 1. In the number
variable minimum and maximum defaults to not being set. On the slider the
minimum defaults to 0, the maximum to 100.

Below is a `live` section containing both number and slider that produces a
plot:

<section-live>
<variable-number>data[0]</variable-number>
<variable-number>data[1]; -100; 100</variable-number>
<variable-number>data[2]; 0; 10; 0.1</variable-number>

<variable-slider>data[0]</variable-slider>
<variable-slider>
  data[1];
  -100;
  100
</variable-slider>
<variable-slider>
  data[2];
  0;
  10;
  0.1
</variable-slider>
`plt.plot(data, 'o');`
</section-live>


#### Table variables

Table variables display a full pandas dataframe. The live code can update one
part of the table as other parts are being edited.

<section-live>
<variable-table>table</variable-table>

```python
table.iloc[:,3] = np.nanmean(table.iloc[:,0:3], axis=1)
```

</section-live>

#### The tick and toggle variables

Tick and toggle variables are simply different representations of a True/False
boolean variable within python. They are provided for use cases such as check
lists and pass fail tests. These variables can interact with each other in
interesting ways via the live Python code.

<section-live>
<variable-tick>hello</variable-tick>

<variable-tick>world</variable-tick>

```python
if bye:
    hello = False
    world = False

if hello and world:
    print('Hello World!')
```

<variable-toggle>bye</variable-toggle>
</section-live>

#### String variables

String variables fill the entire width of the container they are in. They also
expand when new lines are provied. An example use case is an optional notes
field.

<section-live>
<variable-string>notes</variable-string>
`print(notes)`
</section-live>

#### Dropdown variables

Dropdown allow predifined options to be available in a dropdown list. When
defining a dropdown variable the first input is the python variable name. All
remaining inputs are give the options for the dropdown. All options must be
separated by semi colons (`;`). The use of commas (`,`) to separate values is deprecated
and will be removed in version 0.6.0. Surrounding whitespace is ignored.

<section-live>
<variable-dropdown>machine;
  1234;
  2345;
  George
</variable-dropdown>

```python
print(machine)
```

</section-live>

### Button sections

Button groups are designed for long running or standalone tasks that
should not run whenever a user changes a variable.

They are defined as following:

<variable-string>notes</variable-string>

<section-button>
`print(notes)`
</section-button>

They will not run until their respective button is pressed.

#### Button customistion

Button sections are customisable, their content can be changed to words by
changing the name property.

<section-button name="Submit">

```python
submit_count += 1
print('Submitted {} times!'.format(submit_count))
```

</section-button>

Buttons can also be disabled using the conditional property. An example is the
following button which is only enabled once the submit count becomes at least
10.

<section-button name="Super Submit" conditional="submit_count >= 10">

```python
print('SUPER SUBMIT!!!')
```

</section-button>

### Output sections

Code placed within output groups will run after any variable on the form
changes in value.

<section-output>

```python
output_count += 1
print(output_count)

print(_scriptedforms_variable_handler.variables_json)
```

</section-output>

[Not yet implemented] Any variable placed within an output group will format as a non-editable card.
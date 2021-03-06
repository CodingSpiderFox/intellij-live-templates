<!--
Copyright 2021 Tamás Balog

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Misc

## clipboard()

Just like what you'd expect, quoting the official documentation it *"Returns the contents of the system clipboard."*

**Related macro:** [ClipboardMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/ClipboardMacro.java)

## complete()

Invokes code completion at the position of the variable.

For the sake of an example let's say I would like a live template to create new `HashMap` instances with an initial capacity that I get passed in from somewhere.
A template can be:

```java
java.util.Map<String, String> $name$ = new java.util.HashMap<>($capacity$);
```

Then you can configure `$capacity$` with the `complete()` macro which will trigger the code completion:

![complete](images/complete.gif)

**Official documentation:** [Code completion/Basic completion](https://www.jetbrains.com/help/idea/auto-completing-code.html#basic_completion)

**Related macro:** [CompleteMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/CompleteMacro.java)

## completeSmart()

Invokes smart type completion at the position of the variable.

Let's say you would like to create an instance of Google Guava's `StopWatch` including querying the elapsed time, which code snippet will surround your code to be developed.

For that you might assemble for the following template:

```java
com.google.common.base.Stopwatch stopwatch = Stopwatch.createStarted();
        
//do something

long elapsedTime = stopwatch.elapsed($timeunit$);
```

The only thing needing configuration is the `$timeunit$` variable whose Expression value will be `completeSmart()`. Then at the place of this variable
the smart type completion will appear.

![complete_smart](images/complete_smart.gif)

**Official documentation:** [Code completion#Smart completion](https://www.jetbrains.com/help/idea/auto-completing-code.html#smart_completion)

**Related macro:** [CompleteSmartMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/CompleteSmartMacro.java)

## date(sDate)

Returns the current system date in the specified format.

Let's say, for some reason, you like to keep track when you did certain changes in your code and you'd like to insert the current date and/or time
as a comment or into your javadoc. For that a template might be:

```java
//Latest modification: $currentDate$
```

The only thing you need to configure is the Expression part of the `$currentDate$` variable which can be done multiple ways.

#### No date format

In case there is no parameter specific for the `date()` macro, as the official documentation also states
     
> the current date is returned in the default system format.

You can be on Mac, Unix or Windows, it will return the date in its default system format. All this logic is handled in `com.intellij.util.text.DateFormatUtil`.

![date_without_params](images/date_without_params.gif)

#### Single custom date format

If you'd like to use a different date format you can pass in a date-time format String value that is according to the `java.text.SimpleDateFormat` specification.

Using the format `"Y-MM-d, E, H:m"` the result will change as following: 

![date_with_single_param](images/date_with_single_param.gif)

**NOTE:** this example is made on a Hungarian language system, so the current day abbreviation is according to that: V for Vasárnap, Sunday in Hungarian. 

#### Multiple macro parameters

The macro takes only one parameter and ignores all extra parameters, but according to the macro implementation if you pass in more than one parameters,
it acts as if there were no parameters specified at all, falling back to the default system format.

#### Invalid date format

If you specify a custom date format string but it is not valid according to the `SimpleDateFormat` specification, e.g. `"Y-MM-d, E, H:mmmXSAm"`,
it returns an error message defined in the macro implementation, `com.intellij.codeInsight.template.macro.CurrentDateMacro`.

![date_incorrect_format](images/date_incorrect_format.gif)

**Related macro:** [CurrentDateMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/CurrentDateMacro.java)

## groovyScript("groovy code", arg1)

Executes the Groovy script passed as a string.

### Return a single value

In its simplest form you can return a single value from the script. Although it might not make sense in certain cases,
it is good for demonstration purposes.

```groovy
groovyScript("'groovy code'")
```

It returns the String `groovy code`.

### Path to Groovy file

The first parameter of the macro may not just be actual code to execute, but also a path to the file that contains the script.

The underlying logic uses `java.io.File` to handle the input as a file, so whatever input it accepts should work as an input for the `groovyScript()` macro.

Absolute path: `groovyScript("E:\\livetemplates\\src\\main\\java\\picimako\\sandbox\\AGroovyClass.groovy")`

### Escaping

In case there is a more complex Groovy script provided to the macro it might contain additional quotation that has to be escaped
due to the quotation marks of the macro input parameter. For example in case of wanting to use the literal `"` character:

```groovy
groovyScript("return '\"groovy code\"'")
```

It returns `"groovy code"`.

### Multiline scripts

It is inevitable to write multiline scripts, but it somehow has to be passed into the macro. The easiest way to do that is to end each statement
with a semicolon and separate it from the next line with a whitespace.

```groovy
groovyScript("def thisIs = 'This is'; def sparta = 'Spartaaaaa...'; return thisIs + ' ' + sparta;")
```

It returns `This is Spartaaaaa...`.

### Parameters

It is also possible to pass in optional input parameters to the script. Those parameters might be literal values or other macros
that evaluate to some (not necessarily primitive) values.

The input parameters are indexed and can be reached as follows: `_1`, `_2`, `_3` and so on for the first, seconds, third and so on
parameters accordingly. Parameter indexing starts from 1 instead of 0 (since the 0th parameter is the script itself).

```groovy
groovyScript("return _1 + _2", "Conca", "tenated")
```

If, for instance, the input parameter evaluates to a list, you can reach each of its elements by querying them by their indexes:

```groovy
groovyScript("return _1[2] + ' ' + _1[1] + ' ' + _1[0]", methodParameters())
```

Apparently this macro doesn't handle literal values other than Strings. Although it is possible to pass in literal int, boolean and
other values, it handles them as an empty value.

This is not the case when you pass in a macro that evaluates to an int, boolean, etc.

```groovy
groovyScript("return 'Current line number: ' + _1;", lineNumber())
```

### _editor variable

There is a designated variable called `_editor` that holds an instance of [`com.intellij.openapi.editor.Editor`](https://github.com/JetBrains/intellij-community/blob/master/platform/editor-ui-api/src/com/intellij/openapi/editor/Editor.java).

Via this variable you can access a lot of extra information about the editor, the project, etc.

You can see this variable in action in the built-in template, **output / soutp**.

### Caveats

- Live templates in general, and the `groovyScript()` macro [cannot be used in File and Code templates](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206155379-How-to-show-24-hour-format-time).
- Other Live template macros [cannot be called/referenced inside the macro script](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206286559-Any-Live-Templates-groovyScript-examples), it is only their values that can be passed in as parameters.
So the following macro won't work, it will return an error message:

    ```groovy
    groovyScript("return 'Current line number: ' + lineNumber();")
    ```

### Other resources 

**JetBrains support discussions:**
- [groovyScript() as parameter of a groovyScript()](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206491509-Is-there-a-way-to-use-an-environment-variable-in-a-live-template-script-path-)
- [Javadoc formatting and throws clause](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360003143400-live-templates-method-format-question)
- [Classpath settings and availability](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206151709-What-is-or-where-is-the-internal-classpath-supplied-to-the-groovyScript-runtime-in-Live-Templates-)

**Stackoverflow:**
- [Current file path in Live Template](https://stackoverflow.com/questions/38488564/current-file-path-in-live-template)
- [Relative file path parameter value](https://stackoverflow.com/questions/32998389/intellij-idea-live-templates-variable-run-groovy-script-relation-file-path)

**Built-in templates using the groovyScript macro:**
- output / soutp ([Related Stackoverflow post](https://stackoverflow.com/questions/1440525/idea-live-template-to-log-method-args/21168027#21168027))

**Related macro:** [GroovyScriptMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/GroovyScriptMacro.java)

## lineNumber()

Returns the line number at which the associated template parameter is positioned after invoking the live template.

Considering that the live template is invoked in line 11, the results would be the following:

| Template | Result  |
|---|---|
| <pre>lineNumber$</pre> | 11 |
| <pre>$start$<br><br><br>$lineNumber$</pre> | 14 |

**Related macro:** [LineNumberMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/LineNumberMacro.java)

## showParameterInfo()

NOTE: This is not included in the official documentation at the moment.

While you are defining the parameters passed to a method or an annotation, a little tooltip on top of the parameter list is displayed with all the signatures that match the current state of passed parameters.
In some cases all signatures or options are shown but the most relevant to the typed text is highlighted in that list.

The tooltip cannot be interacted with, so for example your cannot select a signature/option from that list to pre-populate the required number of parameter separator commas.

### Annotation parameters

In case of one annotation attribute it doesn't matter if you use a single template variable for the whole value (including the attribute name),
or define the attribute name and its value separately, the tooltip is displayed regardless. Below are two examples for a single annotation attribute. 

#### One attribute
```java
@org.openqa.selenium.support.FindBy($strategy$)
org.openqa.selenium.WebElement $element$;
```

![showParameterInfo_annotation_one_param](images/showParameterInfo_annotation_one_param.gif)

```java
@org.openqa.selenium.support.FindBy(css = $locator$)
org.openqa.selenium.WebElement $element$;
```

![showParameterInfo_annotation_one_param_attribute](images/showParameterInfo_annotation_one_param_attribute.gif)

#### Multiple attributes
In case of multiple annotation attributes (and template variables), the parameter info tooltip is displayed only when you start defining the value of the template variable
this macro is added to. From that point on, it is displayed for any further attributes.

```java
@io.cucumber.junit.CucumberOptions(tags = "$tags$", features = "$features$")
```

![showParameterInfo_annotation_multiple_params](images/showParameterInfo_annotation_multiple_params.gif)

### Method call parameters

In case of method call parameters this macro behaves the same way as it behaves in case of annotations.

```java
List<String> items = java.util.List.copyOf($params$);
```

![showParameterInfo_method_one_param](images/showParameterInfo_method_one_param.gif)

In case of multiple parameters:
> ... (and template variables), the parameter info tooltip is displayed only when you start defining the value of the template variable
  this macro is added to. From that point on, it is displayed for any further parameters.

```java
List<String> items = java.util.List.of($params$, $param2$);
```

![showParameterInfo_method_multiple_params](images/showParameterInfo_method_multiple_params.gif)

**Related Bug ticket:** [showParameterInfo() macro tooltip causes the red rectangle to not appear](https://youtrack.jetbrains.com/issue/IDEA-237488)

**Related macro:** [ShowParameterInfoMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/ShowParameterInfoMacro.java)

## suggestFirstVariableName(sFirstVariableName)

Quoting from a related [JetBrains support ticket](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360007850440-Live-template-How-does-suggestFirstVariableName-macro-work-) (kudos to Olga Klisho):

> `suggestFirstVariableName` macro is the same as [`variableOfType`](types.md#variableoftypetype), both suggest variables of the given type available in the context,
> but the latter one also suggests "standard expressions" if they have a compatible type: "true", "false", "this" and "Outer.this".

> The arguments can look like `"double"` or `"java.util.Collection"` or other macros, e.g. `methodReturnType()`
> which returns the return type of the method where the caret is positioned.

### Primitive types

Primitive types can be specified by their names as the macro's parameter, like: `suggestFirstVariableName("short")`.

Let's say you have the index of some special days stored in `short` type fields, and with a live template you'd like to
create `short` variables with name suggestions.

```java
short $aSpecialDay$ = $day$;
```

Then you can configure the `$day$` variable's Expression as `suggestFirstVariableName("short")`.

In case of specifying a primitive type as the parameter of this macro, the suggestions list
will show variables of their respective non-primitive types as well (including `java.lang.Void` for the keyword `void`).

![suggestFirstVariableName_primitives_incl_non_primitive](images/suggestFirstVariableName_primitives_incl_non_primitive.gif)

### Non-primitive types

Let's say you are doing UI test automation with Selenium, and you have a bunch of Page Object classes that all extend a base class.

This base class has a method with which you can query a custom HTML attribute in your application, from a `WebElement` , in a more readable way:

```java
public String customDataAttrOf(WebElement element) {
    //return custom attribute value from 'element'
}
```

If you create a template for inserting this method e.g.:

```java
customDataAttrOf($WebElement$);
```

you can also configure the `$WebElement$` variable's Expression to be `suggestFirstVariableName("org.openqa.selenium.WebElement")` so that when inserted,
all the WebElement type variable names are shown as suggestions.

Variable names with the specified type are collected from super classes as well.

![suggestFirstVariableName_object_type](images/suggestFirstVariableName_non_primitive_type.gif)

If you pass a type into the macro but not with its fully qualified name, e.g. `"WebElement"` instead of `"org.openqa.selenium.WebElement"`, and you have
a custom type with the same name, then upon insertion, the suggestions will show variables only whose type is actually imported into the current class.

When you are aware of that multiple types with the same name might exist and be available in your project, it might be better to use fully qualified names,
otherwise the simple name of the type may be more feasible.

![suggestFirstVariableName_object_type_not_fully_qualified_name](images/suggestFirstVariableName_non_primitive_type_not_fully_qualified_name.gif) 

### Macros

Other macros that return a type can also be specified as the parameter of this macro, so that this one will suggest variable names
according to the returned type.

One example is [`methodReturnType()`](types.md#methodreturntype), in which case the Expression will be `suggestFirstVariableName(methodReturnType())` when configured in a template variable.

In case it doesn't find a variable with the returned type, then the suggestions list will be empty.

### Edge cases

This macro accepts exactly one parameter, so either there is no parameter, or multiple parameters specified the suggestions list will be ewmpty.

**Related macro:** [SuggestFirstVariableNameMacro](https://github.com/JetBrains/intellij-community/blob/master/java/java-impl/src/com/intellij/codeInsight/template/macro/SuggestFirstVariableNameMacro.java)

## suggestIndexName()

Suggests the name of an index variable from most commonly used ones: i, j, k, and so on (first one that is not used in the current scope).

For this I'm going to use the built-in template called **fori**:

```java
for(int $INDEX$ = 0; $INDEX$ < $LIMIT$; $INDEX$++) {
  $END$
}
```

where the Expression part of `$INDEX$` is `suggestIndexName()`.

The index suggestions start at **i** and goes all the way to **z**.

![suggest_index_name_default](images/suggest_index_name_default.gif)

In case the template (and along with that the macro) is invoked at a place where the outmost iterations index variable's name is "greater" (in terms of ASCII char index) than **i**,
e.g. j, k and so on, the name suggestion starts at **i** again.

![suggest_index_name_greater](images/suggest_index_name_greater.gif)

If the template is invoked in an iteration where the index variable name is **z**, from that point on the next suggested variable name will always be an empty string.

![suggest_index_name_beyond_z](images/suggest_index_name_beyond_z.gif)

**Related macro:** [SuggestIndexNameMacro](https://github.com/JetBrains/intellij-community/blob/master/java/java-impl/src/com/intellij/codeInsight/template/macro/SuggestIndexNameMacro.java)

## suggestVariableName()

Suggests the name for a variable based on the variable type and its initializer expression...

This example is based on the built-in template called **iter** whose template is:

```java
for ($ELEMENT_TYPE$ $VAR$ : $ITERABLE_TYPE$) {
  $END$
}
```

having the Expression part of `$VAR$` set to `suggestVariableName()`.

In case the source iterable's name is in plural form, then the suggested variable's name is going to be in singular form: 
 
![suggest_variable_name_plural](images/suggest_variable_name_plural.gif)

Though not necessarily related, when there are multiple iterables suggested, the `suggestVariableName()` macro will always suggests
a name according to the currently selected iterable.

![suggest_variable_name_multiple_iterable](images/suggest_variable_name_multiple_iterable.gif)

If the source iterable's name is not in plural the suggestion will either be a predefined name or the lowercase version of the
variable's name.

In certain cases when there would be name collision, the suggested variable name is indexed to make it unique.

| Type | Suggested variable name |
|---|---|
| String | (Collection item type + collection name -> suggested variable name)<br>String + integer -> s<br>String + string -> s<br>String + strings -> string<br>String + s -> s1<br>String + s1 -> s |
| Integer | <br>Integer + integer -> integer1<br>Integer + string -> integer<br>Integer + integers -> integer<br>Integer + i -> integer |
| Custom types | the lowercase version of the type: AnAwesomeClassName -> anawesomeclassname |

**Related macro:** [SuggestVariableNameMacro](https://github.com/JetBrains/intellij-community/blob/master/java/java-impl/src/com/intellij/codeInsight/template/macro/SuggestVariableNameMacro.java)

## time(sSystemTime)

Returns the current system time in the specified format.

The underlying logic is almost the same as in case of the `date(sDate)` macro because they are both handled in the `com.intellij.codeInsight.template.macro.CurrentDateMacro` class
in a common static method. The only difference is that in case of there is no parameter specified for `time(sSystemTime)`
the system default time is returned instead of the system default date.

For the sake of comprehensiveness I include all the cases from the `date(sDate)` macro here as well, of course extended with the `time` specific details.

Let's say, for some reason, you like to keep track when you did certain changes in your code and you'd like to insert the current time
as a comment or into your javadoc. For that a template might be:

```java
//Latest modification: $currentTime$
```

The only thing you need to configure is the Expression part of the `$currentTime$` variable which can be done multiple ways.

#### No time format

In case there is no parameter specific for the `time()` macro, as the official documentation also states
     
> the current time is returned in the default system format.

You can be on Mac, Unix or Windows, it will return the time in its default system format. All this logic is handled in `com.intellij.util.text.DateFormatUtil`.

![time_without_params](images/time_without_params.gif)

#### Single custom time format

If you'd like to use a different time format you can pass in a date-time format String value that is according to the `java.text.SimpleDateFormat` specification.

Using the format `"H.mm"` the result will change as following: 

![time_with_single_param](images/time_with_single_param.gif)

However nothing stops you from defining a complete date format as well because behind the scenes it uses a `SimpleDateFormat` to get the time.

`"Y-MM-d, E, H:m"`

![time_with_single_param_date_format](images/time_with_single_param_date_format.gif)

**NOTE:** this example is made on a Hungarian language system, so the current day abbreviation is according to that: V for Vasárnap, Sunday in Hungarian. 

#### Multiple macro parameters

The macro takes only one parameter and ignores all extra parameters, but according to the macro implementation if you pass in more than one parameters,
it acts as if there were no parameters specified at all, falling back to the default system format.

#### Invalid time format

If you specify a custom time format string but it is not valid according to the `SimpleDateFormat` specification, e.g. `"H:mmmXSAm"`,
it returns an error message defined in the macro implementation, `com.intellij.codeInsight.template.macro.CurrentDateMacro`.

![time_incorrect_format](images/time_incorrect_format.gif)

**Related macro:** [CurrentTimeMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/CurrentTimeMacro.java)

## user()

Returns the name of the current system user, more specifically the value of the `user.name` system property.

**Related macro:** [CurrentUserMacro](https://github.com/JetBrains/intellij-community/blob/master/platform/lang-impl/src/com/intellij/codeInsight/template/macro/CurrentUserMacro.java)

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

# annotated("annotation qname")

Adding this function with a certain annotation will show a suggestion list of available symbols that are annotated
as the specific annotation.

The search's scope is global, so it searches not just in the current project but provides suggestions from the classpath as well,
so this function is rather usable for cases where a rather project specific annotation needs to be targeted.

**Example (Spring Components):**

Having the following template:
```java
@javax.inject.Inject
public $type$ $name$;
```
and setting the `$type$` variable's  to `annotated("org.springframework.stereotype.Component")`, using the component
template keyword the templates gets injected to the code and a suggestions list opens with all available `@Component` annotated
types:

![annotated_component](images/annotated_component.png)

**Example (Spring Beans):**

The same template and configuration but targeting the `org.springframework.context.annotation.Bean` annotation will show a suggestion list
containing the names of the available `@Bean` annotated methods.

![annotated_bean](images/annotated_bean.png)

If you combine the `annotated()` function with e.g. a function that does some kind of string manipulation (i.e. `snakeCase()`), this latter function gets
automatically applied to the first item in the suggestions list without offering any suggestions.

**Related macro:** [AnnotatedMacro](https://github.com/JetBrains/intellij-community/blob/master/java/java-impl/src/com/intellij/codeInsight/template/macro/AnnotatedMacro.java)

**Articles:**
- https://stackoverflow.com/questions/34786643/intellij-idea-live-template-annotated-variable-location/34788167#34788167
- https://stackoverflow.com/questions/31110054/intellij-live-template-annotated-expression

**Example built-in templates:**
- RESTful Web Services -> cxf/Generate CXF Rest web service invocation

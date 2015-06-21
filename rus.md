<article class="post" role="article">

_[ES6 In Depth](https://hacks.mozilla.org/category/es6-in-depth/) is a series on new features being added to the JavaScript programming language in the 6th Edition of the ECMAScript standard, ES6 for short._

Today’s post is about two features that make JavaScript’s function syntax more expressive: rest parameters and parameter defaults.

### Rest parameters

A common need when creating an API is a _variadic function_, a function that accepts any number of arguments. For example, the [String.prototype.concat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/concat) method takes any number of string arguments. With rest parameters, ES6 provides a new way to write variadic functions.

To demonstrate, let’s write a simple variadic function `containsAll` that checks whether a string contains a number of substrings. For example, `containsAll("banana", "b", "nan")` would return `true`, and `containsAll("banana", "c", "nan")` would return `false`.

Here is the traditional way to implement this function:

    function containsAll(haystack) {
      for (var i = 1; i < arguments.length; i++) {
        var needle = arguments[i];
        if (haystack.indexOf(needle) === -1) {
          return false;
        }
      }
      return true;
    }

This implementation uses the magical [`arguments` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments), an array-like object containing the parameters passed to the function. This code certainly does what we want, but its readibility is not optimal. The function parameter list contains only one parameter `haystack`, so it’s impossible to tell at a glance that the function actually takes multiple arguments. Additionally, we must be careful to start iterating through `arguments` at index `1` not `0`, since `arguments[0]` corresponds to the `haystack` argument. If we ever wanted to add another parameter before or after `haystack`, we would have to remember to update the for loop. Rest parameters address both of these concerns. Here is a natural ES6 implementation of `containsAll` using a rest parameter:

    function containsAll(haystack, ...needles) {
      for (var needle of needles) {
        if (haystack.indexOf(needle) === -1) {
          return false;
        }
      }
      return true;
    }

This version of the function has the same behavior as the first one but contains the special `...needles` syntax. Let’s see how calling this function works for the invocation `containsAll("banana", "b", "nan")`. The argument `haystack` is filled as usual with the parameter that is passed first, namely `"banana"`. The ellipsis before `needles` indicates it is a _rest parameter_. All the other passed parameters are put into an array and assigned to the variable `needles`. For our example call, `needles` is set to `["b", "nan"]`. Function execution then continues as normal. (Notice we have used the ES6 [for-of](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/) looping construct.)

Only the last parameter of a function may be marked as a rest parameter. In a call, the parameters before the rest parameter are filled as usual. Any “extra” arguments are put into an array and assigned to the rest parameter. If there are no extra arguments, the rest parameter will simply be an empty array; the rest parameter will never be `undefined`.

### Default parameters

Often, a function doesn’t need to have all its possible parameters passed by callers, and there are sensible defaults that could be used for parameters that are not passed. JavaScript has always had a inflexible form of default parameters; parameters for which no value is passed default to `undefined`. ES6 introduces a way to specify arbitrary parameter defaults.

Here’s an example. (The backticks signify template strings, which were [discussed last week](https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2/).)

    function animalSentence(animals2="tigers", animals3="bears") {
        return `Lions and ${animals2} and ${animals3}! Oh my!`;
    }

For each parameter, the part after the `=` is an expression specifying the default value of the parameter if a caller does not pass it. So, `animalSentence()` returns `"Lions and tigers and bears! Oh my!"`, `animalSentence("elephants")` returns `"Lions and elephants and bears! Oh my!"`, and `animalSentence("elephants", "whales")` returns `"Lions and elephants and whales! Oh my!"`.

The are several subtleties related to default parameters:

*   Unlike Python, **default value expressions are evaluated at function call time** from left to right. This also means that default expressions can use the values of previously-filled parameters. For example, we could make our animal sentence function more fancy as follows:

        function animalSentenceFancy(animals2="tigers",
            animals3=(animals2 == "bears") ? "sealions" : "bears")
        {
          return `Lions and ${animals2} and ${animals3}! Oh my!`;
        }

    Then, `animalSentenceFancy("bears")` returns `"Lions and bears and sealions. Oh my!"`.

*   Passing `undefined` is considered to be equivalent to not passing anything at all. Thus, `animalSentence(undefined, "unicorns")` returns `"Lions and tigers and unicorns! Oh my!"`.
*   A parameter without a default implicitly defaults to undefined, so

        function myFunc(a=42, b) {...}

    is allowed and equivalent to

        function myFunc(a=42, b=undefined) {...}

### Shutting down `arguments`

We’ve now seen that rest parameters and defaults can replace usage of the `arguments` object, and removing `arguments` usually makes the code nicer to read. In addition to harming readibility, the magic of the `arguments` object notoriously causes [headaches for optimizing JavaScript VMs](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#3-managing-arguments).

It is hoped that rest parameters and defaults can completely supersede `arguments`. As a first step towards this, functions that use a rest parameter or defaults are forbidden from using the `arguments` object. Support for `arguments` won’t be removed soon, if ever, but it’s now preferable to avoid `arguments` with rest parameters and defaults when possible.

### Browser support

Firefox has had support for rest parameters and defaults since version 15.

Unfortunately, no other released browser supports rest parameters or defaults yet. V8 recently [added experimental support for rest parameters](https://code.google.com/p/v8/issues/detail?id=2159), and there is an open V8 [issue for implementing defaults](https://code.google.com/p/v8/issues/detail?id=2160). JSC also has open issues for [rest parameters](https://bugs.webkit.org/show_bug.cgi?id=38408) and [defaults](https://bugs.webkit.org/show_bug.cgi?id=38409).

The [Babel](http://babeljs.io/) and [Traceur](https://github.com/google/traceur-compiler#what-is-traceur) compilers both support default parameters, so it is possible to start using them today.

### Conclusion

Although technically not allowing any new behavior, rest parameters and parameter defaults can make some JavaScript function declarations more expressive and readable. Happy calling!

* * *

_Note: Thanks to Benjamin Peterson for implementing these features in Firefox, for all his contributions to the project, and of course for this week’s post._

Next week, we’ll introduce another simple, elegant, practical, everyday ES6 feature. It takes the familiar syntax you already use to write arrays and objects, and turns it on its head, producing a new, concise way to _take arrays and objects apart._ What does that mean? Why would you want to take an object apart? Join us next Thursday to find out, as Mozilla engineer [Nick Fitzgerald](https://twitter.com/fitzgen) presents ES6 destructuring in depth.

Jason Orendorff

[ES6 In Depth](https://hacks.mozilla.org/category/es6-in-depth/ "ES6 In Depth") editor

<footer class="entry-meta">

Posted by [Benjamin Peterson](https://hacks.mozilla.org/author/bpbenjamin-pe/ "Posts by Benjamin Peterson") on <time datetime="2015-05-21T13:32:10-07:00">May 21, 2015</time> at <time datetime="PDT13:32:10-07:00">13:32</time>

</footer>

</article>
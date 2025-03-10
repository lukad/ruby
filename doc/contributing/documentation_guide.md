# Documentation Guide

This guide discusses recommendations for documenting
classes, modules, and methods
in the Ruby core and in the Ruby standard library.

## Generating documentation

Most Ruby documentation lives in the source files and is written in [RDoc format](https://ruby.github.io/rdoc/RDoc/Markup.html).

Some pages live under the `doc` folder and can be written in either `.rdoc` or `.md` format, determined by the file extension.

To generate the output of documentation changes in HTML in the `{build folder}/.ext/html` directory, run the following inside your build directory:

```
make html
```

Then you can preview your changes by opening `{build folder}/.ext/html/index.html` file in your browser.


## Goal

The goal of Ruby documentation is to impart the most important
and relevant in the shortest time.
The reader should be able to quickly understand the usefulness
of the subject code and how to use it.

Providing too little information is bad, but providing unimportant
information or unnecessary examples is not good either.
Use your judgment about what the user needs to know.

## General Guidelines

- Keep in mind that the reader may not be fluent in \English.
- Write short declarative or imperative sentences.
- Group sentences into (ideally short) paragraphs,
  each covering a single topic.
- Organize material with [headers](rdoc-ref:RDoc::Markup@Headers).
- Refer to authoritative and relevant sources using
  [links](rdoc-ref:RDoc::Markup@Links).
- Use simple verb tenses: simple present, simple past, simple future.
- Use simple sentence structure, not compound or complex structure.
- Avoid:
    - Excessive comma-separated phrases;
      consider a [list](rdoc-ref:RDoc::Markup@Simple+Lists).
    - Idioms and culture-specific references.
    - Overuse of headers.
    - Using US-ASCII-incompatible characters in C source files;
      see [Characters](#label-Characters) below.

### Characters

Use only US-ASCII-compatible characters in a C source file.
(If you use other characters, the Ruby CI will gently let you know.)

If want to put ASCII-incompatible characters into the documentation
for a C-coded class, module, or method, there are workarounds
involving new files <tt>doc/*.rdoc</tt>:

- For class `Foo` (defined in file <tt>foo.c</tt>),
  create file <tt>doc/foo.rdoc</tt>, declare <tt>class Foo; end</tt>,
  and place the class documentation above that declaration:

    ```ruby
    # Documentation for class Foo goes here.
    class Foo; end
    ```

- Similarly, for module `Bar` (defined in file <tt>bar.c</tt>,
  create file <tt>doc/bar.rdoc</tt>, declare <tt>module Bar; end</tt>,
  and place the module documentation above that declaration:

    ```ruby
    # Documentation for module Bar goes here.
    module Bar; end
    ```

- For a method, things are different.
  Documenting a method as above disables the "click to toggle source" feature
  in the rendered documentation.

    Therefore it's best to use file inclusion:
    
    - Retain the call-seq in the C code.
    - Use file inclusion (`:include:`) to include text from an .rdoc file.

    Example:

    ```c
    /*
     *  call-seq:
     *    each_byte {|byte| ... } -> self
     *    each_byte               -> enumerator
     *
     *  \:include: doc/string/each_byte.rdoc
     *
     */
    ```

### \RDoc

Ruby is documented using RDoc.
For information on \RDoc syntax and features, see the
[RDoc Markup Reference](rdoc-ref:RDoc::Markup@RDoc+Markup+Reference).

### Output from <tt>irb</tt>

For code examples, consider using interactive Ruby,
[irb](https://ruby-doc.org/stdlib/libdoc/irb/rdoc/IRB.html).

For a code example that includes `irb` output,
consider aligning <tt># => ...</tt> in successive lines.
Alignment may sometimes aid readability:

```ruby
a = [1, 2, 3] #=> [1, 2, 3]
a.shuffle!    #=> [2, 3, 1]
a             #=> [2, 3, 1]
```

### Headers

Organize a long discussion with [headers](rdoc-ref:RDoc::Markup@Headers).

### Blank Lines

A blank line begins a new paragraph.

A [code block](rdoc-ref:RDoc::Markup@Paragraphs+and+Verbatim)
or [list](rdoc-ref:RDoc::Markup@Simple+Lists)
should be preceded by and followed by a blank line.
This is unnecessary for the HTML output, but helps in the `ri` output.

### Auto-Linking

In general, \RDoc's auto-linking should not be suppressed.
For example, we should write `Array`, not <tt>\Array</tt>.

We might consider whether to suppress when:

- The word in question does not refer to a Ruby entity
  (e.g., some uses of _Class_ or _English_).
- The reference is to the current class document
  (e.g., _Array_ in the documentation for class `Array`).
- The same reference is repeated many times
  (e.g., _RDoc_ on this page).

## Documenting Classes and Modules

The general structure of the class or module documentation should be:

- Synopsis
- Common uses, with examples
- "What's Here" summary (optional)

### Synopsis

The synopsis is a short description of what the class or module does
and why the reader might want to use it.
Avoid details in the synopsis.

### Common Uses

Show common uses of the class or module.
Depending on the class or module, this section may vary greatly
in both length and complexity.

### What's Here Summary

The documentation for a class or module may include a "What's Here" section.

Guidelines:

- The section title is <tt>What's Here</tt>.
- Consider listing the parent class and any included modules; consider
  [links](rdoc-ref:RDoc::Markup@Links)
  to their "What's Here" sections if those exist.
- List methods as a bullet list:

    - Begin each item with the method name, followed by a colon
      and a short description.
    - If the method has aliases, mention them in parentheses before the colon
      (and do not list the aliases separately).
    - Check the rendered documentation to determine whether \RDoc has recognized
      the method and linked to it;  if not, manually insert a
      [link](rdoc-ref:RDoc::Markup@Links).

- If there are numerous entries, consider grouping them into subsections with headers.
- If there are more than a few such subsections,
  consider adding a table of contents just below the main section title.

## Documenting Methods

### General Structure

The general structure of the method documentation should be:

- Calling sequence (for methods written in C).
- Synopsis (short description).
- Details and examples.
- Argument description (if necessary).
- Corner cases and exceptions.
- Aliases.
- Related methods (optional).

### Calling Sequence (for methods written in C)

For methods written in Ruby, \RDoc documents the calling sequence automatically.

For methods written in C, \RDoc cannot determine what arguments
the method accepts, so those need to be documented using \RDoc directive
[:call-seq:](rdoc-ref:RDoc::Markup@Method+arguments).

Example:

```
*  call-seq:
*    array.count -> integer
*    array.count(obj) -> integer
*    array.count {|element| ... } -> integer
```

When creating the <tt>call-seq</tt>, use the form

```
receiver_type.method_name(arguments) {|block_arguments|} -> return_type
```

Omit the parentheses for cases where the method does not accept arguments,
and omit the block for cases where a block is not accepted.

In the cases where method can return multiple different types, separate the
types with "or".  If the method can return any type, use "object".  If the
method returns the receiver, use "self".

In cases where the method accepts optional arguments, use a <tt>call-seq</tt>
with an optional argument if the method has the same behavior when an argument
is omitted as when the argument is passed with the default value.  For example,
use:

```
obj.respond_to?(symbol, include_all=false) -> true or false
```

Instead of:

```
obj.respond_to?(symbol) -> true or false
obj.respond_to?(symbol, include_all) -> true or false
```

However, as shown above for <tt>Array#count</tt>, use separate lines if the
behavior is different if the argument is omitted.

Omit aliases from the call-seq, but mention them near the end (see below).


A `call-seq` block should have <tt>{|x| ... }</tt>, not <tt>{|x| block }</tt> or <tt>{|x| code }</tt>.

A `call-seq` output should:

- Have `self`, not `receiver` or `array`.
- Begin with `new_` if and only if the output object is a new instance
  of the receiver's class, to emphasize that the output object is not `self`.

### Synopsis

The synopsis comes next, and is a short description of what the
method does and why you would want to use it.  Ideally, this
is a single sentence, but for more complex methods it may require
an entire paragraph.

For <tt>Array#count</tt>, the synopsis is:

```
Returns a count of specified elements.
```

This is great as it is short and descriptive.  Avoid documenting
too much in the synopsis, stick to the most important information
for the benefit of the reader.

### Details and Examples

Most non-trivial methods benefit from examples, as well as details
beyond what is given in the synopsis.  In the details and examples
section, you can document how the method handles different types
of arguments, and provides examples on proper usage.  In this
section, focus on how to use the method properly, not on how the
method handles improper arguments or corner cases.

Not every behavior of a method requires an example.  If the method
is documented to return `self`, you don't need to provide an example
showing the return value is the same as the receiver.  If the method
is documented to return `nil`, you don't need to provide an example
showing that it returns `nil`.  If the details mention that for a
certain argument type, an empty array is returned, you don't need
to provide an example for that.

Only add an example if it provides the user additional information,
do not add an example if it provides the same information given
in the synopsis or details.  The purpose of examples is not to prove
what the details are stating.

### Argument Description (if necessary)

For methods that require arguments, if not obvious and not explicitly
mentioned in the details or implicitly shown in the examples, you can
provide details about the types of arguments supported.  When discussing
the types of arguments, use simple language even if less-precise, such
as "level must be an integer", not "level must be an Integer-convertible
object".  The vast majority of use will be with the expected type, not an
argument that is explicitly convertible to the expected type, and
documenting the difference is not important.

For methods that take blocks, it can be useful to document the type of
argument passed if it is not obvious, not explicitly mentioned in the
details, and not implicitly shown in the examples.

If there is more than one argument or block argument, use a
[labeled list](rdoc-ref:RDoc::Markup@Labeled+Lists).

### Corner Cases and Exceptions

For corner cases of methods, such as atypical usage, briefly mention
the behavior, but do not provide any examples.

Only document exceptions raised if they are not obvious.  For example,
if you have stated earlier than an argument type must be an integer,
you do not need to document that a \TypeError is raised if a non-integer
is passed.  Do not provide examples of exceptions being raised unless
that is a common case, such as \Hash#fetch raising a \KeyError.

### Aliases

Mention aliases in the form

```c
// Array#find_index is an alias for Array#index.
```

### Related Methods (optional)

In some cases, it is useful to document which methods are related to
the current method.  For example, documentation for `Hash#[]` might
mention `Hash#fetch` as a related method, and `Hash#merge` might mention
`Hash#merge!` as a related method.  Consider which methods may be related
to the current method, and if you think the reader would benefit it,
at the end of the method documentation, add a line starting with
"Related: " (e.g. "Related: #fetch").  Don't list more than three
related methods. If you think more than three methods are related,
pick the three you think are most important and list those three.

### Methods Accepting Multiple Argument Types

For methods that accept multiple argument types, in some cases it can
be useful to document the different argument types separately.  It's
best to use a separate paragraph for each case you are discussing.

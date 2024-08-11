# subtex (*sub*stitute *tex*t): a text editor for LLMs

LLMs need a simple and structured way to make edits to text files. Rewriting entire files can be error prone, slow, and increases the context length. On the other hand, manually implementing a series of small edits can be time-consuming, and the intended location of edits can be unclear. Commands like sed, cat, and :%s can help, but can be error-prone without being careful, and complex for large edits.

The ideal program should be simple enough for an LLM to fully understand with a short explanation, robust enough to make whatever edits are necessary, and easy enough for humans to incorporate.

This program should have a way for LLMs to indicate what text they want to edit, or where they want to insert. It should be able to find sections, functions, or exact matches to text. It should be able to replace and insert text precisely, and be able to see the text that it modified, and the result of the modification.

Complexity should be abstracted from the LLM and the human. If the LLM makes a mistake, it is subtex’s fault for not being clear and intuitive enough for the LLM, or not handling the case properly.

```find``` should find a selection of text for the next part of the command, and also print the text. 

```replace``` must come after find, and replaces the found text.

```insert``` must come after find, and inserts text after the found text.

There should be no need for escaping, since that is too complex. Instead, we should have a character to enclose what we wish to find and replace which is very unique (humans do not need to type it, so we can use an uncommon unicode character). The examples here use regular quotation marks, since I have not decided on which character to use yet. Here are some options:

ʺ → U+02BA

■ → U+25A0

❝ and ❞ → U+275D and U+275E

◆ → U+25C6


I have not decided whether to write subtex in Rust or Python. I also have not decided on the best way to implement “undo” or automatic file backups.

**The ```on``` command**

Specifies the file to edit. ```on [file path]``` is a standalone command that will allow subsequent commands to know to make edits at [file path]. It can be updated or changed by just writing a new command, ```on [other file path]```.


**The ```find``` command:**

The find command finds text.

```find part “[line containing beginning of function or section]”``` should find the entire function or section by looking at brackets or parentheses (or fallback to indentation if there are no brackets or parentheses). For example, ```find part “.nav-links a {”``` could find the following:

```
    .nav-links a {
        margin-left: 1rem;
        margin-right: 1rem;
    }
```

```find exact “[verbatim text]”``` should find only the text. For example, ```find exact “.nav-links a {”``` could find the following:

```
.nav-links a {
```

```find all [piece of text]``` should not exist, since LLMs may not intend the full consequences of finding and replacing all occurrences of a string, and should instead just make a series of smaller edits.



**The ```replace``` command:**

The replace command replaces found text. ```replace``` and ```replace with``` are synonyms. Commas are optional.

For example, ```find exact “[text]”, replace with “[new text]”``` should find “text” and replace it with “new text”. Or, ```find “margin-left: 1rem”, replace with [margin-left: 2rem]``` would change the margin-left from 1rem to 2rem.

However, if there are multiple instances of “margin-left: 1rem” in the file, we should not make any changes, and instead print a message displaying why no changes were made, and showing both instances of “margin-left: 1rem” in context. We want to make sure the LLM is specific and intentional about the edits it is making.

For example,
```
find part “.nav-links a {”, replace with “    .nav-links a {
        margin-right: 0.2rem;
    }

”
```
could find the .nav-links a section, and replace it with:

```
    .nav-links a {
        margin-right: 0.2rem;
    }
```

A replace command without the indentation should act the same as the above example, by understanding that there is implied intended indentation: ```find part “.nav-links a {”, replace with “.nav-links a {
    margin-right: 0.2rem;
}”``` should result in the following (assuming the file has the ```.nav-links a``` section indented once):
```
    .nav-links a {
        margin-right: 0.2rem;
    }
```

**The ```insert``` command:**

The ```insert``` command inserts text after found text.

For example, ```find exact “[new]”, insert “[text]”``` should find “new” and insert “text,” which will result in “new text.”

However, if there are multiple instances of “new” in the file, we should not make any changes, and instead print a message displaying why no changes were made, and showing all instances of “new” in context. We want to make sure the LLM is specific and intentional about the edits it is making.

For example,
```
find exact
“    .sticky-nav {
        flex-direction: column;
        padding: 1rem;
    }”, insert “

    .nav-links {
        margin-top: 1rem;
    }”
```

should result in:

```
    .sticky-nav {
        flex-direction: column;
        padding: 1rem;
    }

    .nav-links {
        margin-top: 1rem;
    }
```

The insert command, similar to replace, should understand when there is implied intended indentation, and should automatically apply the same level of indentation (if not already applied) as the last line of the found text.

For example, 	```find part “.sticky-nav {”, insert “
.nav-links {
    margin-top: 1rem;
}”``` should result in the following (assuming the ```.sticky-nav``` section was indented once):

```
    .sticky-nav {
        flex-direction: column;
        padding: 1rem;
    }

    .nav-links {
        margin-top: 1rem;
    }
```

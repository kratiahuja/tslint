---
layout: page
title: Testing Rules
permalink: /develop/testing-rules/
---

Every TSLint rule has a corresponding directory inside `test/rules/` which contains one or more test cases for the rule.

Each test case contains:

* A `tslint.json` file which specifies the configuration for TSLint to use
* `.ts.lint` test files which contain TypeScript code and a special markup which indicate where lint failures should be found  

The test system lints the `.ts.lint` files with the settings specified in `tslint.json` and makes sure that failures generated by the TSLint match the failures marked in the `.ts.lint` files.

### Example ###

##### Testing a New Rule #####

Say we're contributing a new rule to TSLint that bans the use of animal variable names and we now need to test it. First we create our configuration file which enables only the rule to be tested:

`test/rules/no-animal-variable-names/default/tslint.json`:

```
{
  "rules": {
    "no-animal-variable-names": true
  }
}
```

In this case, we placed our configuration inside `no-animal-variable-names/default` to indicate that we're testing the default configuration for our rule.

Now let's make the actual test file:

`test/rules/no-animal-variable-names/default/test.ts.lint`:

``` 
const octopus = 5;
      ~~~~~~~      [Variables named after animals are not allowed!]

let giraffe: number, tiger: number;
    ~~~~~~~                 [Variables named after animals are not allowed!]
                     ~~~~~  [Variables named after animals are not allowed!]

const tree = 5;
const skyscraper = 100;
```

In the above file, `~` characters "underline" where our rule should generate a lint failure
and the message that should be produced appears in brackets afterwards.

If multiple lint failures occur on the same line of TypeScript code, the markup for them is placed on consecutive lines,
as shown in the above example with the line `let giraffe: number, tiger: number;`

Notice how we also include lines of code that *shouldn't* generate lint failures.
This is important to ensure that the rule isn't creating false-positives.

We can now run `grunt test` to make sure that our rule produces the output we expect.

##### Testing a New Rule Option #####

Let's look at one more example. Say we've added an `also-no-plants` option to our rule above that disallows variable names that are plants. We should add a test for this new option:

`test/rules/no-animal-variable-names/also-no-plants/tslint.json`:

```
{
  "rules": {
    "no-animal-variable-names": [true, "also-no-plants"]
  }
}
```

`test/rules/no-animal-variable-names/also-no-plants/test.ts.lint`:

``` 
const octopus = 5;
      ~~~~~~~     [no-animal]

let giraffe: number, tiger: number;
    ~~~~~~~                 [no-animal]
                     ~~~~~  [no-animal]

const tree = 5;
      ~~~~      [no-plant]
const skyscraper = 100;

[no-animal]: Variables named after animals are not allowed!
[no-plant]: Variables named after plants are not allowed!
```

We've now used a special message shorthand syntax so we don't have to type out the same failure message over and over.
Instead of writing out the full lint failure message after each occurance of it, we instead just specify a shortcut name.
(Shortcut names can only consist of letters, numbers, underscores, and hyphens.)
Then, at the bottom of our test file, we specify what full message each shortcut should expand to.

Again, we can run `grunt test` to make sure our rule is producing the output we expect. If it isn't we'll see the difference between the output from the rule and the output we marked.


### Tips & Tricks ###

* You can use this system to test rules outside of the TSLint build! Use the `tslint --test path/to/dir` command to test your own custom rules.
The directory you pass should contain a `tslint.json` file and `.ts.lint` files. You can try this out on the TSLint rule test cases, for example, `tslint --test path/to/tslint-code/test/rules/quotemark/single`.

* Lint failures sometimes span over multiple lines. To handle this case, don't specify a message until the end of the error. For example:

```
for (let key in obj) {
~~~~~~~~~~~~~~~~~~~~~~
  console.log(key);
~~~~~~~~~~~~~~~~~~~
}
~ [for-in loops are not allowed]
```

* If for some weird reason your lint rule generates a failure that has zero width, you can use the `~nil` mark to indicate this.
In all reality though, you shouldn't have rules that do this, and this feature is more of a carryover to accomodate testing
some of TSLint's legacy rules.


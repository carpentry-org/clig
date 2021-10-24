# clig

A carp command line library that follows the [command line
guidelines](https://clig.dev). This library takes significant inspiration from
[cli.carp](https://github.com/carpentry-org/cli.carp), however, it uses no
macros and prefers an imperative API over the more functional, combinator-based
API of cli.carp. It also eschews richer functionality for a simpler experience.
For example, this library does not provide any means of making flags required or
defining custom validation for flag argument values--instead those tasks are
left up to the caller to perform themselves after parsing is successful. 

## Usage

To get started, define a clig program using `new`. Give it a name and brief
usage synopsis.

```clojure
(def my-program (Clig.new @"my-program" @"doesn't do anything interesting"))
```

Next, define command line flags you want your program to support. Each flag must
have a name, default value and usage description and can optionally have a
shorthand name. clig supports defining flags of four types that correspond to
the carp types, String, Bool, Int, and Float. clig will validate that user input
is for a flag is a valid instance of the flag's type but performs no other input
validation:

```clojure
(def my-bool-flag (Clig.bool-flag @"force" 
                                  (Maybe.Just @"b") 
                                  @"false" 
                                  @"abandon all safety"))
(def my-int-flag (CLig.int-flag @"count" 
                                (Maybe.Nothing) 
                                @"1" 
                                @"repeat n times"))
```

Finally, add your flags to the clig program you defined, and call parse to
process command line flags. When parsing is successful, clig returns
`(Result.Success ())`. If parsing is unsuccessful, clig returns `(Result.Error
CligErr)` where the contained error is the last error encountered by the parser.
To stop parsing immediately on an error, pass `true` as the second argument to
`parse`, to keep going, pass `false`. All the parsing errors clig encounters are
collected in your program's `errors` field.

```clojure
(defn main []
  (do (Clig.add-flag &p &my-bool-flag)
      (Clig.add-flag &p &my-int-flag)
      (match (Clig.parse &p true)
        (Result.Error err) (IO.println &(str err))
        (Result.Success _)
          ;; your program logic here  
        )))
```

After parsing, your program's `flags` field will contain its flags filled with
values. Each flag's value field will have the value passed by the user as a flag
argument, whether  or not it's valid. You can use `get` to obtain the value of a
flag by name directly if it was passed, and an empty string otherwise:

```clojure
(Clig.get &p "force")
```

After calling parse, your clig program's `args` field will contain all non-flag
arguments (excepting the program name). 

To print a usage string for your program use `Clig.usage`.

Thus ends our tour of the basics, for more information see the API documentation
and program source.

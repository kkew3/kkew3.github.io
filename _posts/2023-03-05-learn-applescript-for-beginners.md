---
layout: post
title:  "Learn Applescript for Beginners"
date:   2023-03-05 15:54:00 +0800
tags:   dev--applescript
---

<!-- TODO: proofread -->

> [youtube link](https://www.youtube.com/playlist?list=PL5iB9WEZe2j04doUAKVxqxaCv3JJ8TxQr)

> Good referential text:
>
> - [AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html)
> - [AppleScript Language Guide (PDF)](https://applescriptlibrary.files.wordpress.com/2013/11/applescriptlanguageguide-2013.pdf)

## Script editor

`log "message"` can be used to log messages that will be shown in "Messages" accessory pane at the bottom.

## File formats

- Text: save as plain text
- Script: save as compiled library
- Application: used to create applet;
    If in code `on open(droppedFiles) ... end open` is written, then it becomes a dropplet, which allows drag-n-drop action on the created applet.

## Dictionaries

Words inside square bracket are optional arguments.
Words on the right of right arrow is what will be returned.
What will be returned can be caught by setting it to `set variable to`.
For example,

```applescript
set myDoc to make new document
```

## Variables

```applescript
on run
    set theName to "Jill"
    log theName
end run
```

### Scope of variable

By default within current handler, e.g. `on run` handler, `on sub_handler()` handler (subroutine).
However, if claiming a variable as property at the top of script:

```applescript
property theName : null
```

`theName` will become a global variable.

## Data types

- number (integer or real) `set x to 23.25`
- string: `set x to "23.25"`; but `set x to "23.25" as number` cast `x` again to a number
- date: `set x to date "3/5/2023"` means set it to 2023-03-05
- list: `set x to {"item1", "item2", "item3"}`. The list may comprise of heterogeneous types
- record (dictionary): `set x to {keyA: "valueA", keyB: valueB}`

Examples:

```applescript
set var1 to 22.5
set var2 to 2
set theResult to var1 + var2  # may be +, -, *, /, mod
```

```applescript
set var1 to "hello"
set var2 to "world"
set theResult to var1 & space & var2 & "!"  # string concatenation
```

```applescript
set var1 to date "12/22/2021"
set var2 to date "12/24/2021"
set theResult to var1 + (2 * days)  # may be days, hours, etc
set theResult to year of var1  # may be year of, month of, day of, etc
```

```applescript
set var1 to {"a", 23.9, "c"}
set var2 to item 2 of var1 as string  # get "23.9", indexed from 1
```

```applescript
set var1 to {FirstName: "Jon", LastName: "Voight"}
set theResult to FirstName of var1 & space & LastName of var1

# First and Last are reserved words, to use them as keys, surround
# with pipes (`|`), e.g.:
set var1 to {|First|: "Jon", |Last|: "Voight"}
set theResult to |First| of var1 & space |Last| of var2
```

## First script

```applescript
on run  # explicit on run handler, responding to double clicking
    delay 2  # delay for 2 seconds
    activate  # activate the script
    display dialog "Hello World!"
end run

on open (theFiles)  # on open handler, responding to drag-n-drop
    repeat with aFile in theFiles
        set myText to read file aFiles
        display dialog myText
    end repeat
end open

# to use on idle handler, save it with "Stay open after run handler"
on idle
    activate
    display dialog "Join us soon"
    return 3  # rerun this block 3 seconds later until manualy quit
end idle
```

## Commenting

Block comment:

```applescript
(*
    Version History
    This is the first build of my script
*)
```

in line comment: led by `--` or `#`

## Repeat loops

```applescript
on run
    repeat 3 times
    end repeat

    repeat with i from 1 to 3
    end repeat

    set myList to {"Jason", "Joan", "Jack"}
    repeat with anItem in myList
        display dialog "Hello " & anItem as string
    end repeat

    set test to true
    set i to 1
    repeat while test = true
        if i >= 4 then
            set test to false
        end if
        set i tto i + 1
    end repeat
end run
```

Use `exit repeat` to break out of `repeat` earlier.

## Conditionals

```applescript
on run
    set x to 6
    if x = 6 then
        display dialog "x is 6"
    else if x = 5 then
        display dialog "x is 5"
    else
        display dialog "x is neither 5 nor 6"
    end if
end run
```

## Error handling

```applescript
on run
    try  # ignore quietly errors and break out of try block
        set myDemo to "Hello"
        display dialog myTest
    end try

    try
        display dialog myTest
    on error errName
        display dialog errName
    end try

    try
        if myDemo = "Hello" then
            # this will raise error "message" with errName assigned "message"
            error "message"
            # or phrased as
            # error "message" number -1000
        end if
    on error errName number n
        display dialog errName & return & "with number " & n
    end try
end run
```

## Aias, HFS and POSIX

```applescript
on run
    set posixPath to "/Users/user/Desktop/name.jpg"

    # converts a POSIX path to an HFS file reference
    set hfsFilePathRef to posix file posixPath

    # converts a POSIX path to an HFS file path
    set hfsFilePath to posix file posixPath as string

    # cannot convert POSIX path to alias directly
    set aliasExample to hfsPath as alias

    # convert an HFS path to a POSIX path
    set backToPosix to posix path of hfsFilePath
end run
```

## Handlers (aka functions)

A handler is a collection of applescript statements that you give a descriptive name.

```applescript
on run
    set theResult to doMath(8, 2, "+")
    log theResult
end run

on doMath(num1, num2, mathFunc)
    try
	    if mathFunc = "+" then
	        return num1 + num2
	    else if mathFunc = "-" then
	        return num1 - num2
	    else if mathFunc = "*" then
	        return num1 * num2
	    else if mathFunc = "/" then
	        return num1 / num2
	    else
	        error "You must supply a proper math function"
	on error e
	    activate
	    display dialog (e as string) giving up after 8
	end try
end doMath
```

Note that `giving up after N` means the dialog will disappear itself if not clicking the dialog button after N seconds.

Within `tell application` block, be sure to call custom handler with `my` keyword.
For example,

```applescript
tell application "Numbers"
    set theResult to my doMath(8, 2, "-")
end tell
```

## Quit handler

```applescript
on run
    set someCondition to false
    if someCondition then
        # tell the script to quit;
		# this will trigger the `on quit` handler if present
        quit
    end if
end run

on quit
    # write cleanup actions your script should run before quitting
    activate
    display dialog "I quit" giving up after 4
    # this will quit current script immediately; without this statement
	# previous `quit` statement will be caught by `on quit` block and
	# quit won't be performed
    continue quit
end quit
```

## Case study: most recent modified file from a folder

```applescript
on run
    set thePath to "/Users/user/Desktop/folder"
    set newestFile to getNewestFile(thePath)
    return newestFile
end run

on getNewestFile(thePath)
    try
        set posixPath to my convertPathTo(thePath, "POSIX")
        set theFile to do shell script "ls -tp " & quoted form of posixPath & " | grep -Ev '/' | head -n1"
        if theFile is not equal to "" then
            set theFile to posixPath & "/" & theFile
        end if
        return theFile
    on error e
        return ""
    end try
end getNewestFile

on convertPathTo(inputPath, requestedForm)
    try
        set standardPosixPath to POSIX path of inputPath as string
        if requestedForm contains "posix" then
            set transformedPath to POSIX path of standardPosixPath as string
            if transformedPath ends with "/" then
                set transformedPath to character 1 thru -2 of transformedPath as string
            end if
        else if requestedForm contains "alias" then
            set transformedPath to POSIX file (standardPosixPath) as string
            try
                set transformedPath to transformedPath as alias
            on error
                error "The file \"" & transformedPath & "\" doesn't exist and can't be returned as \"alias\""
            end try
        else if requestedForm contains "hfs" then
            set transformedPath to POSIX file (standardPosixPath) as string
        else
            error "Requested path transformation type was an unexpected type"
        end if
        return transformedPath
    on error e
        return false
    end try
end convertPathTo
```

## Case study: automatically scale images

```applescript
on run
    set filePath to "/Users/user/Desktop/test.png"
    resizeImageWidth(fielPath, 450)
end run

on resizeImageWidth(filePath, pxls)
    try
        do shell script "sips --resampleWidth " & pxls & space & quoted form of filePath
        return true
    on error e
        # TODO do something with the error
        return false
    end try
end resizeImageWidth
```

## Case study: simple hot folder creation

A hot folder is a folder where a script monitor whatever is drag-n-dropped to that folder and perform actions on the object.

```applescript
on run
    # any startup activities required to run this script can be done here
end run

on idle
    set input to "/Users/user/Desktop/Hot Folder"
    set output to "/Users/user/Desktop/Results"
    set errors to "/Users/user/Desktop/Errors"

    set filePaths to getFiles(input)
    if filePaths is not equal to {} then
        repeat with filePath in filePaths
            if resizeImageWidth(filePath as string, output, 450) then
                removeFile(filePath as string)
            else
                do shell script "mv -f " & quoted form of filePaht & space & quoted form of errors
            end if
        end repeat
    end if
    return 5
end idle

on convertPathTo(inputPath, requestedForm)
    try
        set standardPosixPath to POSIX path of inputPath as string
        if requestedForm contains "posix" then
            set transformedPath to POSIX path of standardPosixPath as string
            if transformedPath ends with "/" then
                set transformedPath to character 1 thru -2 of transformedPath as string
            end if
        else if requestedForm contains "alias" then
            set transformedPath to POSIX file (standardPosixPath) as string
            try
                set transformedPath to transformedPath as alias
            on error
                error "The file \"" & transformedPath & "\" doesn't exist and can't be returned as \"alias\""
            end try
        else if requestedForm contains "hfs" then
            set transformedPath to POSIX file (standardPosixPath) as string
        else
            error "Requested path transformation type was an unexpected type"
        end if
        return transformedPath
    on error e
        return false
    end try
end convertPathTo

on stringToList(inputString, theDelimiter)
    try
        set tid to AppleScript's text item delimters
        set AppleScript's text item delimiters to theDelimiter
        set theList to text items of (inputString as string)
        set AppleScript's text item delimiters to tid
        return theList
    on error e
        set AppleScript's text item delimiters to tid
        return {}
    end try
end stringToList

on resizeImageWidth(filePath, output, pxls)
    try
        do shell script "sips --resampleWidth " & pxls & space & quoted form of filePath & " -o " & qutoed form of output
        return true
    on error e
        return false
    end try
end resizeImageWidth

on getFiles(thePath)
    try
        set posixPath to my convertPathTo(thePath, "posix")
        set theFiles to do shell script "find " & quoted form of posixPath & " -type f ! -name \".*\""
        if theFiles is not equal to "" then
            set fileList to stringToList(theFiles, return)
        end if
        return fileList
    on error e
        # log the error here at some point
        return {}
    end try
end getFiles

on removeFile(theFile)
    try
        set posixPath to convertPathTo(theFile, "posix")
        do shell script "rm -f " & quoted form of posixPath
	    return true
    on error e
        return false
    end try
end removeFile
```

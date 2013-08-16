> Note: This documentation is for the [phpcs-fixer](https://github.com/squizlabs/PHP_CodeSniffer/tree/phpcs-fixer) branch only. Everything described here is still experimental.

> Testing of the `phpcbf` command is more easily done from a Git clone rather than being installed via PEAR. If you choose to use a Git clone, use the commands `/path/to/PHP_CodeSniffer/scripts/phpcs` and `/path/to/PHP_CodeSniffer/scripts/phpcbf` instead of the ones shown in the documentation below.

PHP_CodeSniffer is able to fix many errors and warnings automatically. The `diff` report can be used to generate a diff that can be applied using the `patch` command. Alternatively, the PHP Code Beautifier and Fixer (`phpcbf`) can be used in place of `phpcs` to automatically generate and apply the diff for you.

Screen-based reports, such as the [full](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Reporting#printing-full-and-summary-reports), [summary](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Reporting#printing-full-and-summary-reports) and [source](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Reporting#printing-a-source-report) reports, provide information about how many errors and warnings are found. If any of the issues can be fixed automatically by `phpcbf`, an additional line of information will be printed:

    $ phpcs /path/to/code/myfile.php
    
    FILE: /path/to/code/myfile.php
    --------------------------------------------------------------------------------
    FOUND 5 ERRORS AFFECTING 4 LINES
    --------------------------------------------------------------------------------
     2 | ERROR | Missing file doc comment
     3 | ERROR | TRUE, FALSE and NULL must be lowercase; expected "false" but found
       |       | "FALSE"
     5 | ERROR | Line indented incorrectly; expected at least 4 spaces, found 1
     8 | ERROR | Missing function doc comment
     8 | ERROR | Opening brace should be on a new line
    --------------------------------------------------------------------------------
    PHPCBF CAN FIX 2 OF THESE SNIFF VIOLATIONS AUTOMATICALLY
    --------------------------------------------------------------------------------

## Printing a Diff Report
> The `diff` report is described here as it is not currently included in the main PHPCS release. It will be moved to the [[Reporting]] page at a late date.

PHP_CodeSniffer can output a diff file that can be applied using the `patch` command. The suggested changes will fix some of the sniff violations that are present in the source code. To print a diff report, use the `--report=diff` command line argument. The output will look like this:

    $ phpcs --report=diff /path/to/code
    
    --- /path/to/code/file.php
    +++ PHP_CodeSniffer
    @@ -1,8 +1,8 @@
     <?php
     
    -if ($foo === FALSE) {
    +if ($foo === false) {
    +    echo 'hi';
         echo 'hi';
    - echo 'hi';
     }
     
     function foo() {

Diff reports are more easily used when output to a file. They can then be applied using the `patch` command:

    $ phpcs --report-diff=/path/to/changes.diff /path/to/code
    $ patch -p0 -ui /path/to/changes.diff
    patching file /path/to/code/file.php

## Using the PHP Code Beautifier and Fixer

To automatically fix as many sniff violations as possible, use the `phpcbf` command in place of the `phpcs` command. When using this command, you do not need to specify a report type. PHPCBF will automatically produce a diff file and apply it to your code:

    $ phpcbf /path/to/code
    Patched 1 files
    Time: 35 ms, Memory: 5.00Mb

The source code is now ready for you to test and commit.

## Viewing Debug Information

To see the fixes that are being made to a file, specify the `-vv` command line argument. This works with both the `phpcs --report=diff` and `phpcbf` commands. There is quite a lot of debug output concerning the standard being used and the tokenizing of the file, but the end of the output will look like this:

    *** START FILE FIXING ***
    Generic_Sniffs_WhiteSpace_ScopeIndentSniff (line 378) replaced token 14 (T_ECHO) "echo" => "····echo"
    Generic_Sniffs_PHP_LowerCaseConstantSniff (line 102) replaced token 9 (T_FALSE) "FALSE" => "false"
    Fixed 2 violations, starting over
    *** END FILE FIXING ***

Sometimes the file may need to be processed multiple times in order to fix all the violations. This can happen when multiple sniffs need to modify the same part of a file, or if a fix causes a new sniff violation somewhere else in the standard. When this happens, the output will look like this:

    *** START FILE FIXING ***
    Generic_Sniffs_WhiteSpace_ScopeIndentSniff (line 378) replaced token 14 (T_ECHO) "echo" => "····echo"
    Squiz_Sniffs_Commenting_ClosingDeclarationCommentSniff (line 115) replaced token 37 (T_CLOSE_CURLY_BRACKET) "}" => "}//end·foo()\n"
    Squiz_Sniffs_WhiteSpace_FunctionSpacingSniff (line 239) replaced token 26 (T_WHITESPACE) "\n" => "\n\n"
    Fixed 3 violations, starting over
    Squiz_Sniffs_WhiteSpace_FunctionSpacingSniff (line 132) replaced token 40 (T_COMMENT) "//end·foo()\n" => "//end·foo()\n\n\n"
    Fixed 1 violations, starting over
    *** END FILE FIXING ***
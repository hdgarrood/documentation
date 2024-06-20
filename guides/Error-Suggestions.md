# Error/warning suggestions

Several of the compiler errors/warnings provide a suggestion of how to fix the issue, not only in the textual
ouput of the compiler, but in the JSON output in a machine-usable format.

### Supported tools

* [purescript-suggest](https://github.com/nwolverson/purescript-suggest) - command line tool which can display or apply all suggestions at once
* [pscid](https://github.com/kRITZCREEK/pscid#suggestions) - apply a single suggestion on a keypress
* [psc-ide-emacs](https://github.com/epost/psc-ide-emacs#insert-suggestion-from-error-c-c-m-s)
* [psc-ide-vim](https://github.com/FrigoEU/psc-ide-vim/blob/master/doc/psc-ide-vim.txt#L61)
* [atom-ide-purescript](https://github.com/nwolverson/atom-ide-purescript#error-suggestions--quick-fix)
* [vscode-ide-purescript](https://github.com/nwolverson/vscode-ide-purescript)
* [purescript-language-server](https://github.com/nwolverson/purescript-language-server)

### Fixable issues

Classes of issues with support for auto-fix suggestions:

* Import warnings (e.g. remove unused imports, make import explicit)
* Missing type annotation (top level type annotations and `_`)

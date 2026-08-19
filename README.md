# tree-sitter-gql

A tree-sitter grammar for zuQL, the GQL dialect the [zu](https://github.com/tamnd/zu)
engine speaks. It is what an editor uses to parse a statement: syntax
highlighting, structural selection, folding, and the rest of what a
tree buys.

    grammar.js                 the grammar
    src/                       the generated parser, committed
    queries/highlights.scm     the highlight query, generated from zu's word list
    test/corpus/               the grammar's own tests
    corpus.mjs                 the grammar against zu's conformance corpus

This is not a second parser of the language. Nothing here decides what
a statement means, and a statement this grammar and the engine disagree
about is a bug here rather than a dialect. What it is for is the answer
a parser that reports errors cannot give: an editor needs a tree for
text that is being typed and is not a statement yet.

## Using it

`src/` is committed, so the grammar builds without the tree-sitter CLI,
which is what an editor package or a Neovim install needs. It is
generated from `grammar.js`, and CI regenerates it and fails if the
committed files move, so the C is derived rather than maintained.

With [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter):

```lua
require("nvim-treesitter.parsers").get_parser_configs().gql = {
  install_info = {
    url = "https://github.com/tamnd/tree-sitter-gql",
    files = { "src/parser.c" },
    branch = "main",
  },
  filetype = "gql",
}
```

The scope is `source.gql` and the file types are `.gql` and `.zuql`.

## Building and testing

    npm install
    npm test

`npm test` regenerates the parser, runs the tests in `test/corpus`, and
then runs `corpus.mjs`.

## What holds this grammar to the engine

zu's conformance corpus, which is a thousand statements the engine
answers and the one body of zuQL written without this grammar in mind.
`corpus.mjs` parses every statement the engine accepts and fails on an
error node in any of them, parses every statement the engine answers
with a syntax error and fails on one that parsed, and runs the
highlight query over all of them so a renamed node fails here rather
than in an editor.

The statements come out of `cargo xtask grammar --queries` in zu, which
reads the corpus with the engine's own reader, so the check needs a
checkout of zu. It looks at `$ZU_ROOT`, then at a sibling directory
called `zu`, and it takes the statements ready-made from
`$ZU_GRAMMAR_QUERIES` when that is set.

Two statements the engine refuses parse here, and both are decisions
written down in `corpus.mjs`. `CAST(1 AS NOPE)` names a type nobody
has, which is a table the binder holds and not a shape. And an empty
file is what an editor opens with.

## The word list

zu keeps one list of every word of zuQL in `grammar/vocabulary.toml`,
grouped by what a word is: a keyword the parser gives meaning to by
position, a literal, a built-in function, a graph algorithm, a value
type name. The shell's colours, the website's TextMate grammar and this
grammar's `queries/highlights.scm` all come from that one list.

So `queries/highlights.scm` is generated, and the generator lives in
zu. CI here checks out zu and runs it against this checkout:

    ZU_TREE_SITTER=<this repo> cargo run -p xtask -- grammar --check

That check runs both ways. Every keyword `grammar.js` spells has to be
a word the list knows, because a word the grammar parses and nothing
colours is a word that comes out plain in an editor. And the committed
highlight query has to be what the list writes now. The reverse is not
required: the list holds the words the parser refuses by name, `MERGE`,
`FILTER`, `LET` and the rest, and those are coloured everywhere and
parsed nowhere.

To regenerate the query after a word moves, run the same command from a
zu checkout without `--check`:

    ZU_TREE_SITTER=<this repo> cargo run -p xtask -- grammar

## History

This grammar lived in zu's `grammar/tree-sitter-gql` until August 2026.
Its history came across with it, so `git log` here goes back to the
commit that added it.

## License

Apache-2.0, the same as zu.

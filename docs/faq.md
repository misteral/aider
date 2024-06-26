
# Frequently asked questions

- [How does aider use git?](#how-does-aider-use-git)
- [Can I use aider with other LLMs, local LLMs, etc?](https://aider.chat/docs/llms.html)
- [Can I run aider in Google Colab?](#can-i-run-aider-in-google-colab)
- [How can I run aider locally from source code?](#how-can-i-run-aider-locally-from-source-code)
- [Can I script aider?](#can-i-script-aider)
- [What code languages does aider support?](#what-code-languages-does-aider-support)
- [How to use pipx to avoid python package conflicts?](#how-to-use-pipx-to-avoid-python-package-conflicts)
- [Aider isn't editing my files?](#aider-isnt-editing-my-files)
- [How can I add ALL the files to the chat?](#how-can-i-add-all-the-files-to-the-chat)
- [Can I specify guidelines or conventions?](#can-i-specify-guidelines-or-conventions)
- [Can I change the system prompts that aider uses?](#can-i-change-the-system-prompts-that-aider-uses)

## How does aider use git?

Aider works best with code that is part of a git repo.
Aider is tightly integrated with git, which makes it easy to:

  - Use git to undo any GPT changes that you don't like
  - Go back in the git history to review the changes GPT made to your code
  - Manage a series of GPT's changes on a git branch

Aider specifically uses git in these ways:

  - It asks to create a git repo if you launch it in a directory without one.
  - Whenever GPT edits a file, aider commits those changes with a descriptive commit message. This makes it easy to undo or review GPT's changes.
  - Aider takes special care if GPT tries to edit files that already have uncommitted changes (dirty files). Aider will first commit any preexisting changes with a descriptive commit message. This keeps your edits separate from GPT's edits, and makes sure you never lose your work if GPT makes an inappropriate change.

Aider also allows you to use in-chat commands to `/diff` or `/undo` the last change made by GPT.
To do more complex management of your git history, you cat use raw `git` commands,
either by using `/git` within the chat, or with standard git tools outside of aider.

While it is not recommended, you can disable aider's use of git in a few ways:

  - `--no-auto-commits` will stop aider from git committing each of GPT's changes.
  - `--no-dirty-commits` will stop aider from committing dirty files before applying GPT's edits.
  - `--no-git` will completely stop aider from using git on your files. You should ensure you are keeping sensible backups of the files you are working with.


## Can I run aider in Google Colab?

User [imabutahersiddik](https://github.com/imabutahersiddik)
has provided this
[Colab notebook](https://colab.research.google.com/drive/1J9XynhrCqekPL5PR6olHP6eE--rnnjS9?usp=sharing).

## How can I run aider locally from source code?

To run the project locally, follow these steps:

```
# Clone the repository:
git clone git@github.com:paul-gauthier/aider.git

# Navigate to the project directory:
cd aider

# Install the dependencies listed in the `requirements.txt` file:
pip install -r requirements.txt

# Run the local version of Aider:
python -m aider.main
```

# Can I script aider?

You can script aider via the command line or python.

## Command line

Aider takes a `--message` argument, where you can give it a natural language instruction.
It will do that one thing, apply the edits to the files and then exit.
So you could do:

```bash
aider --message "make a script that prints hello" hello.js
```

Or you can write simple shell scripts to apply the same instruction to many files:

```bash
for FILE in *.py ; do
    aider --message "add descriptive docstrings to all the functions" $FILE
done
```

User `aider --help` to see all the command line options, but these are useful for scripting:

```
--stream, --no-stream
                      Enable/disable streaming responses (default: True) [env var:
                      AIDER_STREAM]
--message COMMAND, --msg COMMAND, -m COMMAND
                      Specify a single message to send GPT, process reply then exit
                      (disables chat mode) [env var: AIDER_MESSAGE]
--message-file MESSAGE_FILE, -f MESSAGE_FILE
                      Specify a file containing the message to send GPT, process reply,
                      then exit (disables chat mode) [env var: AIDER_MESSAGE_FILE]
--yes                 Always say yes to every confirmation [env var: AIDER_YES]
--auto-commits, --no-auto-commits
                      Enable/disable auto commit of GPT changes (default: True) [env var:
                      AIDER_AUTO_COMMITS]
--dirty-commits, --no-dirty-commits
                      Enable/disable commits when repo is found dirty (default: True) [env
                      var: AIDER_DIRTY_COMMITS]
--dry-run, --no-dry-run
                      Perform a dry run without modifying files (default: False) [env var:
                      AIDER_DRY_RUN]
--commit              Commit all pending changes with a suitable commit message, then exit
                      [env var: AIDER_COMMIT]
```


## Python

You can also script aider from python:

```python
from aider.coders import Coder
from aider.models import Model

# This is a list of files to add to the chat
fnames = ["greeting.py"]

model = Model("gpt-4-turbo", weak_model="gpt-3.5-turbo")

# Create a coder object
coder = Coder.create(main_model=model, fnames=fnames)

# This will execute one instruction on those files and then return
coder.run("make a script that prints hello world")

# Send another instruction
coder.run("make it say goodbye")
```

See the
[Coder.create() and Coder.__init__() methods](https://github.com/paul-gauthier/aider/blob/main/aider/coders/base_coder.py)
for all the supported arguments.

It can also be helpful to set the equivalend of `--yes` by doing this:

```
from aider.io import InputOutput
io = InputOutput(yes=True)
# ...
coder = Coder.create(client=client, fnames=fnames, io=io)
```

## What code languages does aider support?

Aider supports pretty much all the popular coding languages.
This is partly because GPT-4 is fluent in most mainstream languages,
and familiar with popular libraries, packages and frameworks.

In fact, coding with aider is sometimes the most magical
when you're working in a language that you
are less familiar with.
GPT often knows the language better than you,
and can generate all the boilerplate to get to the heart of your
problem.
GPT will often solve your problem in an elegant way
using a library or package that you weren't even aware of.

Aider uses tree-sitter to do code analysis and help
GPT navigate larger code bases by producing
a [repository map](https://aider.chat/docs/repomap.html).

Aider can currently produce repository maps for most mainstream languages, listed below.
But aider should work quite well for other languages, even without repo map support.

- C
- C#
- C++
- Emacs Lisp
- Elixir
- Elm
- Go
- Java
- Javascript
- OCaml
- PHP
- Python
- QL
- Ruby
- Rust
- Typescript

## How to use pipx to avoid python package conflicts?

If you are using aider to work on a python project, sometimes your project will require
specific versions of python packages which conflict with the versions that aider
requires.
If this happens, the `pip install` command may return errors like these:

```
aider-chat 0.23.0 requires somepackage==X.Y.Z, but you have somepackage U.W.V which is incompatible.
```

You can avoid this problem by installing aider using `pipx`,
which will install it globally on your system
within its own python environment.
This way you can use aider to work on any python project,
even if that project has conflicting dependencies.

Install [pipx](https://pipx.pypa.io/stable/) then just do:

```
pipx install aider-chat
```

## Aider isn't editing my files?

Sometimes GPT will reply with some code changes that don't get applied to your local files.
In these cases, aider might say something like "Failed to apply edit to *filename*".

This usually happens because GPT is not specifying the edits
to make in the format that aider expects.
GPT-3.5 is especially prone to disobeying the system prompt instructions in this manner, but it also happens with GPT-4.

Aider makes every effort to get GPT to conform, and works hard to deal with
replies that are "almost" correctly formatted.
If Aider detects an improperly formatted reply, it gives GPT feedback to try again.
Also, before each release new versions of aider are
[benchmarked](https://aider.chat/docs/benchmarks.html).
This helps prevent regressions in the code editing
performance of GPT that could have been inadvertantly
introduced.

But sometimes GPT just won't cooperate.
In these cases, here are some things you might try:

  - Try the older GPT-4 model `gpt-4-0613` not GPT-4 Turbo by running `aider --model gpt-4-0613`.
  - Use `/drop` to remove files from the chat session which aren't needed for the task at hand. This will reduce distractions and may help GPT produce properly formatted edits.
  - Use `/clear` to remove the conversation history, again to help GPT focus.

## How can I add ALL the files to the chat?

People regularly ask about how to add **many or all of their repo's files** to the chat.
This is probably not a good idea and will likely do more harm than good.

The best approach is think about which files need to be changed to accomplish
the task you are working on. Just add those files to the chat.

Usually when people want to add "all the files" it's because they think it
will give GPT helpful context about the overall code base.
Aider will automatically give GPT a bunch of additional context about
the rest of your git repo.
It does this by analyzing your entire codebase in light of the
current chat to build a compact
[repository map](https://aider.chat/2023/10/22/repomap.html).

Adding a bunch of files that are mostly irrelevant to the
task at hand will often distract or confuse GPT.
GPT will give worse coding results, and sometimese even fail to correctly edit files.
Addings extra files will also increase the token costs on your OpenAI invoice.

Again, it's usually best to just add the files to the chat that will need to be modified.
If you still wish to add lots of files to the chat, you can:

- Use a wildcard when you launch aider: `aider src/*.py`
- Use a wildcard with the in-chat `/add` command: `/add src/*.py`
- Give the `/add` command a directory name and it will recurisvely add every file under that dir: `/add src`

## Can I specify guidelines or conventions?

Sometimes you want GPT to be aware of certain coding guidelines,
like whether to provide type hints, which libraries or packages
to prefer, etc.

Just put any extra instructions in a file
like `CONVENTIONS.md` and then add it to the chat.

For more details, see this documentation on
[using a conventions file with aider](https://aider.chat/docs/conventions.html).

## Can I change the system prompts that aider uses?

Aider is set up to support different system prompts and edit formats
in a modular way. If you look in the `aider/coders` subdirectory, you'll
see there's a base coder with base prompts, and then there are
a number of
different specific coder implementations.

If you're thinking about experimenting with system prompts
this document about
[benchmarking GPT-3.5 and GPT-4 on code editing](https://aider.chat/docs/benchmarks.html)
might be useful background.

While it's not well documented how to add new coder subsystems, you may be able
to modify an existing implementation or use it as a template to add another.

To get started, try looking at and modifying these files.

The wholefile coder is currently used by GPT-3.5 by default. You can manually select it with `--edit-format whole`.

- wholefile_coder.py
- wholefile_prompts.py

The editblock coder is currently used by GPT-4 by default. You can manually select it with `--edit-format diff`.

- editblock_coder.py
- editblock_prompts.py

The universal diff coder is currently used by GPT-4 Turbo by default. You can manually select it with `--edit-format udiff`.

- udiff_coder.py
- udiff_prompts.py

When experimenting with coder backends, it helps to run aider with `--verbose --no-pretty` so you can see
all the raw information being sent to/from GPT in the conversation.

You can also refer to the
[instructions for installing a development version of aider](https://aider.chat/docs/install.html#install-development-versions-of-aider-optional).
# charter

Working rules for building software with a model.

## The problem

A model has no memory between sessions, so decisions get argued again and the
reasoning behind them is lost. It will also agree with you when you are wrong,
call things done that are not, and let the records drift away from the code.

## What it is

A set of rules a project works to. They cover how the work is done, not what you
build. You get all of them by running [charter init](#install), which writes them
into your project. This is a handful:

- argue on merit, concede on merit
- one home for every fact
- evidence, or it did not happen
- reproduce a bug before fixing it
- a change ships whole, or it has not shipped
- decisions are recorded with their reasoning, so they are not argued twice
- test what fails silently, and ask a person about what only an eye can judge

## How it works

charter writes the rules into your project as files, and the model reads them at
the start of every session.

```
charter init      set it up
charter update    bring in a newer version of the rules
charter clean     remove it
charter help      every command and its flags
```

`charter init` creates `.charter/`. Commit the whole directory.

- `PART-I.md` is charter's base rules.
- `PART-II.md` is yours, for describing your own project and rules.
- `state.json` is charter's record of what it installed.

It also adds a marked block to `AGENTS.md`, `CLAUDE.md`, `.gitattributes` and
`.prettierignore`, creating any that are not there, plus `repos/.gitignore` if
this is a workspace. Nothing outside those markers is touched, and charter has no
opinion about the rest of your project: it does not write a `.gitignore` for you.

`update` does not touch files you have edited, and lists the ones it skipped.

If you delete something charter installed, run `charter init` again. It writes
back what is missing and leaves everything else alone, so you can run it as often
as you like. A project you set up by hand gets finished rather than refused.

`clean` removes charter. It shows every file it will delete and asks before it
does. It never deletes `PART-II.md`, because you wrote that one.

charter installs the rules, but it cannot enforce them.

It works the same whether you are writing Go, Rust, or PHP.

## Two layouts

Every project is one of two. charter is told which, and you can change your mind
later.

### flat

One codebase. The default, and what most projects are.

```sh
charter init
```

The rules apply to that codebase and there is nothing else to decide.

### workspace

Several codebases, one set of rules. When a product is more than one repository
(a web app, an API, some infrastructure) you can keep one brief and one history
across all of them.

Make a directory to hold them, run charter in it, and clone each codebase into a
directory underneath.

The root carries the rules and your records, and no product code. The codebases
stay separate repositories with their own histories, and nothing is moved.

```
your-product/
  .charter/        the rules, and the records you write
  AGENTS.md
  repos/
    .gitignore     charter's. Keeps repos/ on a fresh clone
    web/           a clone, still its own repository
    api/           a clone, still its own repository
```

```sh
charter init --workspace
```

That makes `repos/` and puts a `.gitignore` inside it, marked like every other
file charter writes into. Your own `.gitignore` is not touched. Then clone your
codebases in.

### Changing your mind

```sh
charter init --workspace     # this project holds sibling repositories
charter init --flat          # it is one codebase again
```

Going flat takes charter's block back out of `repos/.gitignore`, and removes the
file if you never added anything of your own. Your clones and the `repos/`
directory are left where they are. charter does not move your code.

Without that file, `repos/` stops appearing in a fresh clone. Run
`charter init --workspace` if you want it back. `charter clean` takes it too:
charter removes what charter can put back.

Passing both flags at once is refused. Passing neither leaves the layout alone,
so `charter init` on a workspace writes back whatever is missing without changing
what the project is.

## Install

### Linux

```sh
curl -L -o charter https://github.com/Script47/charter/releases/latest/download/charter-linux-amd64
chmod +x charter
sudo mv charter /usr/local/bin/charter
charter version
```

### Windows

```powershell
$dir = "$env:LOCALAPPDATA\Programs\charter"
New-Item -ItemType Directory -Force $dir | Out-Null
Invoke-WebRequest https://github.com/Script47/charter/releases/latest/download/charter-windows-amd64.exe -OutFile "$dir\charter.exe"
Unblock-File "$dir\charter.exe"

$path = [Environment]::GetEnvironmentVariable('Path', 'User')
if ($path -notlike "*$dir*") {
  [Environment]::SetEnvironmentVariable('Path', "$path;$dir", 'User')
}
```

Open a new terminal, then run `charter version`.

## Updating

```sh
charter update --self     # the binary
charter update            # the files charter put in this project
charter update --all      # every project charter has set up on this machine
```

`charter version` shows where the binary and this project stand:

```
binary  v0.0.49-dev
project v0.0.44-dev
root    /home/you/your-product
layout  flat

v0.0.49-dev is available. Run charter update --self
https://github.com/Script47/charter/releases/tag/v0.0.49-dev
```

`charter update` lists every file:

```
charter v0.0.44-dev -> v0.0.49-dev
root    /home/you/your-product

  updated    .charter/PART-I.md
  yours      .charter/PART-II.md
  modified   .gitattributes
             charter wrote 8b3bc17c1095, it is now 823b293ddee4
             undo the edit and run update again to take charter's version
  unchanged  AGENTS.md
  unchanged  CLAUDE.md
  unchanged  .prettierignore

What changed between v0.0.44-dev and v0.0.49-dev:
  https://github.com/Script47/charter/releases
```

`modified` means you edited that file. charter skips it and updates the rest.
`yours` means charter wrote it once and will not write to it again.

When a version moves and nothing in your project changes, charter says so rather
than leaving you to work it out from a list of files that did not move. What
changed was in the binary.

Notes:

- Installed in `/usr/local/bin`? `sudo charter update --self`.
- Windows leaves a `charter.exe.old` behind. The next charter command removes it.
- Update the binary before the project. charter will not move a project onto an
  older version than it is on.
- Changes are applied in place, so `git checkout -- <file>` is the undo.
- The update notice goes to stderr, and does not appear at all if charter cannot
  reach the internet.
- To avoid `--self` entirely, re-run the download from [Install](#install).

## Several projects

charter keeps a list of the projects it has set up on this machine, in
`~/.charter/charter.json`. One command updates all of them:

```sh
charter update --all
```

It shows one plan naming every project and asks once, because this writes into
repositories other than the one you are standing in. A project that has moved or
been deleted is skipped with a reason on its own line, and the run ends with a
total across all of them, so one file charter would not overwrite is visible
without reading ten reports.

Projects you set up before charter kept that list are not on it yet. Name them
once and they are added as they are updated:

```sh
charter update ~/work/api ~/work/web
```

`charter clean` takes a project back off the list. That file holds those paths
and a random ID for this machine, and nothing else. Nothing in it is sent
anywhere, and it is safe to delete: charter puts a project back the next time you
run it there.

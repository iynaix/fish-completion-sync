# fish-completion-sync

A fish plugin to help dynamically load fish completions from `$XDG_DATA_DIRS`.

## Installation

### home-manager

```nix
programs.fish = {
  # ...
  plugins = [
    #...
    {
      pkgs.fetchFromGitHub {
        owner = "pfgray";
        repo = "fish-completion-sync";
        rev = "...";
        sha256 = "...";
      }
    }
  ];
};
```

## Why

Fish only considers `$XDG_DATA_DIRS` on shell startup, and thus will not dynamically load completions if `$XDG_DATA_DIRS` changes sometime after the shell starts. This leads to problems when using tools like [direnv](https://github.com/direnv/direnv), which try to add completions on a per-folder basis.

This plugin tries to alleviate this issue.

tl;dr; it's the bit of glue between:
https://github.com/fish-shell/fish-shell/issues/8261
and:
https://github.com/direnv/direnv/issues/443

## How it works

Fish _will_ search `$fish_complete_path` dynamically, so the idea is to implement a function which listens for changes to `$XDG_DATA_DIRS`, and attempts to keep that in sync with `$fish_complete_path`.

```
function fish_completion_sync --on-variable XDG_DATA_DIRS
   
  # If there are paths in $FISH_COMPLETION_ADDITIONS,
  # but not in $XDG_DATA_DIRS
  #   remove them from $fish_complete_path
  #   remove them from $FISH_COMPLETION_ADDITIONS

  # if there are paths in $XDG_DATA_DIRS
  # but not in $FISH_COMPLETION_ADDITIONS
  #   add them to $fish_complete_path
  #   add them to $FISH_COMPLETION_ADDITIONS

  echo "got new data dirs: $XDG_DATA_DIRS"
end
```

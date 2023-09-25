# Zed

[Zed](zed.dev) is a fast and lightweight code editor.

The command palette can be accessed by typing `Cmd + Shift + P`.

## Configuration

Zed configuration is stored at `~/.config/zed/settings.json`, accessible via `Cmd + ,`, `Zed > Preferences > Open Settings`, or `open settings` in the command palette.

Zed default configuration can be accessed via `Zed > Preferences > Open Default Settings`, or `open default settings` in the command palette.

Zed provides by default a language server that is used to provide code intelligence. To configure the language server:
```json
"lsp": {
  "rust-analyzer": {
    "initialization_options": {
      "checkOnSave": {
        "command": "clippy" // rust-analyzer.checkOnSave.command
      }
    }
  }
}
```

Zed enables by default code formatting on save. In order to perform the formatting using the current language server:
```json
"formatter": "language-server"
```

or to use an external command:
```json
"formatter": {
  "external": {
    "command": "sed",
    "arguments": ["-e", "s/ *$//"]
  }
}
```

To configure specific settings for different languages:
```json
"language_overrides": {
  "C": {
    "format_on_save": "off",
    "preferred_line_length": 64,
    "soft_wrap": "preferred_line_length"
  },
  "JSON": {
    "tab_size": 4
  }
}
```

## Key bindings

Key bindings are configured in `~/.config/zed/keymap.json`, accessible via `Cmd + K, Cmd + S`, `Zed > Preferences > Open Key Bindings`, or `open keymap` in the command palette.

An example of custom key binding:
```json
[
  {
    "context": "Workspace",
    "bindings": {
      "cmd-alt-shift-left": "workspace::ActivatePreviousPane",
      "cmd-alt-shift-right": "workspace::ActivateNextPane"
    }
  }
]
```

Useful default key bindings:
- Command palette: `Cmd + Shift + P`
- Find files: `Cmd + P`
- Gloabl search: `Cmd + Shift + F`
- Activate previous or next pane: `Cmd + K, Cmd + Left|Right`
- Activate previous or next tab: `Cmd + Alt + Left|Right`
- Add selection above or below: `Cmd + Alt + Up|Down`
- Format: `Cmd + Shift + I`
- Show character palette: `Cmd + Ctrl + Space`
- Show completions: `Ctrl + Space`

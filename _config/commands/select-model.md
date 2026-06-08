Switch the Claude Code model by updating the config file.

The user wants to select a model. Available models:
- `claude-sonnet-4-6` (fast, efficient)
- `claude-opus-4-7` (most capable, slower)

The user passed this argument: $ARGUMENTS

Based on the argument:
- If argument contains "opus" → set model to `claude-opus-4-7`
- If argument contains "sonnet" → set model to `claude-sonnet-4-6`
- If no argument or unclear → ask the user which model they want

To switch the model, read the files `/Users/mac/.claude/settings.json` and `/Users/mac/Library/Application Support/Claude/claude_desktop_config.json`, update the `"model"` field in both files to the chosen model ID, and write them back. Then inform the user that the change will take effect after restarting Claude Code or opening a new terminal session.

# stims
My Claude Plugin Marketplace intended to accelerate dev efficiency -- AI stimulants for developers.

## Installation

Add this marketplace to your Claude Code:

```bash
/plugin marketplace add bradlet/stims
```

## Available Plugins

### review-plugin

TODO: REMOVE THIS PLUGIN (generated with claude for boilerplate)

Adds a `/review` command for quick code reviews that checks for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

**Install:**
```bash
/plugin install review-plugin@my-plugins
```

**Usage:**
```bash
/review
```

## Adding More Plugins

To add new plugins to this marketplace:

1. Create a new plugin directory under `plugins/`:
   ```bash
   mkdir -p plugins/your-plugin/.claude-plugin
   mkdir -p plugins/your-plugin/commands
   ```

2. Create the plugin manifest at `plugins/your-plugin/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "your-plugin",
     "description": "Description of your plugin",
     "version": "1.0.0"
   }
   ```

3. Add commands, agents, or other components under `plugins/your-plugin/`

4. Register the plugin in `.claude-plugin/marketplace.json` by adding to the `plugins` array:
   ```json
   {
     "name": "your-plugin",
     "source": "./plugins/your-plugin",
     "description": "Description of your plugin",
     "version": "1.0.0"
   }
   ```

5. Commit and push your changes

## Local Testing

Test the marketplace locally before pushing:

```bash
# Add local marketplace
/plugin marketplace add .

# Install a plugin
/plugin install review-plugin@my-plugins

# Test the plugin
/review
```

## Documentation

For more information about creating Claude Code plugins and marketplaces, visit:
- [Plugins Documentation](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces Documentation](https://code.claude.com/docs/en/plugin-marketplaces)

## TODO
- [ ] Add a plugin which emulates a small feature dev team: skill / command for high level feature spec creation; subagents for an architect, hot shot IC and skeptical senior engineer. Skill / command should build the spec and pass to the architect to formalize constraints based on architectural principles for the given project. Skeptical senior should give pointed advice to the IC on approaching implementation. IC should implement a first pass then send back to senior for review. Architect should have final word on whether the implementation aligns with desired architectural principles. Repeat IC loop if not.

# stims
My Claude Plugin Marketplace intended to accelerate dev efficiency -- AI stimulants for developers.

## Installation

Add this marketplace to your Claude Code:

```bash
/plugin marketplace add bradlet/stims
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

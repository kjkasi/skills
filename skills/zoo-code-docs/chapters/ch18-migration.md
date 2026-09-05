# Chapter 18: Migration Guide

## Core Idea
Migrating from Roo Code to Zoo Code is a two-step process: export your settings from Roo Code, then import them into Zoo Code. The process preserves your configuration but requires careful handling of sensitive data.

## Key Concepts
- **Settings Export**: Roo Code produces a JSON file containing API keys, custom modes, custom instructions, auto-approval settings, and other configuration. This file may contain sensitive data.
- **Settings Import**: Zoo Code reads the exported Roo Code file and merges settings into its current configuration, preserving existing Zoo Code-specific settings.
- **OAuth Token Exclusion**: OAuth tokens (used by providers like ChatGPT Plus/Pro and Kimi Code) are stored in VS Code SecretStorage and are NOT included in the export—these must be re-authenticated in Zoo Code.
- **Sensitive Data Handling**: The exported file can include API keys. Store it carefully and never share it publicly.
- **Settings Merge**: Zoo Code merges imported settings rather than replacing them, so existing Zoo Code configuration is preserved alongside imported Roo Code settings.

## Mental Models
- **Export-import pipeline**: Roo Code Export → JSON file → Zoo Code Import → merged configuration. The JSON file is the only artifact in this pipeline.
- **Security boundary**: The export file crosses a security boundary (Roo Code → Zoo Code) and may contain secrets—treat it like a credentials file.
- **Partial migration**: Not everything transfers—OAuth tokens and some Roo Code-specific features may require manual re-configuration.
- **Incremental adoption**: Import first, then customize. The merge approach lets you adopt Zoo Code features gradually without losing your Roo Code workflow.

## Anti-patterns
- **Sharing the exported settings file**: The file may contain API keys—never commit it to version control or share it in public channels.
- **Assuming full feature parity**: Some Roo Code features may not have exact Zoo Code equivalents; review imported settings after migration.
- **Not re-authenticating OAuth providers**: Providers using OAuth (ChatGPT Plus/Pro, Kimi Code) require fresh sign-in in Zoo Code since tokens don't transfer.
- **Importing without backing up**: Always keep a copy of the exported file before importing, in case you need to roll back or re-import.

## Code Examples
```
# Step 1: Export from Roo Code
1. Open Roo Code settings
2. Click "Export"
3. Choose save location and save the file
# File contains: API keys, custom modes, instructions, auto-approval settings

# Step 2: Import into Zoo Code
1. Open Zoo Code
2. Click "Import Settings"
3. Select the Roo Code exported file
# Settings are merged into Zoo Code configuration

# Post-import checklist:
# - Verify API keys are correct (re-enter if needed)
# - Re-authenticate OAuth providers (ChatGPT Plus/Pro, Kimi Code, Qwen Code)
# - Review custom modes for compatibility
# - Test auto-approval settings
# - Check custom instructions are applied correctly
```

## Worked Example
**Scenario**: You're a Roo Code user migrating to Zoo Code for its additional provider support.

1. In Roo Code: Open Settings → Click Export → Save the JSON file to a secure location.
2. Verify the file contains your expected settings (open in a text editor to review—be careful with API keys).
3. In Zoo Code: Open Settings → Click Import Settings → Select the exported file.
4. Zoo Code merges the settings—your custom modes, instructions, and API keys are now available.
5. Re-authenticate any OAuth-based providers (ChatGPT Plus/Pro, Kimi Code) since tokens don't transfer.
6. Test your workflow to ensure all settings are applied correctly.
7. Delete or securely store the exported file—it contains sensitive data.

## Key Takeaways
1. Migration is two steps: Export from Roo Code, Import into Zoo Code—settings are merged, not replaced.
2. The export file may contain API keys; store it securely and never share it.
3. OAuth tokens don't transfer—re-authenticate providers like ChatGPT Plus/Pro and Kimi Code after import.
4. Review imported settings post-migration to ensure compatibility and correct configuration.

## Connects To
- **Ch 15**: After migration, you may want to configure additional providers not available in Roo Code (e.g., AWS Bedrock, GCP Vertex AI).
- **Ch 16**: Common post-migration issues relate to API key validation and OAuth re-authentication—check the troubleshooting guide.
- **Ch 17**: Import your Roo Code custom modes, then enhance them with Zoo Code-specific features like sticky models and file restrictions.

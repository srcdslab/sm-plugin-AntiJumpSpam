# Copilot Instructions for SourceMod Plugin Development

## Repository Overview
This repository contains **AntiJumpSpam**, a SourceMod plugin that prevents players from spamming jump inputs to avoid knockback in crawl spaces. The plugin is specifically designed for Zombie Reloaded servers running on Source engine games (CS:S/CS:GO).

## Technical Environment
- **Language**: SourcePawn (.sp files)
- **Platform**: SourceMod 1.11.0+ (currently targeting 1.11.0-git6934)
- **Build System**: SourceKnight 0.2 (configured in `sourceknight.yaml`)
- **Compiler**: SourcePawn compiler (spcomp) via SourceKnight
- **CI/CD**: GitHub Actions using `maxime1907/action-sourceknight@v1`

## Project Structure
```
addons/sourcemod/
├── scripting/
│   └── AntiJumpSpam.sp        # Main plugin source code
└── plugins/                   # Compiled output (generated)

.github/
├── workflows/
│   └── ci.yml                 # Automated build and release
└── dependabot.yml             # Dependency updates

sourceknight.yaml              # Build configuration
```

## Code Style & Standards

### Core Requirements
- **Always use**: `#pragma semicolon 1` and `#pragma newdecls required`
- **Indentation**: 4 spaces (tabs converted to spaces)
- **Variables**: camelCase for locals, PascalCase for functions, prefix globals with `g_`
- **No trailing spaces**
- **Descriptive naming**: Variable and function names should be self-documenting

### SourcePawn-Specific Conventions
```sourcepawn
// Good examples from codebase:
ConVar g_cvarJumpsUntilBlock = null;           // Global ConVar with g_ prefix
float g_fLastJumpTime[MAXPLAYERS + 1];        // Global array with type prefix
int g_iFastJumpCount[MAXPLAYERS + 1];         // Global array with type prefix

public void OnPluginStart()                   // PascalCase for public functions
{
    // Function implementation
}

static bool bHoldingJump[MAXPLAYERS + 1];     // Static with type prefix
```

## Build & Development Workflow

### Building the Plugin
```bash
# The repository uses SourceKnight for building
# CI/CD automatically builds on push/PR using GitHub Actions
# Manual build would require SourceKnight setup locally
```

### Key Build Files
- `sourceknight.yaml`: Defines build configuration, dependencies, and output
- `.github/workflows/ci.yml`: Automated CI/CD pipeline
- Build outputs go to `/addons/sourcemod/plugins/`

### Testing Approach
- **No automated unit tests**: SourceMod plugins are typically tested on live servers
- **Manual testing required**: Deploy to test server and verify functionality
- **Key test scenarios**:
  - Rapid jump spam detection
  - Cooldown period functionality
  - ZombieReloaded integration
  - ConVar configuration changes

## Plugin-Specific Architecture

### Core Functionality
The AntiJumpSpam plugin implements anti-spam protection through:
- **ConVar system**: Configurable jump limits and intervals
- **OnPlayerRunCmd hook**: Real-time button input monitoring
- **Client arrays**: Per-player state tracking
- **ZombieReloaded integration**: Library detection and team filtering

### Key Components
1. **Configuration (ConVars)**:
   - `sm_ajs_jumpsuntilblock`: Jump threshold before blocking
   - `sm_ajs_jumpinterval`: Minimum time between jumps
   - `sm_ajs_cooldowninterval`: Reset time for spam counter

2. **State Management**:
   - `g_fLastJumpTime[]`: Tracks last jump timestamp per client
   - `g_iFastJumpCount[]`: Counts rapid jumps per client
   - `bHoldingJump[]`: Prevents multiple detections per jump

3. **Integration Points**:
   - ZombieReloaded library detection
   - CS:GO team filtering (Terrorists only)
   - Engine time-based calculations

## Performance Considerations

### Critical Performance Rules
- **OnPlayerRunCmd runs every tick**: Minimize operations in this hook
- **Avoid string operations**: In frequently called functions
- **Cache expensive calculations**: Don't recalculate ConVar values every tick
- **Memory management**: Plugin uses arrays, no dynamic allocation concerns

### Current Optimizations
- Static arrays for per-client data (O(1) access)
- Minimal calculations in OnPlayerRunCmd
- Early returns for non-applicable clients
- ConVar caching through direct access

## Development Best Practices

### When Making Changes
1. **Understand game context**: This plugin affects player movement mechanics
2. **Consider server performance**: Changes must not impact tick rate
3. **Test with ZombieReloaded**: Integration is core to functionality
4. **Validate ConVar ranges**: Ensure sensible limits for server operators

### Memory Management
- **No dynamic allocation**: Plugin uses static arrays
- **Client disconnect cleanup**: Reset arrays in OnClientDisconnect
- **No handles to close**: Plugin doesn't create handles requiring cleanup

### Error Handling
- **Validate client indices**: Always check IsClientInGame
- **Team/alive checks**: Verify game state before processing
- **Library existence**: Check g_bZRLoaded before applying logic

## Version Control & Releases

### Versioning
- **Current version**: 1.2 (in plugin info)
- **Semantic versioning**: MAJOR.MINOR.PATCH for releases
- **Auto-tagging**: CI creates "latest" tag on main branch pushes

### Release Process
- **Automated**: GitHub Actions handles build and release
- **Artifacts**: Compiled .smx files packaged in tar.gz
- **Testing**: Manual server testing required before version bumps

## Common Modification Patterns

### Adding New ConVars
```sourcepawn
// In OnPluginStart():
ConVar g_cvarNewSetting = CreateConVar("sm_ajs_newsetting", "defaultvalue", "Description");
// Don't forget AutoExecConfig(true); at the end
```

### Modifying Jump Logic
- Changes should be in OnPlayerRunCmd
- Consider impact on server tick rate
- Maintain backward compatibility with existing ConVars
- Test edge cases (rapid disconnect/reconnect, etc.)

### Library Integration
- Follow existing ZombieReloaded pattern
- Use OnLibraryAdded/OnLibraryRemoved for dynamic detection
- Graceful degradation when optional libraries unavailable

## Documentation Requirements
- **Plugin header**: Keep plugin info block current (name, author, description, version)
- **ConVar descriptions**: Clear, concise explanations for server operators
- **Code comments**: Focus on complex logic sections only
- **No excessive headers**: Avoid unnecessary file/function headers per guidelines
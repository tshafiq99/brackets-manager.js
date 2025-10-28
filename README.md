# brackets-manager.js

[![npm](https://img.shields.io/npm/v/brackets-manager.svg)](https://www.npmjs.com/package/brackets-manager)
[![Downloads](https://img.shields.io/npm/dt/brackets-manager.svg)](https://www.npmjs.com/package/brackets-manager)
[![Package Quality](https://packagequality.com/shield/brackets-manager.svg)](https://packagequality.com/#?package=brackets-manager)

A TypeScript library to manage tournament brackets (round-robin, single elimination, double elimination). Part of the Brackets ecosystem for tournament management and visualization.

## Brackets Ecosystem

The Brackets libraries work together to provide a complete tournament solution:

- **brackets-manager.js** (this library): Manages tournament structures, matches, and results
- **brackets-viewer.js**: Displays tournament brackets and visualizations  
- **brackets-model**: Contains shared TypeScript types used by both libraries

These libraries can be used independently or together, depending on your project needs.

## What It Does

`brackets-manager.js` provides a complete solution for managing tournament brackets with all the logic needed to:
- Create and manage different tournament types
- Update match results and standings
- Track tournament progression
- Handle complex scenarios like BYEs, forfeits, and multiple stages
- Work with any storage backend

This library contains **only the business logic**—it doesn't include a GUI or storage implementation. You can use [brackets-viewer.js](https://github.com/Drarig29/brackets-viewer.js) to display brackets.

## Features

- **Multiple Tournament Types**: Single elimination, double elimination, and round-robin
- **BYE Support**: Automatic handling of byes during creation for seeding and balancing
- **Forfeit Support**: Handle forfeits during match updates
- **Match States**: Locked, waiting, ready, running, completed, archived
- **Multi-Stage Tournaments**: Combine multiple stages (e.g., round-robin → elimination)
- **Best-Of Series**: Support for Bo1, Bo3, Bo5 matches with game tracking
- **Flexible Storage**: Works with any storage backend (JSON, SQL, Redis, in-memory)
- **Seeding Methods**: Natural, reverse, inner-outer, and more
- **Consolation Final**: Optional consolation matches
- **Grand Final Variants**: Simple and double grand finals

## Installation

```bash
npm install brackets-manager
```

## Storage Interface

Before using `brackets-manager.js`, you need to implement a storage solution. The library uses a `CrudInterface` that abstracts data persistence, allowing you to work with any backend.

### Available Storage Implementations

The [brackets-storage](https://github.com/drarig29/brackets-storage) repository provides several ready-to-use implementations:

- **JSON Database**: `brackets-json-db` - Stores data in JSON files
- **In-Memory Database**: `brackets-memory-db` - Stores data in memory (for testing)
- **SQL Database**: `brackets-sql` - Works with SQL databases
- **Redis Database**: `brackets-redis` - Uses Redis for storage

### Custom Storage Implementation

If you need a custom storage solution, implement the `CrudInterface`:

```js
class MyStorage implements CrudInterface {
  async select(table, filter) {
    // Return array of records matching the filter
  }
  
  async insert(table, data) {
    // Insert data and return the inserted record with ID
  }
  
  async update(table, id, data) {
    // Update record by ID and return the updated record
  }
  
  async delete(table, id) {
    // Delete record by ID
  }
}
```

The storage interface handles these tables:
- `tournament`: Tournament metadata
- `stage`: Tournament stages (round-robin, elimination, etc.)
- `group`: Groups within stages
- `round`: Rounds within groups
- `match`: Individual matches
- `match_game`: Sub-matches for best-of series
- `participant`: Teams/players

## Quick Start

### Basic Setup

```js
const { JsonDatabase } = require('brackets-json-db');
const { BracketsManager } = require('brackets-manager');

const storage = new JsonDatabase();
const manager = new BracketsManager(storage);
```

### Creating Different Tournament Types

#### Single Elimination Tournament

```js
await manager.create.stage({
  tournamentId: 0,
  name: 'Single Elimination',
  type: 'single_elimination',
  seeding: ['Team 1', 'Team 2', 'Team 3', 'Team 4', 'Team 5', 'Team 6', 'Team 7', 'Team 8'],
  settings: {
    consolationFinal: true, // Include 3rd place match
  },
});
```

#### Double Elimination Tournament

```js
await manager.create.stage({
  tournamentId: 0,
  name: 'Double Elimination',
  type: 'double_elimination',
  seeding: ['Team 1', 'Team 2', 'Team 3', 'Team 4'],
  settings: {
    grandFinal: 'double', // Double grand final
  },
});
```

#### Round Robin Tournament

```js
await manager.create.stage({
  tournamentId: 0,
  name: 'Round Robin Groups',
  type: 'round_robin',
  seeding: ['Team 1', 'Team 2', 'Team 3', 'Team 4', 'Team 5', 'Team 6'],
  settings: {
    groupCount: 2, // Split into 2 groups
    roundRobinMode: 'simple', // Each team plays others once
  },
});
```

### Managing Matches

#### Update Match Results

```js
// Basic match update
await manager.update.match({
  id: matchId,
  opponent1: { score: 16, result: 'win' },
  opponent2: { score: 12 },
});

// Best-of series (Bo3)
await manager.update.match({
  id: matchId,
  opponent1: { score: 2, result: 'win' },
  opponent2: { score: 1 },
});

// Handle forfeits
await manager.update.match({
  id: matchId,
  opponent1: { result: 'win' },
  opponent2: { result: 'forfeit' },
});
```

#### Find Next Matches

```js
// Find matches ready to be played
const nextMatches = await manager.find.nextMatches(matchId);

// Find next match for a specific participant
const participantNextMatch = await manager.find.nextMatches(matchId, participantId);
```

### Retrieving Data

```js
// Get tournament data
const tournamentData = await manager.get.tournamentData(0);

// Get stage data
const stageData = await manager.get.stageData(stageId);

// Get final standings
const standings = await manager.get.finalStandings(stageId);

// Get current seeding
const seeding = await manager.get.seeding(stageId);

// Get match games for best-of series
const matchGames = await manager.get.matchGames(matches);
```

## Visualization with brackets-viewer.js

To display tournament brackets, use `brackets-viewer.js` alongside the manager:

### HTML Setup

```html
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/brackets-viewer@latest/dist/brackets-viewer.min.css" />
</head>
<body>
    <div id="brackets-viewer"></div>
    <script src="https://cdn.jsdelivr.net/npm/brackets-viewer@latest/dist/brackets-viewer.min.js"></script>
</body>
</html>
```

### JavaScript Integration

```js
// Get tournament data from manager
const tournamentData = await manager.get.tournamentData(0);
const stageData = await manager.get.stageData(stageId);

// Render the bracket
window.bracketsViewer.render({
    stages: stageData.stage,
    matches: stageData.match,
    matchGames: stageData.match_game,
    participants: stageData.participant,
}, {
    selector: '#brackets-viewer',
    participantOriginPlacement: 'before',
    separatedChildCountLabel: true,
    showSlotsOrigin: true,
    showLowerBracketSlotsOrigin: true,
});
```

### Real-time Updates

```js
// Update match result
await manager.update.match({
    id: matchId,
    opponent1: { score: 16, result: 'win' },
    opponent2: { score: 12 },
});

// Refresh the viewer with updated data
const updatedData = await manager.get.stageData(stageId);
window.bracketsViewer.render({
    stages: updatedData.stage,
    matches: updatedData.match,
    matchGames: updatedData.match_game,
    participants: updatedData.participant,
});
```

For more examples, see the [documentation](https://drarig29.github.io/brackets-docs/getting-started/).

## How It Works

### Architecture

```
┌─────────────────────────────────────────┐
│   Your Application                       │
│   (Node.js, Express, etc.)              │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│   BracketsManager                       │
│   - create.stage()                      │
│   - update.match()                      │
│   - get.finalStandings()                │
│   - find.nextMatches()                  │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│   Storage Interface (Abstract)          │
│   - select(table, filter)               │
│   - insert(table, data)                 │
│   - update(table, id, data)             │
│   - delete(table, id)                    │
└───────────────┬─────────────────────────┘
                │
                ▼
        ┌───────┴────────┐
        │                │
        ▼                ▼
  JSON Database    SQL Database
  In-Memory DB     Redis
  Custom Storage   etc.
```

### Key Concepts

1. **Storage Interface**: Abstract interface that can be implemented for any backend
2. **Stage**: A complete tournament bracket (e.g., "Elimination Round")
3. **Group**: Subdivision within a stage (e.g., groups in round-robin)
4. **Round**: Set of matches within a group
5. **Match**: Individual game between two participants
6. **Match Game**: Sub-match for Best-Of series

### Data Flow

1. **Creation**: Define participants → Create stage → Generate matches
2. **Updates**: Change match results → Propagate to next matches
3. **Progression**: System automatically determines ready/locked states
4. **Completion**: Get standings, reset matches, etc.

### Module Structure

The manager is organized into logical modules:

- **`create`**: Stage, participant creation
- **`get`**: Retrieve data (standings, seeding, matches)
- **`update`**: Modify matches, scores, seeding
- **`reset`**: Clear results, reset matches
- **`delete`**: Remove stages, tournaments
- **`find`**: Locate previous/next matches
- **`storage`**: Direct storage access (last resort)

## Project Structure

```
brackets-manager.js/
├── src/
│   ├── base/           # Core functionality
│   │   ├── getter.ts   # Data retrieval helpers
│   │   ├── updater.ts  # Update logic
│   │   └── stage/
│   │       └── creator.ts  # Stage creation
│   ├── create.ts       # Creation module
│   ├── update.ts       # Update module
│   ├── get.ts          # Get module
│   ├── delete.ts       # Delete module
│   ├── find.ts         # Find module
│   ├── reset.ts         # Reset module
│   ├── ordering.ts     # Seeding algorithms
│   ├── helpers.ts       # Utility functions
│   ├── manager.ts      # Main manager class
│   ├── types.ts        # TypeScript definitions
│   └── index.ts        # Public exports
├── test/               # Comprehensive test suite
│   ├── custom.spec.js
│   ├── delete.spec.js
│   ├── double-elimination.spec.js
│   ├── find.spec.js
│   ├── get.spec.js
│   ├── round-robin.spec.js
│   ├── single-elimination.spec.js
│   ├── update.spec.js
│   └── unit/
├── dist/               # Compiled JavaScript
└── package.json        # NPM configuration
```

## Development Setup

### Prerequisites

- Node.js 14+
- npm 7+

### Clone and Install

```bash
git clone https://github.com/Drarig29/brackets-manager.js.git
cd brackets-manager.js
npm install
```

### Development Scripts

```bash
# Run tests
npm test

# Run tests with coverage
npm run coverage

# Lint code
npm run lint

# Build TypeScript
npm run build

# Watch mode for development
npm start

# Watch mode for database (for testing)
npm run db
```

### Building

The project uses TypeScript and compiles to the `dist/` directory:

```bash
npm run build
```

This compiles all TypeScript files to JavaScript with source maps.

### Testing

Tests use Mocha and Chai with comprehensive coverage of all features:

```bash
# Run all tests
npm test

# Run specific test file
npm test -- test/single-elimination.spec.js

# Run with coverage
npm run coverage
```

Test files cover:
- Tournament creation (all types)
- Match updates and progression
- BYE handling
- Forfeit scenarios
- Seeding algorithms
- Best-of series
- Complex bracket navigation

## API Overview

### Core Methods

```js
// Creation
manager.create.stage(options)

// Retrieval
manager.get.stageData(stageId)
manager.get.seeding(stageId)
manager.get.finalStandings(stageId)
manager.get.matchGames(matches)

// Updates
manager.update.match(matchData)
manager.update.seeding(stageId, newSeeding)
manager.update.confirmSeeding(stageId)

// Finding
manager.find.match(matchId)
manager.find.nextMatches(matchId, participantId?)
manager.find.previousMatches(matchId, participantId?)

// Resetting
manager.reset.matchResults(matchId)
manager.reset.seeding(stageId)

// Deletion
manager.delete.stage(stageId)
manager.delete.tournament(tournamentId)
```

### Storage Implementation

Implement the storage interface to work with your backend:

```js
class MyStorage implements Storage {
  async select(table, filter) { /* ... */ }
  async insert(table, data) { /* ... */ }
  async update(table, id, data) { /* ... */ }
  async delete(table, id) { /* ... */ }
}
```

## Tournament Types

Understanding tournament structure is crucial for effective bracket management:

### Single Elimination
- **Structure**: Standard knockout tournament with one bracket
- **Flow**: Teams compete until one winner remains
- **Features**: 
  - Supports consolation final for 3rd place
  - Automatic BYE handling for odd numbers
  - Simple progression logic

### Double Elimination  
- **Structure**: Two brackets - winner bracket and loser bracket
- **Flow**: Teams get a second chance after first loss
- **Features**:
  - Winner bracket: Standard elimination
  - Loser bracket: Second-chance elimination  
  - Grand final(s) between bracket winners
  - Supports both simple and double grand finals

### Round Robin
- **Structure**: Multiple groups where every team plays every other team
- **Flow**: All teams play each other once (or twice)
- **Features**:
  - Supports multiple groups for large tournaments
  - Various seeding methods (natural, reverse, inner-outer)
  - Flexible standings calculation
  - Can be combined with elimination stages

### Tournament Structure Hierarchy

```
Tournament
├── Stage (e.g., "Group Stage", "Elimination Round")
│   ├── Group (e.g., "Group A", "Winner Bracket")
│   │   ├── Round (e.g., "Round 1", "Quarterfinals")
│   │   │   ├── Match (e.g., "Team 1 vs Team 2")
│   │   │   │   ├── Match Game (for Bo3, Bo5 series)
│   │   │   │   └── Match Game
│   │   │   └── Match
│   │   └── Round
│   └── Group
└── Stage
```

## Glossary

Understanding key terminology is essential for working with tournament brackets:

### Core Concepts

- **Tournament**: The overall competition containing one or more stages
- **Stage**: A distinct phase of a tournament (e.g., "Group Stage", "Elimination Round")
- **Group**: A logical subdivision within a stage (e.g., "Group A", "Winner Bracket")
- **Round**: A set of matches played simultaneously within a group
- **Match**: A competition between two participants
- **Match Game**: Individual games within a best-of series (Bo3, Bo5, etc.)
- **Participant**: A team or individual competing in the tournament

### Tournament Flow

- **Seeding**: The initial arrangement of participants in the bracket
- **BYE**: Automatic advancement when there's an odd number of participants
- **Forfeit**: When a participant cannot or chooses not to play
- **Progression**: How participants advance through the tournament
- **Standings**: Current ranking of participants based on results

### Match States

- **Locked**: Match cannot be played yet (prerequisites not met)
- **Waiting**: Match is ready but not yet started
- **Ready**: Match can be played
- **Running**: Match is currently in progress
- **Completed**: Match has finished
- **Archived**: Match results are finalized

### Bracket Types

- **Winner Bracket**: Main elimination bracket (double elimination)
- **Loser Bracket**: Second-chance bracket (double elimination)
- **Grand Final**: Final match between bracket winners
- **Consolation Final**: Match for 3rd place (single elimination)

## Documentation

### Official Documentation

- **[Complete Documentation](https://drarig29.github.io/brackets-docs/)** - Main documentation site
- **[Getting Started Guide](https://drarig29.github.io/brackets-docs/getting-started/)** - Step-by-step setup
- **[User Guide](https://drarig29.github.io/brackets-docs/user-guide/)** - Comprehensive usage guide

### API Reference

- **[BracketsManager API](https://drarig29.github.io/brackets-docs/reference/manager/)** - Complete API documentation
- **[Storage Interface](https://drarig29.github.io/brackets-docs/user-guide/storage/)** - Storage implementation guide
- **[Helpers Documentation](https://drarig29.github.io/brackets-docs/reference/manager/modules/helpers.html)** - Utility functions

### Related Libraries

- **[brackets-viewer.js](https://github.com/Drarig29/brackets-viewer.js)** - Tournament visualization
- **[brackets-model](https://github.com/Drarig29/brackets-model)** - Shared TypeScript types
- **[brackets-storage](https://github.com/drarig29/brackets-storage)** - Storage implementations

### Demos and Examples

- **[Angular Demo](https://drarig29.github.io/brackets-docs/demos/angular/)** - Angular integration example
- **[React Demo](https://drarig29.github.io/brackets-docs/demos/react/)** - React integration example
- **[Toornament API Demo](https://drarig29.github.io/brackets-docs/demos/toornament/)** - External API integration

## FAQ

### Common Questions

**Q: What is `tournamentId` and why is it always 0?**
A: The `tournamentId` is used to group related stages together. For simple tournaments, you can use 0. For complex tournaments with multiple stages, use different IDs to organize them.

**Q: How do I handle odd numbers of participants?**
A: The library automatically handles BYEs (automatic advancement) when there's an odd number of participants. No manual intervention needed.

**Q: Can I combine different tournament types?**
A: Yes! You can create multiple stages with different types (e.g., round-robin group stage followed by single elimination playoffs).

**Q: How do I implement custom storage?**
A: Implement the `CrudInterface` with `select`, `insert`, `update`, and `delete` methods. See the Storage Interface section above.

**Q: Can I use this with React/Vue/Angular?**
A: Yes! The library is framework-agnostic. Use `brackets-viewer.js` for visualization in any frontend framework.

### Troubleshooting

- **Matches not progressing**: Check that match results are properly set with `result: 'win'` or `result: 'loss'`
- **Storage errors**: Ensure your storage implementation returns data in the expected format
- **Viewer not rendering**: Verify that all required data (stages, matches, participants) is provided to the render function

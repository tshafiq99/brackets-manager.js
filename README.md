# brackets-manager.js

[![npm](https://img.shields.io/npm/v/brackets-manager.svg)](https://www.npmjs.com/package/brackets-manager)
[![Downloads](https://img.shields.io/npm/dt/brackets-manager.svg)](https://www.npmjs.com/package/brackets-manager)
[![Package Quality](https://packagequality.com/shield/brackets-manager.svg)](https://packagequality.com/#?package=brackets-manager)

A TypeScript library to manage tournament brackets (round-robin, single elimination, double elimination).

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

## Quick Start

```js
const { JsonDatabase } = require('brackets-json-db');
const { BracketsManager } = require('brackets-manager');

const storage = new JsonDatabase();
const manager = new BracketsManager(storage);

// Create a tournament stage
await manager.create.stage({
  tournamentId: 0,
  name: 'Elimination Stage',
  type: 'double_elimination',
  seeding: ['Team 1', 'Team 2', 'Team 3', 'Team 4'],
  settings: { grandFinal: 'double' },
});

// Update match results
await manager.update.match({
  id: 0,
  opponent1: { score: 16, result: 'win' },
  opponent2: { score: 12 },
});

// Get final standings
const standings = await manager.get.finalStandings(0);
console.log(standings);
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

### Single Elimination
- Standard knockout tournament
- Teams compete until one winner remains
- Supports consolation final

### Double Elimination
- Loser bracket for second-chance
- Winner bracket and loser bracket
- Grand final(s) between bracket winners
- Supports both simple and double grand finals

### Round Robin
- Every team plays every other team
- Supports multiple groups
- Various seeding methods
- Flexible standings calculation

## Documentation

- [Complete API Reference](https://drarig29.github.io/brackets-docs/reference/manager/classes/BracketsManager.html)
- [Getting Started Guide](https://drarig29.github.io/brackets-docs/getting-started/)
- [Storage Guide](https://drarig29.github.io/brackets-docs/user-guide/storage/)
- [Helpers Documentation](https://drarig29.github.io/brackets-docs/reference/manager/modules/helpers.html)

## Contributing

Contributions are welcome! Please ensure:
- All tests pass (`npm test`)
- Code is linted (`npm run lint`)
- New features include tests
- TypeScript types are properly defined

## Credits

This library was created to be used by [Nantarena](https://nantarena.net/) and was inspired by:

- [Toornament](https://www.toornament.com/en_US/) (configuration, API and data format)
- [Challonge's bracket generator](https://challonge.com/tournaments/bracket_generator)
- [jQuery Bracket](http://www.aropupu.fi/bracket/) (feature examples)

## License

ISC

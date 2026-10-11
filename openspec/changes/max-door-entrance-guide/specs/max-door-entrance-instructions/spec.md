## ADDED Requirements

### Requirement: Project-specific entrance exercise
The guide SHALL identify `MaxSpawn`, `LevelStart`, `Walkable`, `MaxBody`, and `Max` by their saved names and SHALL explain each object's role in the Intro entrance.

#### Scenario: Learner prepares the scene
- **WHEN** a learner opens the guide before editing Intro
- **THEN** the guide tells them where to find the door marker, destination, navigation mesh, Character body, and visible child

### Requirement: Navigation and control handoff instructions
The guide SHALL instruct the learner to request navmesh route points, move `MaxBody` through Character physics, prevent the existing A/D component from writing movement during entry, and restore A/D control on arrival.

#### Scenario: Learner follows the entrance steps
- **WHEN** the learner builds the level-specific entrance sequence
- **THEN** the guide gives an ordered route and control handoff procedure without supplying a completed game implementation

### Requirement: Linked character ownership
The guide SHALL explain that source game properties and shared `MaxMovement` settings belong in `Max.blend`, while Intro-specific routing belongs in `Intro.blend`. It SHALL state that Blender blocks editing source game properties on Intro's linked override.

#### Scenario: Learner sees an override edit error
- **WHEN** Blender refuses a game-property edit on Intro's `MaxBody`
- **THEN** the guide directs the learner to the editable source file and runtime state change

### Requirement: Visible animation and verification
The guide SHALL distinguish physical travel from the visible walk cycle and SHALL include checks for door entry, control handoff, failed paths, canyon respawn, and Escape.

#### Scenario: Learner reviews completion
- **WHEN** the learner tests the exercise in the game
- **THEN** the guide provides observable outcomes for the entrance and for existing gameplay behavior

### Requirement: Documentation-only delivery
This change SHALL provide instructions and acceptance criteria without implementing the entrance behavior or adding game code.

#### Scenario: Change is reviewed before the exercise
- **WHEN** this OpenSpec change is committed
- **THEN** no runtime entrance component, Logic Node change, or animation Action is introduced by this change

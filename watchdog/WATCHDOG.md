# Watchdog System (Hlídací pes) Documentation

## Overview

The Watchdog system is a queue management system that automatically handles program signups when users are waitlisted. It ensures that users are moved from waiting lists to active signups based on their preferences and queue positions.

**NEW: Parent-Child Timeslot Support** - The system now supports shorter timeslots that can occur within longer ones, with automatic conflict detection between overlapping programs.

## How It Works

### Core Concepts

1. **PŘIHLÁŠEN (Signed Up)**: User is within program capacity and has preference 0
2. **HLÍDACÍ PES (Watchdog)**: User is outside program capacity or has preference > 0
3. **Parent Timeslots**: Long timeslots (e.g., 1:00-2:00) with `parent_id = null`
4. **Child Timeslots**: Shorter timeslots (e.g., 1:00-1:30, 1:30-2:00) with `parent_id` pointing to parent

### Rules

1. **One Active Program Per Timeslot**: Each user can have maximum 1 signed up program per timeslot
2. **Multiple Watchdog Programs**: Users can have multiple programs in watchdog within the same timeslot
3. **Priority System**: Lower preference number = higher priority (0 = signed up, 1+ = watchdog priority)
4. **Parent-Child Conflicts**: Users cannot be signed up for both parent and child programs simultaneously
5. **Maximum 2 Children**: Each parent timeslot can have at most 2 child timeslots
6. **No Nested Children**: Child timeslots cannot have their own children (max 1 level of nesting)

### Automatic Processing

When capacity is freed up (user unsigns or capacity is increased):

1. **Find First in Queue**: Get the first user in waiting list (ordered by signup time)
2. **Check Conflicts**: Look for overlapping programs the user is signed up for (including parent-child relationships)
3. **Priority Comparison**: Compare preferences to determine if user should be moved
4. **Unsign Lower Priority**: Remove user from currently signed program if new one has higher priority
5. **Remove Lower Priority Watchdogs**: Remove programs from watchdog with lower priority than the new signup
6. **Update Preferences**: Set new program preference to 0 (signed up)
7. **Send Notification**: Email user about the change
8. **Cascade Processing**: Process watchdog for programs the user was removed from

### Parent-Child Timeslot Behavior

- **Conflict Detection**: Programs in parent and child timeslots are considered conflicting
- **Cross-Timeslot Watchdog**: Users can be moved between parent and child program queues
- **Example**: User in watchdog for 1:00-1:30 program can be moved to 1:00-2:00 program when it becomes available
- **Priority Respect**: System respects user's preference ordering across parent-child relationships

## API Endpoints

### Program Signup
```
POST /events/{eventId}/programy/{programId}/signup
```
- Signs user up for a program
- Automatically handles watchdog preferences
- Detects conflicts with parent-child timeslot relationships
- Triggers watchdog processing when needed

### Program Unsign
```
DELETE /events/{eventId}/programy/{programId}/signup
```
- Removes user from program
- Triggers watchdog processing to move waiting users

### Manage Watchdog Preferences
```
GET /events/{eventId}/programy/preferences
PUT /events/{eventId}/programy/preferences
```
- View and update user's watchdog priorities
- Allows users to reorder their watchdog preferences

### Update Program Capacity (Admin)
```
PUT /events/{eventId}/programy/{programId}
```
- When capacity is increased, automatically processes watchdog

### Timeslot Management (Admin)
```
POST /events/{eventId}/timeslots
```
- Create new timeslots with optional parent_id for child timeslots
- Validates parent-child relationships and prevents overlaps
- Enforces maximum 2 children per parent

```
GET /events/{eventId}/timeslots
```
- Returns all timeslots including parent_id information
- Orders parent timeslots first, then by start time

## Database Schema

### program_signup Table
```sql
user_id INT
program_id INT
preference INT DEFAULT 0  -- 0 = signed up, 1+ = watchdog priority
created_at DATETIME       -- Queue order for waiting list
```

### timeslots Table
```sql
id INT PRIMARY KEY
event_id INT
start DATETIME
end DATETIME  
parent_id INT NULL        -- NULL for parent timeslots, references parent for children
```

### Key Relationships
- Programs belong to timeslots
- Timeslots have start/end times and optional parent relationships
- Child timeslots reference their parent via parent_id
- Users can have multiple signups per timeslot with different preferences
- Conflict detection includes parent-child timeslot relationships

## Example Scenarios

### Scenario 1: Basic Watchdog
```
Initial State:
- Program A (capacity: 1)
- User 1: PŘIHLÁŠEN na Program A
- User 2: HLÍDACÍ PES na Program A (preference: 0, position: 1 in queue)

Action: User 1 unsigns from Program A

Result:
- User 2: PŘIHLÁŠEN na Program A (preference becomes 0)
- User 2 receives email notification
```

### Scenario 2: Priority-Based Movement
```
Initial State:
- Program A (capacity: 1), Program B (capacity: 1)
- User 1: PŘIHLÁŠEN na Program B (preference: 0)
- User 1: HLÍDACÍ PES na Program A (preference: 1, higher priority)
- User 2: PŘIHLÁŠEN na Program A

Action: User 2 unsigns from Program A

Result:
- User 1: PŘIHLÁŠEN na Program A (preference becomes 0)
- User 1: removed from Program B
- Program B processes its own watchdog
- User 1 receives email notification
```

### Scenario 3: Multiple Watchdog Programs
```
Initial State:
- Program A, B, C in same timeslot
- User 1: PŘIHLÁŠEN na Program C (preference: 0)
- User 1: HLÍDACÍ PES na Program A (preference: 1, highest priority)
- User 1: HLÍDACÍ PES na Program B (preference: 2, lower priority)

Action: Program A becomes available

Result:
- User 1: PŘIHLÁŠEN na Program A (preference becomes 0)
- User 1: removed from Program B and Program C
- Programs B and C process their own watchdogs
- User 1 receives email notification listing removed programs
```

### Scenario 4: Parent-Child Timeslot Conflicts
```
Initial State:
- Parent Timeslot: 1:00-2:00 (Program X)
- Child Timeslot: 1:00-1:30 (Program Y)  
- User 1: PŘIHLÁŠEN na Program X (preference: 0)

Action: User 1 tries to sign up for Program Y

Result:
- User 1: HLÍDACÍ PES na Program Y (preference: 1, cannot sign up due to conflict)
- System detects parent-child relationship and prevents double booking
```

### Scenario 5: Cross-Timeslot Watchdog Movement
```
Initial State:
- Parent Timeslot: 1:00-2:00 (Program Long, capacity: 1, User A signed up)
- Child Timeslot: 1:00-1:30 (Program Short, capacity: 1, User B signed up)
- User C: HLÍDACÍ PES na Program Short (preference: 1)

Action: User A unsigns from Program Long

Result:
- User C: PŘIHLÁŠEN na Program Long (moved from child watchdog to parent program)
- Program Short processes its own watchdog
- User C receives email notification
```

### Scenario 6: Priority-Based Movement Across Parent-Child Timeslots
```
Initial State:
- Parent Timeslot: 1:00-2:00 (Long Workshop, capacity: 1)
- Child Timeslot A: 1:00-1:30 (Short Talk A, capacity: 1, User X signed up)
- Child Timeslot B: 1:30-2:00 (Short Talk B, capacity: 1, User Y signed up)
- User Z: PŘIHLÁŠEN na Long Workshop (preference: 0)
- User Z: HLÍDACÍ PES na Short Talk A (preference: 1, highest priority)
- User Z: HLÍDACÍ PES na Short Talk B (preference: 2, lowest priority)

Action: User X unsigns from Short Talk A (highest priority program becomes available)

Result:
- User Z: PŘIHLÁŠEN na Short Talk A (preference becomes 0)
- User Z: removed from Long Workshop (triggers watchdog processing for Long Workshop)
- User Z: remains HLÍDACÍ PES na Short Talk B (preference: 2, no space available)
- User Z receives email notification about the change
- Long Workshop processes its own watchdog to fill the freed space
```

## Email Notifications

When watchdog moves a user, they receive an email containing:
- New program they were signed up for
- List of programs they were removed from
- Link to view their current signups

## Technical Implementation

### Core Functions

#### `processWatchdog(programId: number)`
- Main watchdog processing function
- Called when capacity is freed up
- Handles user movement and notifications

#### `updateWatchdogPreferences(userId, eventId, timeslotId, excludeProgramId?)`
- Calculates correct preference number for new signups
- Ensures preferences are consecutive and valid

### Key Features
- **Recursive Processing**: When a user is moved, watchdog is triggered for programs they left
- **Transaction Safety**: Database operations are atomic
- **Email Integration**: Automatic notifications with template
- **Preference Validation**: Ensures valid preference ordering
- **Queue Ordering**: FIFO for users with same preference level
- **Parent-Child Support**: Handles conflicts between overlapping timeslots
- **Cross-Timeslot Movement**: Users can be moved between parent and child program queues

### Parent-Child Implementation Details

#### Conflict Detection Logic
The system detects conflicts between programs in related timeslots using:
```sql
-- Programs in same timeslot OR parent-child relationships
WHERE programs.timeslot_id = target_timeslot_id
   OR programs.timeslot_id = target_timeslot.parent_id  -- parent conflict
   OR programs.timeslots.parent_id = target_timeslot_id -- child conflict
```

#### Watchdog Processing
When processing the watchdog queue:
1. **Find First in Queue**: Gets earliest signup with preference > 0
2. **Check Parent-Child Conflicts**: Looks for programs in both:
   - Same timeslot as the available program
   - Parent timeslot (if available program is in child timeslot)  
   - Child timeslots (if available program is in parent timeslot)
3. **Priority Resolution**: Moves user if new program has higher priority than conflicts
4. **Cascade Processing**: Triggers watchdog for any programs user was removed from

#### API Validation
- **Timeslot Creation**: Validates parent exists, prevents deep nesting, enforces 2-child limit
- **Program Signup**: Detects parent-child conflicts before allowing signup/watchdog
- **Preference Calculation**: Considers overlapping programs across parent-child relationships

## Testing

Run the test suite:
```bash
cd web/backend
bun test tests/watchdog.test.ts
```

Tests cover:
- Basic queue movement
- Priority-based switching
- Preference calculation
- Edge cases and error handling

## Error Handling

The system gracefully handles:
- Invalid program IDs
- User permission checks
- Database constraint violations
- Email delivery failures (logged, doesn't block processing)
- Concurrent access scenarios

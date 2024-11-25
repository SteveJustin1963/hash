# hash
# MINT Version 2.0 Hashing Functions

## Overview

This document describes two hashing functions for MINT Version 2.0:

1. Function `H`: Hashes values on the stack
2. Function `S`: Hashes values in an array

Key corrections from the original implementation:
- Removed multiple-character labels
- Simplified stack operations
- Fixed byte array handling
- Added proper variable initialization
- Ensured compatibility with MINT 2.0 syntax rules

## Function H: Hash Stack Values

**Purpose:** Compute a hash of n values from the stack
**Input:** n values followed by count n
**Output:** Single hash value

### Implementation:

```mint
:H              // Hash stack values
0 h !           // Initialize hash in variable h
(               // Loop n times
  " h + h !     // Add next value to hash
  h { h !       // Left shift hash
  h ^ h !       // XOR with original hash
)               // End loop
h .             // Print final hash
;
```

### Usage Example:
```mint
10 20 30 3 H    // Hash three values: 10, 20, 30
```

## Function S: Hash Array Values

**Purpose:** Compute hash of byte array
**Input:** Array address and length
**Output:** Single hash value

### Implementation:

```mint
:S              // Hash array values
p ! n !         // Store array ptr and length
0 h !           // Initialize hash
n (             // Loop length times
  p /i ? " h +  // Get array value, duplicate
  h { +         // Shift and add to hash
  h ^ h !       // XOR with current hash
)               // End loop
h .             // Print final hash
;
```

### Usage Example:
```mint
[ 1 2 3 ] a !   // Create and store array
a 3 S           // Hash array of 3 elements
```

 
##   Notes

1. **Variables**
   - h: Stores current hash value
   - p: Array pointer (for S function)
   - n: Length counter (for S function)

2. **Limitations**
   - Maximum array size: 32767 (signed 16-bit)
   - Hash values limited to 16-bit range
   - Single-character variable names only

3. **Best Practices**
   - Initialize variables before use
   - Clear variables after use
   - Use comments for clarity
   - Remove comments in final code

## Error Handling

The functions assume valid inputs. In production code, add these checks:
- Verify n > 0 before looping
- Check array bounds
- Validate input values

## Example Program

Complete example combining both functions:

```mint
// Define hash functions
:H 0 h ! ( " h + h ! h { h ! h ^ h ! ) h . ;
:S p ! n ! 0 h ! n ( p /i ? " h + h { + h ^ h ! ) h . ;

// Test with stack values
10 20 30 3 H    // Hash stack values

// Test with array
[ 1 2 3 ] a !   // Create test array
a 3 S           // Hash array
```

## Performance Considerations

1. **Speed**
   - Minimal stack operations
   - Efficient bit manipulation
   - Optimized loops

2. **Memory**
   - Uses only required variables
   - No temporary storage
   - Minimal stack usage

## Testing

Test both functions with:
1. Single values
2. Multiple values
3. Zero values
4. Maximum values (#FFFF)
5. Arrays of different sizes

Remember to clear variables between tests:
```mint
0 h ! 0 p ! 0 n !
```

/////////////

Another simple hash table implementation in MINT2 that can store and retrieve key-value pairs. Since MINT2 has limited memory and only works with 16-bit integers, we'll create a basic hash table that stores numbers.



```mint
// Hash table implementation
// Uses array a for the hash table (size 16)
// Uses array b for tracking if slot is occupied
// Uses x for key, y for value, h for hash
// Uses n for table size (16)

:H                     // Hash function
  16 % h!             // Simple modulo hash, store in h
;                     // Note: input value should be on stack

:P                     // Put key-value pair
  y! x!               // Store key in x, value in y
  x H                 // Hash the key
  h b ? /T = (        // If slot is occupied
    h 1+ 16 % h!      // Linear probing
    h b ? /T = (      // If next slot occupied
      h 1+ 16 % h!    // Try next slot
    )
  )
  x h a ?!            // Store key at hash position
  y h 16 + a ?!       // Store value 16 positions after
  /T h b ?!           // Mark slot as occupied
;

:G                     // Get value for key
  x!                  // Store key in x
  x H                 // Hash the key
  h b ? /T = (        // If slot is occupied
    h a ? x = (       // If key matches
      h 16 + a ? .    // Print value
    ) /E (            // Else try next slot
      h 1+ 16 % h!    // Linear probing
      h b ? /T = (    // If next slot occupied
        h a ? x = (   // If key matches
          h 16 + a ? . // Print value
        )
      )
    )
  )
;

:I                     // Initialize hash table
  [ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0           // Keys array
    0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] a!      // Values array
  [ /F /F /F /F /F /F /F /F                    // Occupancy array
    /F /F /F /F /F /F /F /F ] b!              // Initialize all slots as empty
;

// Example usage:
// I              // Initialize hash table
// 123 456 P      // Store key 123 with value 456
// 123 G          // Retrieve value for key 123 (prints 456)

```

This hash table implementation has the following features:

1. Fixed size of 16 entries for simplicity
2. Uses simple modulo hashing
3. Handles collisions using linear probing
4. Supports basic put and get operations
5. Uses three main functions:
   - H: Hash function
   - P: Put key-value pair
   - G: Get value by key
   - I: Initialize hash table

Here's how to use it:

1. First initialize the hash table:
```
I
```

2. Store some key-value pairs:
```
123 456 P    // Store 456 at key 123
789 101 P    // Store 101 at key 789
```

3. Retrieve values:
```
123 G        // Will print 456
789 G        // Will print 101
```

Limitations:
1. Only works with 16-bit integers
2. Fixed size of 16 entries
3. No delete operation (can be added if needed)
4. Simple collision handling with linear probing
5. No handling for table full condition

Options:
1. Add a delete operation?
2. Implement better collision handling?
3. Add table full detection?
4. Add any other specific features?

/////////////////

## GPIO pin state manager
An enhance hash table to work as a GPIO pin state manager, which would be very useful for a small single-board computer. This will let us store and manage pin states, pin modes, and handle pin toggling with easy lookups.



```mint
// GPIO Pin Manager using Hash Table
// Uses array a for pin states and modes
// Uses array b for pin occupation tracking
// Format: Lower 8 bits for state, upper 8 bits for mode
// Modes: 0=INPUT, 1=OUTPUT, 2=INPUT_PULLUP
// States: 0=LOW, 1=HIGH
// Port address for GPIO output is at #80

:H                     // Hash function for pin numbers
  8 % h!              // 8 pins maximum
;

:M                     // Set pin mode (pin mode --)
  m! p!               // Store pin in p, mode in m
  p H                 // Hash the pin number
  m 8 { s!           // Shift mode to upper byte
  0 v!               // Clear current value
  s v | h a ?!       // Combine mode and value, store
  /T h b ?!          // Mark pin as configured
  p #80 /O           // Output to GPIO port
;

:W                     // Write pin state (pin state --)
  s! p!               // Store pin in p, state in s
  p H                 // Hash the pin number
  h a ? #FF00 &      // Get current mode (upper byte)
  s |                // Combine with new state
  h a ?!             // Store back in hash table
  p #80 /O           // Output to GPIO port
;

:R                     // Read pin state (pin -- state)
  p!                  // Store pin in p
  p H                 // Hash the pin number
  h a ? #FF &        // Get current state (lower byte)
  .                  // Print state
;

:T                     // Toggle pin (pin --)
  p!                  // Store pin in p
  p H                 // Hash the pin number
  h a ? " #FF &      // Get current state
  1 ^ t!             // Toggle state
  h a ? #FF00 &      // Get current mode
  t |                // Combine mode and new state
  h a ?!             // Store back
  p #80 /O           // Output to GPIO port
;

:I                     // Initialize GPIO manager
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
;

:D                     // Debug: Display all pin states
  `Pin States:` /N
  8 (                 // For all 8 pins
    `Pin ` /i .       // Print pin number
    `: Mode=` 
    /i H a ? #FF00 & 8 } .  // Print mode
    ` State=`
    /i H a ? #FF & . /N     // Print state
  )
;

// Example usage:
// I              // Initialize GPIO manager
// 3 1 M          // Set pin 3 as OUTPUT
// 3 1 W          // Set pin 3 HIGH
// 3 R            // Read pin 3 state
// 3 T            // Toggle pin 3
// D              // Display all pin states

```

This enhanced version provides a GPIO pin manager with the following features:

1. Manages up to 8 GPIO pins (0-7)
2. Supports 3 pin modes:
   - 0 = INPUT
   - 1 = OUTPUT
   - 2 = INPUT_PULLUP

3. Key Functions:
   - M: Set pin mode (pin mode --)
   - W: Write pin state (pin state --)
   - R: Read pin state (pin -- state)
   - T: Toggle pin (pin --)
   - D: Debug display of all pin states
   - I: Initialize GPIO manager

4. Each pin entry stores:
   - Lower 8 bits: Current state (0/1)
   - Upper 8 bits: Pin mode (0/1/2)

Example usage:

```mint
I               // Initialize GPIO manager

// Set up pins
2 1 M           // Set pin 2 as OUTPUT
3 2 M           // Set pin 3 as INPUT_PULLUP
4 0 M           // Set pin 4 as INPUT

// Control pins
2 1 W           // Set pin 2 HIGH
2 R             // Read pin 2 state
2 T             // Toggle pin 2
3 R             // Read input from pin 3

// Debug
D               // Display all pin configurations
```

Key features that make this useful for an SBC:

1. Fast access to pin states using hash table lookups
2. Compact memory usage (only two 8-element arrays)
3. Built-in pin mode management
4. Direct GPIO port output (#80)
5. Pin toggling without needing to read state first
6. Debug function to view all pin configurations
7. Handles both input and output pins
8. Maintains pin state history in memory

Potential enhancements could be:

1. Input pin polling/monitoring
2. Creating pin-specific callbacks
3. PWM output support
4. Implementing PWM measurement
5. Pin interrupt handling
6. Pin grouping for parallel output
7. Custom pin modes (like analog input)
8. Adding noise filtering
9. Pin state change detection
10. Pin timing functions
11. Adding time stamping
12. Adding edge-triggered interrupts
13. Implementing pattern detection
14. 


 ///////////\\\\\\\\\\\\\\\\\/////////////

## pin polling/monitoring
that can detect changes in input pins and track their history. This is useful for debouncing, edge detection, and monitoring state changes.

```mint
// GPIO Input Pin Monitor and State Change Detector
// Extends previous GPIO manager with polling features
// Uses array c for state history (last 4 states per pin)
// Uses array d for change detection flags
// Uses array e for debounce counters
// Poll interval set by variable t (default 100ms)

:I                     // Initialize GPIO manager and monitor
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] c!    // State history array
  [ 0 0 0 0 0 0 0 0 ] d!    // Change detection array
  [ 0 0 0 0 0 0 0 0 ] e!    // Debounce counters
  100 t!                     // Default poll interval 100ms
;

:S                     // Shift state history for a pin
  p! s!                // s=state, p=pin
  p H h!               // Get hash
  h c ? 4 { #0F &     // Shift old states left
  s |                  // Add new state
  h c ?!               // Store updated history
;

:C                     // Check for state change on pin
  p!                   // Store pin number
  p H h!              // Get hash
  h c ? #0F &         // Get last 4 states
  " #0F &             // Duplicate and mask
  = /F =              // Compare, invert (true if changed)
  h d ?!              // Store change flag
;

:B                     // Check if input is bouncing
  p!                   // Store pin number
  p H h!              // Get hash
  h c ? #0F &         // Get last 4 states
  #5555 &             // Check alternating pattern
  0 >                 // True if bouncing detected
;

:P                     // Poll single input pin
  p!                   // Store pin number
  p H h!              // Get hash
  h a ? #FF00 & 8 }   // Get pin mode
  0 = (                // If INPUT mode
    p #80 /I          // Read from GPIO port
    #01 & s!          // Mask to get state
    s p S             // Store in history
    p C               // Check for change
    p B (             // If bouncing
      h e ? 1 + h e ?! // Increment bounce counter
    ) /E (
      0 h e ?!        // Reset bounce counter
    )
  )
;

:M                     // Monitor all input pins
  /U (                 // Infinite loop
    8 (               // For each pin
      /i P            // Poll pin i
    )
    t @ /N            // Wait interval t
  )
;

:Z                     // Interrupt handler for pin changes
  /v #80 /I           // Read GPIO port
  8 (                 // For each pin
    /i H h!           // Get pin hash
    h a ? #FF00 & 8 } // Get pin mode
    0 = (             // If INPUT mode
      /i P            // Poll the pin
      h d ? /T = (    // If state changed
        `Pin ` /i .   // Print pin number
        ` changed to `
        h c ? #01 & . // Print new state
        /N
      )
    )
  )
;

:D                     // Debug: Display pin states and history
  `Pin Monitor Status:` /N
  8 (                 // For each pin
    /i H h!           // Get hash for current pin
    h a ? #FF00 & 8 } // Get pin mode
    0 = (             // If INPUT mode
      `Pin ` /i .     // Print pin number
      ` History: `
      h c ? #0F &     // Get history
      4 (             // For each bit in history
        " 2 } #01 &   // Shift and mask each bit
        .             // Print bit
        ` `
      ) 
      h e ? 0 > (     // If bouncing
        `(Bouncing) `
      )
      h d ? /T = (    // If changed
        `(Changed)`
      )
      /N
    )
  )
;

// Example usage:
// I              // Initialize
// 2 0 M          // Set pin 2 as INPUT
// 3 0 M          // Set pin 3 as INPUT
// M              // Start monitoring (runs forever)
// Or for manual polling:
// 2 P            // Poll pin 2
// D              // Show debug info

```

This enhanced version adds comprehensive input pin monitoring with these features:

1. State History
   - Tracks last 4 states for each pin
   - Useful for debouncing and edge detection
   - Stored in array c

2. Change Detection
   - Detects and flags state changes
   - Stored in array d
   - Accessible via pin status display

3. Debounce Detection
   - Detects bouncing inputs
   - Counts bounce events
   - Stored in array e

4. Key Functions:
   - S: Shift state history for a pin
   - C: Check for state changes
   - B: Check for bouncing
   - P: Poll single input pin
   - M: Monitor all input pins continuously
   - Z: Interrupt handler for pin changes
   - D: Display detailed pin status

Example Usage:

```mint
I               // Initialize everything

// Set up input pins
2 0 M           // Set pin 2 as INPUT
3 0 M           // Set pin 3 as INPUT

// Option 1: Manual polling
2 P             // Poll pin 2
D               // Show status

// Option 2: Continuous monitoring
M               // Start monitoring all pins
```

The monitor provides:

1. Debouncing through history tracking
2. Edge detection (state changes)
3. Bounce detection and counting
4. Interrupt support
5. Detailed status display
6. Configurable polling interval

The status display shows:
- Pin number
- Last 4 states (0/1)
- Bounce indication
- Change indication

//////////////////

## GPIO monitor with pin-specific callbacks
allows for custom functions to be executed when pin states change. Since MINT2 uses uppercase letters A-Z for functions, we'll create a callback management system that maps pins to these function letters.

```mint
// GPIO Pin Callback Manager
// Extends GPIO monitor with callback functionality
// Uses array f for callback function mappings
// Uses array g for callback conditions (0=any, 1=rising, 2=falling)
// Uses array h for callback enable flags
// Examples of callbacks are defined in A-Y (Z reserved for interrupts)

:I                     // Initialize GPIO manager and callbacks
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] c!    // State history array
  [ 0 0 0 0 0 0 0 0 ] d!    // Change detection array
  [ 0 0 0 0 0 0 0 0 ] e!    // Debounce counters
  [ 0 0 0 0 0 0 0 0 ] f!    // Callback function mappings
  [ 0 0 0 0 0 0 0 0 ] g!    // Callback conditions
  [ /F /F /F /F /F /F /F /F ] h!  // Callback enable flags 
  100 t!                     // Default poll interval 100ms
;

:X                     // Set callback for pin (pin func condition --)
  c! f! p!            // Store pin, function, condition
  p H h!              // Get hash
  f h f ?!            // Store function mapping
  c h g ?!            // Store condition
  /T h h ?!           // Enable callback
;

:Y                     // Disable callback for pin (pin --)
  p!                  // Store pin
  p H h!              // Get hash
  /F h h ?!           // Disable callback
;

:E                     // Execute callback if conditions met
  p!                  // Store pin number
  p H h!              // Get hash
  h h ? /T = (        // If callback enabled
    h c ? #01 &       // Get current state
    h c ? #02 & 1 }   // Get previous state
    h g ? 1 = (       // Rising edge condition
      " < (           // If rising edge
        h f ? /G      // Execute callback
      )
    )
    h g ? 2 = (       // Falling edge condition
      " > (           // If falling edge
        h f ? /G      // Execute callback
      )
    )
    h g ? 0 = (       // Any change condition
      = /F = (        // If changed
        h f ? /G      // Execute callback
      )
    )
  )
;

:P                     // Enhanced poll with callback
  p!                  // Store pin number
  p H h!              // Get hash
  h a ? #FF00 & 8 }   // Get pin mode
  0 = (               // If INPUT mode
    p #80 /I         // Read from GPIO port
    #01 & s!         // Mask to get state
    s p S            // Store in history
    p C              // Check for change
    p B (            // If bouncing
      h e ? 1 + h e ?! // Increment bounce counter
    ) /E (
      0 h e ?!       // Reset bounce counter
      p E            // Execute callback if needed
    )
  )
;

:M                     // Monitor all pins with callbacks
  /U (                // Infinite loop
    8 (              // For each pin
      /i P           // Poll pin i
    )
    t @ /N           // Wait interval t
  )
;

// Example callback functions
:A                     // Example: Print pin went HIGH
  `Pin ` p . ` went HIGH` /N
;

:B                     // Example: Print pin went LOW
  `Pin ` p . ` went LOW` /N
;

:C                     // Example: Toggle another pin
  p 1 + 1 W          // Set next pin HIGH
  500 @              // Wait 500ms
  p 1 + 0 W          // Set next pin LOW
;

:D                     // Example: Count transitions
  p H h!             // Get hash for current pin
  h e ? 1 + h e ?!   // Increment counter
  `Pin ` p . ` transitions: ` h e ? . /N
;

:L                     // List callback settings
  `Callback Settings:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h h ? /T = (     // If callback enabled
      `Pin ` /i .
      ` -> Function ` h f ? .
      ` (Condition: ` h g ? .
      `)` /N
    )
  )
;

// Usage examples:
// I              // Initialize
// 2 0 M          // Set pin 2 as INPUT
// 2 A 1 X        // Set callback A for pin 2 on rising edge
// 2 B 2 X        // Set callback B for pin 2 on falling edge
// 3 C 0 X        // Set callback C for pin 3 on any change
// M              // Start monitoring
// L              // List callback settings
// 2 Y            // Disable callbacks for pin 2

```

This enhanced version adds a comprehensive callback system with these features:

1. Callback Assignment
   - Each pin can have one callback function
   - Callbacks are uppercase letters A-Y (Z reserved for interrupts)
   - Three trigger conditions:
     - 0: Any change
     - 1: Rising edge only
     - 2: Falling edge only

2. Callback Management
   - Enable/disable callbacks per pin
   - List active callbacks
   - Change callback conditions
   - Multiple example callback functions

3. Key Functions:
   - X: Set callback for a pin
   - Y: Disable callback for a pin
   - E: Execute callback if conditions met
   - L: List callback settings
   - A-D: Example callback functions

Example Usage:

```mint
// Initialize and setup
I               // Initialize everything
2 0 M           // Set pin 2 as INPUT
3 0 M           // Set pin 3 as INPUT

// Set up callbacks
2 A 1 X         // Pin 2: Call A on rising edge
3 B 2 X         // Pin 3: Call B on falling edge
4 C 0 X         // Pin 4: Call C on any change
5 D 0 X         // Pin 5: Call D on any change

// Start monitoring
M               // Start monitoring with callbacks

// Management
L               // List all callback settings
2 Y             // Disable callbacks for pin 2
```

Example Callback Functions:

1. Function A: Prints when pin goes HIGH
2. Function B: Prints when pin goes LOW
3. Function C: Toggles next pin (example of pin control)
4. Function D: Counts transitions (example of state tracking)

You can define more callbacks (up to Y) for different purposes like:
- Logging to memory
- Triggering outputs
- Starting timers
- Setting flags
- Controlling other devices
- Pattern detection
- State machine transitions

///////////////

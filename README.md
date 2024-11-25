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

1. Input pin polling/monitoring (1)
2. Creating pin-specific callbacks (2)
3. PWM output support (3) 
4. Implementing PWM measurement(4)
5. Pin interrupt handling (5)
6. Pin grouping for parallel output (6)
7. Custom pin modes (like analog input) (7)
8. Adding noise filtering (8)
9. Pin state change detection (9)
10. Pin timing functions (10)
11. Adding time stamping
12. Adding edge-triggered interrupts
13. Implementing pattern detection
14. 


 ///////////\\\\\\\\\\\\\\\\\/////////////

## pin polling/monitoring (1)
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
- 
Potential enhancements:
- Multiple callbacks per pin
- Callback priorities
- Conditional callback enabling
- Callback chaining
- Timed callbacks
- Event queuing

///////////////

## PWM output support (3)

PWM (Pulse Width Modulation) manager for the GPIO system that supports multiple PWM channels with configurable frequency and duty cycle.

```mint
// GPIO PWM Manager
// Uses array j for PWM periods (frequency)
// Uses array k for PWM duty cycles (0-100)
// Uses array l for PWM counters
// Uses array m for PWM enable flags
// PWM operates through interrupt Z

:I                     // Initialize GPIO and PWM
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 100 100 100 100         // PWM periods (default 100)
    100 100 100 100 ] j!    
  [ 0 0 0 0 0 0 0 0 ] k!    // Duty cycles
  [ 0 0 0 0 0 0 0 0 ] l!    // Counters
  [ /F /F /F /F /F /F /F /F ] m!  // Enable flags
;

:W                     // Start PWM on pin (pin duty-cycle period --)
  r! d! p!            // Store pin, duty%, period
  p H h!              // Get hash
  d 100 > (           // Limit duty cycle to 100%
    100 d!
  )
  d 0 < (             // Limit duty cycle to 0%
    0 d!
  )
  r 1 < (             // Minimum period is 1
    1 r!
  )
  p 1 M              // Set pin as OUTPUT
  r h j ?!            // Store period
  d h k ?!            // Store duty cycle
  0 h l ?!            // Reset counter
  /T h m ?!           // Enable PWM
;

:Q                     // Stop PWM on pin (pin --)
  p!                  // Store pin
  p H h!              // Get hash
  /F h m ?!           // Disable PWM
  0 p W              // Set pin LOW
;

:U                     // Update duty cycle (pin duty-cycle --)
  d! p!               // Store pin and duty cycle
  p H h!              // Get hash
  d 100 > (           // Limit duty cycle to 100%
    100 d!
  )
  d 0 < (             // Limit duty cycle to 0%
    0 d!
  )
  d h k ?!            // Store new duty cycle
;

:F                     // Update frequency (pin period --)
  r! p!               // Store pin and period
  p H h!              // Get hash
  r 1 < (             // Minimum period is 1
    1 r!
  )
  r h j ?!            // Store new period
  0 h l ?!            // Reset counter
;

:Z                     // Interrupt handler for PWM timing
  8 (                 // For each pin
    /i H h!           // Get hash
    h m ? /T = (      // If PWM enabled
      h l ? 1 + n!    // Increment counter
      n h j ? >= (    // If counter >= period
        0 n!          // Reset counter
      )
      n h k ? h j ? * 100 / < ( // If counter < duty point
        /i 1 W        // Set pin HIGH
      ) /E (
        /i 0 W        // Set pin LOW
      )
      n h l ?!        // Store counter
    )
  )
;

:S                     // Show PWM status
  `PWM Status:` /N
  8 (                 // For each pin
    /i H h!           // Get hash
    h m ? /T = (      // If PWM enabled
      `Pin ` /i .     // Show pin number
      ` Period: ` h j ? .
      ` Duty%: ` h k ? .
      ` Count: ` h l ? . /N
    )
  )
;

// Example PWM patterns
:A                     // Fade in over 10 steps
  p!                  // Store pin
  10 (               // 10 steps
    p /i 10 * 100 U  // Increase duty cycle
    500 @            // Wait 500ms
  )
;

:B                     // Fade out over 10 steps
  p!                  // Store pin
  10 (               // 10 steps
    p 100 /i 10 * - 100 U  // Decrease duty cycle
    500 @            // Wait 500ms
  )
;

:C                     // Pulse pattern
  p!                  // Store pin
  /U (               // Run forever
    p A              // Fade in
    p B              // Fade out
  )
;

// Example usage:
// I              // Initialize
// 2 50 100 W     // Start PWM on pin 2, 50% duty, period 100
// 2 75 100 U     // Update pin 2 to 75% duty cycle
// 2 50 U         // Update pin 2 to 50% duty cycle
// 2 200 F        // Update pin 2 to period 200 (slower)
// 2 Q            // Stop PWM on pin 2
// S              // Show PWM status
// 2 C            // Run pulse pattern on pin 2

```

This PWM implementation provides:

1. Core PWM Features:
   - Configurable period (frequency)
   - Adjustable duty cycle (0-100%)
   - Multiple PWM channels (up to 8 pins)
   - Independent control per pin

2. Key Functions:
   - W: Start PWM on a pin
   - Q: Stop PWM on a pin
   - U: Update duty cycle
   - F: Update frequency (period)
   - S: Show PWM status
   - Z: Interrupt handler for timing

3. Example Pattern Functions:
   - A: Fade in effect
   - B: Fade out effect
   - C: Continuous pulse pattern

Example Usage:

```mint
// Basic PWM
I               // Initialize
2 50 100 W      // Pin 2: 50% duty cycle, period 100
3 75 200 W      // Pin 3: 75% duty cycle, period 200

// Adjustments
2 75 U          // Change pin 2 to 75% duty
3 150 F         // Change pin 3 period to 150

// Status
S               // Show PWM status

// Effects
2 C             // Run pulse pattern on pin 2
```

The PWM system provides:

1. Hardware-independent timing
2. Multiple simultaneous PWM outputs
3. Dynamic duty cycle adjustment
4. Frequency control
5. Status monitoring
6. Pattern generation

Practical applications:

1. LED brightness control
2. Motor speed control
3. Tone generation
4. Servo control
5. Dimming effects
6. Temperature control via PWM

more enhancements:

1. Phase control between channels
2. Hardware PWM support
3. More complex patterns
4. Synchronized multi-pin patterns
5. Frequency sweep functions
6. PWM input measurement


////////////////////////////////////////////////////////////////

## Implementing PWM measurement(4)

Analyze incoming PWM signals to determine frequency, duty cycle, pulse width, and track changes in these values.

```mint
// PWM Input Measurement System
// Uses array n for rising edge timestamps
// Uses array o for falling edge timestamps
// Uses array p for period measurements
// Uses array q for duty cycle measurements
// Uses array r for pulse count tracking
// Uses array s for measurement valid flags

:I                     // Initialize PWM measurement
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] n!    // Rising edge times
  [ 0 0 0 0 0 0 0 0 ] o!    // Falling edge times
  [ 0 0 0 0 0 0 0 0 ] p!    // Periods
  [ 0 0 0 0 0 0 0 0 ] q!    // Duty cycles
  [ 0 0 0 0 0 0 0 0 ] r!    // Pulse counts
  [ /F /F /F /F /F /F /F /F ] s!  // Valid flags
;

:T                     // Get current timestamp (0-65535)
  #FF /I              // Read timer port
;

:M                     // Start measuring PWM on pin (pin --)
  p!                  // Store pin
  p H h!              // Get hash
  p 0 M              // Set as INPUT
  0 h r ?!            // Reset pulse count
  /F h s ?!           // Clear valid flag
;

:Z                     // Interrupt handler for edge detection
  8 (                 // For each pin
    /i H h!           // Get hash
    h a ? #FF00 & 8 } 0 = ( // If pin is INPUT
      /i #80 /I #01 & v!   // Read current value
      h n ? t!            // Get last rise time
      h o ? u!            // Get last fall time
      v /T = (            // If HIGH (rising edge)
        T h n ?!          // Store rise time
        t 0 > (           // If we have previous rise
          T t - w!        // Calculate period
          w 0 > (         // If period is positive
            w h p ?!      // Store period
            u t - 100 *   // Calculate duty cycle
            w /          // as percentage
            h q ?!        // Store duty cycle
            h r ? 1 + h r ?!  // Increment pulse count
            h r ? 10 > (  // After 10 pulses
              /T h s ?!   // Mark measurements valid
            )
          )
        )
      ) /E (             // On falling edge
        T h o ?!         // Store fall time
      )
    )
  )
;

:R                     // Read PWM measurements (pin --)
  p!                  // Store pin
  p H h!              // Get hash
  h s ? /T = (        // If measurements valid
    `Pin ` p . `: ` /N
    `  Period: ` h p ? . ` cycles` /N
    `  Freq: ` 100000 h p ? / . ` Hz` /N
    `  Duty%: ` h q ? . /N
    `  Pulses: ` h r ? . /N
  ) /E (
    `No valid measurements yet` /N
  )
;

:A                     // Analyze PWM changes over time
  p!                  // Store pin
  p H h!              // Get hash
  0 v!                // Reset variation counter
  [ 0 0 0 0 ] w!      // Store last 4 duty cycles
  10 (               // Sample 10 times
    h q ? x!         // Get current duty
    w 0 ? 0 > (      // If we have previous samples
      x w 0 ? - y!   // Calculate change
      y 5 > y -5 < | ( // If change > 5%
        v 1 + v!     // Count variation
      )
    )
    // Shift history
    w 0 ? w 1 ?!
    w 1 ? w 2 ?!
    w 2 ? w 3 ?!
    x w 3 ?!
    1000 @          // Wait 1 second
  )
  `Stability Analysis:` /N
  `Changes detected: ` v . /N
  v 3 > (
    `Signal is varying significantly` /N
  ) /E (
    `Signal is stable` /N
  )
;

:V                     // Verbose monitoring mode
  p!                  // Store pin
  `Monitoring PWM on pin ` p . /N
  /U (               // Run forever
    p R              // Read measurements
    1000 @           // Wait 1 second
    /N
  )
;

:C                     // Calculate statistics
  p!                  // Store pin
  p H h!              // Get hash
  h s ? /T = (        // If measurements valid
    `Statistics for pin ` p . `: ` /N
    `  Average Period: ` h p ? . ` cycles` /N
    `  Average Duty%: ` h q ? . /N
    `  Sample Count: ` h r ? . /N
    h q ? 20 < (     // Duty cycle categories
      `  Type: Low duty cycle` /N
    )
    h q ? 80 > (
      `  Type: High duty cycle` /N
    )
    h q ? 45 > h q ? 55 < & (
      `  Type: Square wave` /N
    )
  )
;

// Example usage:
// I              // Initialize
// 2 M            // Start measuring PWM on pin 2
// 2 R            // Read current measurements
// 2 A            // Analyze stability
// 2 V            // Start verbose monitoring
// 2 C            // Show statistics

```

This PWM measurement system provides comprehensive signal analysis:

1. Core Measurements:
   - Period/Frequency
   - Duty Cycle
   - Pulse Count
   - Edge Timing
   - Signal Stability

2. Key Functions:
   - M: Start measuring on a pin
   - R: Read current measurements
   - A: Analyze signal stability
   - V: Verbose monitoring mode
   - C: Calculate statistics
   - Z: Interrupt handler for measurement

Example Usage:

```mint
// Basic measurement
I               // Initialize
2 M             // Start measuring on pin 2
2 R             // Read measurements

// Advanced analysis
2 A             // Analyze stability
2 C             // Show statistics

// Continuous monitoring
2 V             // Start verbose monitoring
```

Features:

1. Real-time Measurements:
   - Accurate period measurement
   - Duty cycle calculation
   - Edge timing capture
   - Pulse counting

2. Signal Analysis:
   - Stability detection
   - Change tracking
   - Pattern recognition
   - Statistical analysis

3. Quality Features:
   - Measurement validation
   - Error detection
   - Signal categorization
   - Trend analysis

Practical Applications:

1. Servo Signal Analysis
   - Verify correct servo signals
   - Detect servo drift
   - Monitor servo response

2. Motor Control
   - Verify PWM drive signals
   - Monitor speed control
   - Detect motor issues

3. Signal Validation
   - Verify PWM generators
   - Test PWM circuits
   - Debug timing issues

4. Frequency Monitoring
   - Track frequency drift
   - Detect timing errors
   - Measure clock signals

Potential Enhancements:

1. Capture buffer for signal history
2. Advanced pattern detection
3. Frequency sweep analysis
4. Multi-channel synchronization
5. Noise filtering
6. Calibration features

//////////////////////////////////////////////////

## Pin interrupt handling (5)

Interrupt handling system for GPIO pins that supports multiple interrupt types, priorities, and custom handlers per pin.

```mint
// GPIO Pin Interrupt Handler
// Uses array t for interrupt types (0=none, 1=rising, 2=falling, 3=both, 4=level)
// Uses array u for interrupt priorities (0-7, 0=highest)
// Uses array v for interrupt counters
// Uses array w for interrupt timestamps
// Uses array x for debounce timeouts
// Uses array y for interrupt queues (4 deep per pin)

:I                     // Initialize interrupt system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] t!    // Interrupt types
  [ 0 0 0 0 0 0 0 0 ] u!    // Priorities
  [ 0 0 0 0 0 0 0 0 ] v!    // Counters
  [ 0 0 0 0 0 0 0 0 ] w!    // Timestamps
  [ 50 50 50 50            // Default 50ms debounce
    50 50 50 50 ] x!
  [ 0 0 0 0 0 0 0 0 ] y!    // Interrupt queues
  0 z!                      // Global interrupt flag
;

:N                     // Enable interrupts on pin
  t! r! p!             // pin, type, priority
  p H h!               // Get hash
  p 0 M               // Set as INPUT
  r h t ?!             // Store interrupt type
  t h u ?!             // Store priority
  0 h v ?!             // Clear counter
  0 h w ?!             // Clear timestamp
  /T z!                // Enable global interrupts
;

:F                     // Disable interrupts on pin
  p!                   // Store pin
  p H h!               // Get hash
  0 h t ?!             // Clear interrupt type
  0 h u ?!             // Clear priority
;

:Q                     // Add to interrupt queue
  v! p!                // Store pin and value
  p H h!               // Get hash
  h y ? 4 { #0F &     // Shift queue left
  v |                  // Add new value
  h y ?!               // Store updated queue
;

:D                     // Set debounce time for pin
  d! p!                // Store pin, debounce time
  p H h!               // Get hash
  d h x ?!             // Store debounce time
;

:T                     // Get current timestamp
  #FF /I              // Read timer port
;

:C                     // Check if debounce timeout expired
  p!                   // Store pin
  p H h!               // Get hash
  T h w ? -           // Time since last interrupt
  h x ? >             // Compare with debounce time
;

:Z                     // Main interrupt handler
  T s!                // Store current time
  8 (                 // Check all pins
    /i H h!           // Get hash for current pin
    h t ? 0 > (       // If interrupts enabled
      /i #80 /I #01 & v!   // Read current pin value
      h t ? 1 = v /T = & ( // Rising edge
        /i C (         // If debounce expired
          v /i Q       // Queue the event
          s h w ?!     // Update timestamp
          h v ? 1 + h v ?!  // Increment counter
          /i P         // Process interrupt
        )
      )
      h t ? 2 = v /F = & ( // Falling edge
        /i C (         // If debounce expired
          v /i Q       // Queue the event
          s h w ?!     // Update timestamp
          h v ? 1 + h v ?!  // Increment counter
          /i P         // Process interrupt
        )
      )
      h t ? 3 = (     // Both edges
        /i C (         // If debounce expired
          v /i Q       // Queue the event
          s h w ?!     // Update timestamp
          h v ? 1 + h v ?!  // Increment counter
          /i P         // Process interrupt
        )
      )
      h t ? 4 = v /T = & ( // Level trigger
        v /i Q         // Queue the event
        h v ? 1 + h v ?!  // Increment counter
        /i P           // Process interrupt
      )
    )
  )
;

:P                     // Process interrupt by priority
  p!                   // Store pin
  p H h!               // Get hash
  h u ? 0 = (         // Priority 0 (highest)
    p A               // Execute handler A
  )
  h u ? 1 = (         // Priority 1
    p B               // Execute handler B
  )
  h u ? 2 = (         // Priority 2
    p C               // Execute handler C
  )
;

// Example interrupt handlers
:A                     // High priority handler
  `High priority interrupt on pin ` p .
  ` Value: ` p #80 /I . /N
;

:B                     // Medium priority handler
  `Medium priority interrupt on pin ` p .
  ` Count: ` p H h! h v ? . /N
;

:C                     // Low priority handler
  `Low priority interrupt on pin ` p .
  ` Queue: ` p H h! h y ? , /N
;

:S                     // Show interrupt status
  `Interrupt Status:` /N
  8 (                 // For each pin
    /i H h!           // Get hash
    h t ? 0 > (       // If interrupts enabled
      `Pin ` /i .
      ` Type:` h t ? .
      ` Pri:` h u ? .
      ` Count:` h v ? .
      ` Queue:` h y ? , /N
    )
  )
;

:R                     // Reset all interrupt counters
  8 (                 // For each pin
    /i H h!           // Get hash
    0 h v ?!          // Clear counter
    0 h y ?!          // Clear queue
  )
;

// Example state machine using interrupts
:M                     // State machine example
  s!                  // Store initial state
  /U (               // Run forever
    s 0 = (          // State 0: Wait for trigger
      2 H h!         // Get pin 2 hash
      h y ? 0 > (    // If interrupt occurred
        1 s!         // Go to state 1
      )
    )
    s 1 = (          // State 1: Response
      3 1 W          // Set pin 3 HIGH
      1000 @         // Wait 1s
      3 0 W          // Set pin 3 LOW
      0 s!          // Return to state 0
    )
  )
;

// Example usage:
// I              // Initialize
// 2 1 0 N        // Pin 2: Rising edge, high priority
// 3 2 1 N        // Pin 3: Falling edge, medium priority
// 4 3 2 N        // Pin 4: Both edges, low priority
// 2 100 D        // Set 100ms debounce on pin 2
// S              // Show status
// R              // Reset counters
// 0 M            // Start state machine

```

This interrupt handling system provides:

1. Interrupt Types:
   - Rising edge (1)
   - Falling edge (2)
   - Both edges (3)
   - Level triggered (4)

2. Priority Levels:
   - 8 priority levels (0-7)
   - Priority-based handlers
   - Queue system per pin

3. Key Features:
   - Debounce handling
   - Event queuing
   - Timestamp tracking
   - Counter per pin
   - Status monitoring

4. Core Functions:
   - N: Enable interrupts
   - F: Disable interrupts
   - D: Set debounce time
   - S: Show status
   - R: Reset counters
   - Z: Main interrupt handler

Example Usage:

```mint
// Basic setup
I               // Initialize
2 1 0 N         // Pin 2: Rising edge, high priority
3 2 1 N         // Pin 3: Falling edge, medium priority
4 3 2 N         // Pin 4: Both edges, low priority

// Configuration
2 100 D         // Set 100ms debounce on pin 2
S               // Show status

// State machine
0 M             // Start state machine
```

Advanced Features:

1. Interrupt Queuing:
   - 4-deep queue per pin
   - FIFO handling
   - Queue overflow detection

2. Debounce Control:
   - Configurable per pin
   - Timestamp-based
   - Dynamic adjustment

3. State Machine:
   - Interrupt-driven
   - Multiple states
   - Conditional transitions

4. Priority System:
   - 8 priority levels
   - Priority-based handlers
   - Preemption support

Applications:

1. Button Handling:
   - Debounced input
   - Long/short press
   - Multi-button combos

2. Sensor Monitoring:
   - Edge detection
   - Level monitoring
   - Change tracking

3. Protocol Handling:
   - Timing measurement
   - Signal decoding
   - Pattern detection

4. Safety Systems:
   - Emergency stops
   - Limit switches
   - Fault detection

Potential Enhancements:

1. Nested interrupts
2. Interrupt masking
3. Edge timing analysis
4. Pattern matching
5. Event logging
6. Interrupt chaining


//////////////////////////////////////////////////////////////


## Pin grouping for parallel output (6)

A pin group manager that allows controlling multiple pins in parallel, useful for parallel data buses, LED arrays, or any application requiring synchronized pin control.

```mint
// GPIO Pin Group Manager
// Uses array g for group definitions (up to 8 groups)
// Uses array h for group masks
// Uses array i for group current values
// Uses array j for group directions
// Format: Each group byte: bits 0-7 = pin assignments

:I                     // Initialize group manager
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] g!    // Group definitions
  [ 0 0 0 0 0 0 0 0 ] h!    // Group masks
  [ 0 0 0 0 0 0 0 0 ] i!    // Current values
  [ 0 0 0 0 0 0 0 0 ] j!    // Direction (1=output)
;

:G                     // Create pin group (group mask --)
  m! n!               // Store group number and mask
  n H h!              // Get hash
  m h g ?!            // Store pin mask
  m h h ?!            // Store active mask
  0 h i ?!            // Clear current value
  #FF h j ?!          // Set as output by default
;

:D                     // Set group direction (group direction --)
  d! n!               // Store group and direction
  n H h!              // Get hash
  d h j ?!            // Store direction
  h g ? m!            // Get pin mask
  8 (                 // For each bit
    m #01 & (         // If bit is set
      /i d 1 = (      // If output
        1 M           // Set as output
      ) /E (
        0 M           // Set as input
      )
    )
    m 1 } m!         // Shift mask right
  )
;

:W                     // Write value to group (group value --)
  v! n!               // Store group and value
  n H h!              // Get hash
  v h h ? & v!        // Mask value with group mask
  v h i ?!            // Store current value
  h g ? m!            // Get pin mask
  v p!                // Copy value to work with
  8 (                 // For each bit
    m #01 & (         // If pin is in group
      p #01 & /i W    // Write bit to pin
    )
    m 1 } m!         // Shift mask right
    p 1 } p!         // Shift value right
  )
;

:R                     // Read value from group (group -- value)
  n!                  // Store group number
  n H h!              // Get hash
  0 v!                // Clear value
  h g ? m!            // Get pin mask
  8 (                 // For each bit
    m #01 & (         // If pin is in group
      v 1 { v!        // Shift value left
      /i #80 /I       // Read pin
      #01 & v |       // Combine with value
    )
    m 1 } m!         // Shift mask right
  )
  v .                 // Print value
;

:M                     // Mirror group to another group (src dst --)
  d! s!               // Store source and dest groups
  s R v!              // Read source value
  d v W               // Write to destination
;

:T                     // Toggle group bits (group mask --)
  m! n!               // Store group and mask
  n H h!              // Get hash
  h i ? m ^          // XOR with current value
  n W                // Write back
;

:S                     // Shift group left/right (group direction steps --)
  s! d! n!            // Store group, direction, steps
  n H h!              // Get hash
  h i ? v!            // Get current value
  s (                 // Repeat steps times
    d /T = (          // If shift left
      v 1 { v!        // Shift left
    ) /E (
      v 1 } v!        // Shift right
    )
  )
  n v W              // Write result
;

:B                     // Bit operations on group
  o! m! n!            // Store group, mask, operation
  n H h!              // Get hash
  h i ? v!            // Get current value
  o 0 = (             // AND operation
    v m & 
  )
  o 1 = (             // OR operation
    v m |
  )
  o 2 = (             // XOR operation
    v m ^
  )
  n W                // Write result
;

// Pattern generators
:P                     // Generate walking 1s pattern
  n!                  // Store group number
  1 v!                // Start with 1
  /U (               // Loop forever
    n v W            // Write pattern
    500 @            // Wait
    v 1 { v!        // Shift left
    v h g ? > (      // If beyond group size
      1 v!           // Reset to 1
    )
  )
;

:Q                     // Generate binary count pattern
  n!                  // Store group number
  0 v!                // Start with 0
  /U (               // Loop forever
    n v W            // Write pattern
    250 @            // Wait
    v 1 + v!        // Increment
    v h g ? > (      // If beyond group size
      0 v!           // Reset to 0
    )
  )
;

:L                     // List group status
  `Group Status:` /N
  8 (                 // For each group
    /i H h!           // Get hash
    h g ? 0 > (       // If group exists
      `Group ` /i .
      ` Mask: ` h g ? ,
      ` Value: ` h i ? ,
      ` Dir: ` h j ? . /N
    )
  )
;

// Example handlers for group interrupts
:Z                     // Interrupt handler
  8 (                 // For each group
    /i H h!           // Get hash
    h j ? 0 = (       // If input group
      /i R v!         // Read group
      v h i ? = /F = ( // If changed
        v h i ?!      // Update stored value
        /i A          // Call handler
      )
    )
  )
;

:A                     // Example group change handler
  `Group ` n .
  ` changed to ` v , /N
;

// Example usage:
// I              // Initialize
// 0 #0F G        // Create group 0 with pins 0-3
// 1 #F0 G        // Create group 1 with pins 4-7
// 0 #05 W        // Write 0b0101 to group 0
// 0 R            // Read group 0
// 0 P            // Start walking 1s pattern
// 0 Q            // Start binary count pattern
// L              // List group status

```

This pin group manager provides comprehensive parallel I/O control:

1. Group Management:
   - Create groups of up to 8 pins
   - Individual pin masking
   - Direction control (input/output)
   - Value tracking

2. Core Operations:
   - G: Create pin group
   - D: Set group direction
   - W: Write to group
   - R: Read from group
   - M: Mirror groups
   - T: Toggle bits
   - S: Shift operations
   - B: Bit operations

Example Usage:

```mint
// Setup groups
I               // Initialize
0 #0F G         // Group 0: pins 0-3
1 #F0 G         // Group 1: pins 4-7

// Basic operations
0 #05 W         // Write 0b0101 to group 0
0 R             // Read group 0
0 #FF T         // Toggle all bits in group 0

// Patterns
0 P             // Run walking 1s pattern
0 Q             // Run binary count pattern

// Status
L               // Show group status
```

Advanced Features:

1. Pattern Generation:
   - Walking 1s
   - Binary counting
   - Custom patterns
   - Shift operations

2. Group Operations:
   - Parallel read/write
   - Bit manipulation
   - Direction control
   - Mirroring

3. Interrupt Support:
   - Group change detection
   - Custom handlers
   - Pattern matching

Applications:

1. Data Bus Control:
   - Parallel data transfer
   - Bus monitoring
   - Address decoding

2. LED Arrays:
   - Pattern generation
   - Animation effects
   - Level indicators

3. Input Monitoring:
   - Switch arrays
   - Keypad scanning
   - Encoder reading

4. Protocol Implementation:
   - Parallel interfaces
   - Custom protocols
   - Timing control

Example Patterns:

1. Walking 1s:
```mint
0 P             // Start pattern on group 0
```

2. Binary Counter:
```mint
0 Q             // Start counter on group 0
```

3. Shift Register:
```mint
0 /T 4 S        // Shift left 4 positions
```

4. Bit Operations:
```mint
0 #AA 0 B       // AND with 10101010
```

Potential Enhancements:

1. Pattern memory
2. Timing control
3. State sequences
4. Pattern chaining
5. Event triggering
6. Group synchronization


///////////////////////////////////////////////////////////

## Custom pin modes (like analog input) (7)

A pin mode manager that handles analog input through an ADC, including features like sampling, averaging, and threshold detection.

```mint
// Custom Pin Mode Manager with ADC Support
// Uses array c for mode settings (0=digital, 1=analog, 2=pwm, 3=servo)
// Uses array d for analog values (10-bit resolution 0-1023)
// Uses array e for analog thresholds
// Uses array f for sample averaging counts
// ADC port is at #90 for control, #91 for data
// Control bits: bit 7=start, bit 6=done, bits 0-2=channel

:I                     // Initialize custom mode manager
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] c!    // Custom modes
  [ 0 0 0 0 0 0 0 0 ] d!    // Analog values
  [ 512 512 512 512         // Default thresholds at mid-range
    512 512 512 512 ] e!    
  [ 4 4 4 4 4 4 4 4 ] f!    // Default 4 samples for averaging
;

:A                     // Set pin as analog input
  p!                   // Store pin number
  p H h!              // Get hash
  1 h c ?!            // Set mode as analog
  0 h d ?!            // Clear value
  4 h f ?!            // Default 4 samples
;

:T                     // Set analog threshold
  t! p!               // Store pin and threshold
  p H h!              // Get hash
  t h e ?!            // Store threshold
;

:S                     // Set sample count for averaging
  s! p!               // Store pin and samples
  p H h!              // Get hash
  s h f ?!            // Store sample count
;

:R                     // Read analog value (raw)
  p!                  // Store pin number
  p H h!              // Get hash
  p #80 & #80 |      // Set start bit and channel
  #90 /O             // Start conversion
  /U (               // Wait for completion
    #90 /I          // Read status
    #40 & /T = /W   // Check done bit
  )
  #91 /I            // Read ADC value
  h d ?!             // Store in value array
;

:V                     // Read averaged analog value
  p!                  // Store pin number
  p H h!              // Get hash
  0 s!               // Clear sum
  h f ? n!           // Get sample count
  n (                // Take n samples
    p R              // Read raw value
    h d ? s +        // Add to sum
    10 @             // Small delay between samples
  )
  s n /             // Calculate average
  h d ?!             // Store result
  h d ? .            // Print result
;

:M                     // Monitor analog value with threshold
  p!                  // Store pin number
  p H h!              // Get hash
  h c ? 1 = (         // If analog mode
    p V              // Read averaged value
    h d ? h e ? > (  // Compare with threshold
      p C            // Call high handler
    ) /E (
      p D            // Call low handler
    )
  )
;

:Z                     // Interrupt handler for monitoring
  8 (                // For each pin
    /i H h!          // Get hash
    h c ? 1 = (      // If analog mode
      /i M           // Monitor pin
    )
  )
;

// Example threshold handlers
:C                     // Above threshold handler
  `Pin ` p .
  ` high: ` p H h! h d ? .
  ` > ` h e ? . /N
;

:D                     // Below threshold handler
  `Pin ` p .
  ` low: ` p H h! h d ? .
  ` < ` h e ? . /N
;

:L                     // Show analog pin status
  `Analog Pin Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h c ? 1 = (      // If analog mode
      `Pin ` /i .
      ` Value: ` h d ? .
      ` Thresh: ` h e ? .
      ` Samples: ` h f ? . /N
    )
  )
;

// Utility functions for voltage conversion
:U                     // Convert ADC to millivolts
  v!                  // Store ADC value
  v 5000 *           // Multiply by 5000mV
  1024 /            // Divide by resolution
  .                  // Print result
;

:F                     // Set threshold by voltage
  v! p!               // Store pin and voltage
  v 1024 *           // Scale to ADC range
  5000 /            // Divide by max voltage
  p T               // Set as threshold
;

// Example data logging
:G                     // Log readings to memory
  p! c!               // Store pin and count
  [                  // Create log array
    c (              // Take count readings
      p V           // Read value
      1000 @        // Wait 1 second
    )
  ] l!              // Save to array l
;

// Example voltage monitoring
:W                     // Watch voltage with alerts
  p!                  // Store pin
  /U (               // Run forever
    p V             // Read value
    p H h! h d ? U  // Convert to mV
    1000 @          // Wait 1 second
  )
;

// Example usage:
// I              // Initialize
// 2 A            // Set pin 2 as analog
// 2 8 S          // Set 8 samples for averaging
// 2 600 T        // Set threshold at 600 (about 3V)
// 2 3000 F       // Set threshold at 3000mV
// 2 V            // Read averaged value
// 2 M            // Monitor with threshold
// 2 10 G         // Log 10 readings
// 2 W            // Watch voltage
// L              // Show status

// Advanced feature - moving average
:N                     // Setup moving average
  s! p!               // Store pin and window size
  p H h!              // Get hash
  [                  // Create average array
    s (              // Fill with initial readings
      p R           // Read raw value
      10 @          // Small delay
    )
  ] m!              // Save to array m
;

:O                     // Read moving average
  p!                  // Store pin
  p H h!              // Get hash
  m 0 ?              // Get first value
  m /S 1 - (         // For remaining values
    /i 1 + m ? +    // Sum values
  )
  m /S /            // Divide by count
  .                  // Print result
;

```

This custom pin mode manager with ADC support provides:

1. Core ADC Features:
   - 10-bit resolution (0-1023)
   - Multiple channel support
   - Configurable sampling
   - Threshold detection

2. Key Functions:
   - A: Set analog mode
   - T: Set threshold
   - S: Set sample count
   - R: Raw reading
   - V: Averaged reading
   - M: Monitor with threshold
   - U: Convert to millivolts
   - F: Set threshold by voltage

Example Usage:

```mint
// Basic setup
I               // Initialize
2 A             // Set pin 2 as analog input
2 8 S           // Use 8 samples for averaging

// Threshold monitoring
2 600 T         // Set threshold at ADC value 600
2 3000 F        // Or set threshold at 3V (3000mV)

// Reading values
2 V             // Read averaged value
2 M             // Monitor with threshold checking

// Voltage monitoring
2 W             // Continuous voltage display
```

Advanced Features:

1. Averaging Methods:
   - Simple averaging
   - Moving average
   - Configurable samples

2. Voltage Conversion:
   - ADC to millivolts
   - Voltage thresholds
   - Scaling support

3. Data Logging:
   - Array storage
   - Timed sampling
   - Value tracking

4. Monitoring:
   - Threshold detection
   - Change notification
   - Continuous watching

Applications:

1. Sensor Reading:
   - Temperature sensors
   - Light sensors
   - Potentiometers
   - Voltage monitoring

2. Process Control:
   - Level detection
   - Threshold monitoring
   - Alarm systems

3. Data Acquisition:
   - Signal logging
   - Trend analysis
   - Event detection

Example Monitoring:

1. Simple Voltage Reading:
```mint
2 V             // Read average
2 R U           // Read and convert to mV
```

2. Threshold Monitoring:
```mint
2 3000 F        // Set 3V threshold
2 M             // Monitor with alerts
```

3. Data Logging:
```mint
2 10 G          // Log 10 readings
```

4. Moving Average:
```mint
2 8 N           // Setup 8-sample moving average
2 O             // Read moving average
```

Potential Enhancements:

1. Calibration support
2. Multiple threshold levels
3. Rate of change detection
4. Signal filtering
5. Peak detection
6. Statistical analysis

//////////////////////////////////////////////////////////////////

## Adding noise filtering (8)

Noise filtering system that works with both ADC input and DAC output, including several filtering methods and signal processing capabilities.

```mint
// Signal Filtering System for ADC/DAC
// ADC port at #90 (control) and #91 (data)
// DAC port at #92 (control) and #93 (data)
// Uses array n for raw samples
// Uses array o for filtered values
// Uses array p for filter coefficients
// Uses array q for filter history
// Uses array r for noise thresholds

:I                     // Initialize filtering system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] n!    // Raw samples
  [ 0 0 0 0 0 0 0 0 ] o!    // Filtered values
  [ 4 4 4 4 4 4 4 4 ] p!    // Default coefficients
  [ 0 0 0 0 0 0 0 0 ] q!    // Filter history
  [ 10 10 10 10            // Default noise threshold
    10 10 10 10 ] r!       // (about 1% of range)
;

:R                     // Read ADC with noise rejection
  p!                   // Store pin number
  p H h!              // Get hash
  p #80 & #80 |       // Set start bit and channel
  #90 /O              // Start conversion
  /U (                // Wait for completion
    #90 /I           // Read status
    #40 & /T = /W    // Check done bit
  )
  #91 /I             // Read ADC value
  h n ?!              // Store raw value
;

:W                     // Write to DAC with filtering
  v! p!               // Store pin and value
  p H h!              // Get hash
  v h o ?!            // Store to filtered array
  p #80 & v |         // Combine channel and value
  #93 /O             // Output to DAC
;

:M                     // Median filter (3 samples)
  p!                  // Store pin
  p H h!              // Get hash
  p R                 // Read first sample
  h n ? x!           // Store in x
  10 @               // Small delay
  p R                // Read second sample
  h n ? y!           // Store in y
  10 @               // Small delay
  p R                // Read third sample
  h n ? z!           // Store in z
  // Sort and take middle value
  x y > (x y $ y! x!) 
  y z > (y z $ z! y!)
  x y > (x y $ y! x!)
  y h o ?!            // Store median
;

:A                     // Average filter
  p!                  // Store pin
  p H h!              // Get hash
  0 s!               // Clear sum
  h p ? n!           // Get sample count
  n (                // Take n samples
    p R              // Read raw value
    h n ? s +        // Add to sum
    5 @              // Small delay
  )
  s n /             // Calculate average
  h o ?!             // Store result
;

:E                     // Exponential filter
  v! p!               // Store pin and value
  p H h!              // Get hash
  h q ? 8 *          // Get history * 8
  v +                // Add new value
  9 /               // Divide by 9 (alpha ≈ 0.889)
  h q ?!             // Store back to history
  h o ?!             // Store filtered result
;

:K                     // Kalman-inspired filter
  p!                  // Store pin
  p H h!              // Get hash
  p R                 // Read new value
  h n ? v!           // Store in v
  h q ? w!           // Get prediction
  v w - 4 / w +      // Simple Kalman update
  h q ?!             // Store new prediction
  h o ?!             // Store filtered result
;

:D                     // Detect noise spike
  v! p!               // Store pin and value
  p H h!              // Get hash
  v h o ? - y!       // Calculate difference
  y 0 < (            // Make absolute
    y -1 * y!
  )
  y h r ? >          // Compare with threshold
;

:F                     // Set noise filter mode
  m! p!               // Store pin and mode
  p H h!              // Get hash
  m h p ?!            // Store filter mode
;

:T                     // Set noise threshold
  t! p!               // Store pin and threshold
  p H h!              // Get hash
  t h r ?!            // Store threshold
;

:S                     // Sample and filter
  p!                  // Store pin
  p H h!              // Get hash
  h p ? 1 = (         // Median filter
    p M
  )
  h p ? 2 = (         // Average filter
    p A
  )
  h p ? 3 = (         // Exponential filter
    p R              // Read value
    h n ? p E        // Apply exp filter
  )
  h p ? 4 = (         // Kalman filter
    p K
  )
  h o ?              // Get filtered value
  p D /F = (         // If not noise
    p W              // Output to DAC
  )
;

:Z                     // Filter monitor interrupt
  8 (                // For each pin
    /i H h!          // Get hash
    h p ? 0 > (      // If filtering enabled
      /i S           // Sample and filter
    )
  )
;

// Example filter patterns
:P                     // High-frequency noise removal
  p!                  // Store pin
  p 2 F              // Set average filter
  p 8 T              // Set noise threshold
  /U (               // Run forever
    p S             // Sample and filter
    50 @            // Wait 50ms
  )
;

:Q                     // Spike detection
  p!                  // Store pin
  p 1 F              // Set median filter
  p 20 T             // Set higher threshold
  /U (               // Run forever
    p S             // Sample and filter
    p H h!          // Get hash
    h n ? h o ? - y! // Calculate difference
    y 0 < (         // Make absolute
      y -1 * y!
    )
    y h r ? > (     // If spike detected
      `Spike on pin ` p .
      ` Value: ` h n ? . /N
    )
    25 @            // Wait 25ms
  )
;

:L                     // Show filter status
  `Filter Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h p ? 0 > (      // If filtering enabled
      `Pin ` /i .
      ` Mode: ` h p ? .
      ` Raw: ` h n ? .
      ` Filtered: ` h o ? .
      ` Threshold: ` h r ? . /N
    )
  )
;

// Example usage:
// I              // Initialize
// 2 1 F          // Set pin 2 to median filter
// 2 2 F          // Set pin 2 to average filter
// 2 3 F          // Set pin 2 to exponential filter
// 2 4 F          // Set pin 2 to Kalman filter
// 2 10 T         // Set noise threshold to 10
// 2 S            // Sample and filter once
// 2 P            // Start noise removal
// 2 Q            // Start spike detection
// L              // Show status

// Advanced analysis
:X                     // Statistical analysis
  p!                  // Store pin
  [                  // Create stats array
    10 (             // Take 10 samples
      p S           // Sample and filter
      10 @          // Wait
    )
  ] s!              // Store in array s
  // Calculate min/max/avg
  s 0 ? m!          // Initialize min
  s 0 ? x!          // Initialize max
  0 a!              // Clear average
  s /S (            // For each sample
    /i s ? a +      // Add to average
    /i s ? m < (    // Check for new min
      /i s ? m!
    )
    /i s ? x > (    // Check for new max
      /i s ? x!
    )
  )
  `Pin ` p .
  ` Min: ` m .
  ` Max: ` x .
  ` Avg: ` a s /S / . /N
;

```

This signal filtering system provides:

1. Filter Types:
   - Median (noise spikes)
   - Average (general smoothing)
   - Exponential (weighted history)
   - Kalman-inspired (predictive)

2. Key Features:
   - Multiple filter modes
   - Adjustable thresholds
   - Spike detection
   - Statistical analysis

3. Core Functions:
   - F: Set filter mode
   - T: Set threshold
   - S: Sample and filter
   - P: Noise removal
   - Q: Spike detection
   - X: Statistical analysis

Example Usage:

```mint
// Basic setup
I               // Initialize
2 1 F           // Set median filter on pin 2
2 10 T          // Set noise threshold

// Different filters
2 2 F           // Switch to average filter
2 3 F           // Switch to exponential filter
2 4 F           // Switch to Kalman filter

// Monitoring
2 P             // Start noise removal
2 Q             // Start spike detection
2 X             // Run statistical analysis
```

Filter Characteristics:

1. Median Filter:
   - Best for spike removal
   - Preserves edges
   - 3-sample window

2. Average Filter:
   - Good general smoothing
   - Adjustable samples
   - Reduces random noise

3. Exponential Filter:
   - Smooth response
   - Memory efficient
   - Good for trends

4. Kalman Filter:
   - Predictive filtering
   - Good for tracking
   - Handles systematic noise

Applications:

1. Sensor Reading:
   - Remove power line noise
   - Smooth temperature readings
   - Filter pressure sensors

2. Signal Processing:
   - Clean audio signals
   - Process analog sensors
   - Remove interference

3. Control Systems:
   - Smooth control inputs
   - Filter feedback signals
   - Detect anomalies

Example Scenarios:

1. Clean Noisy Sensor:
```mint
2 2 F           // Average filter
2 5 T           // Low threshold
2 P             // Start filtering
```

2. Detect Spikes:
```mint
2 1 F           // Median filter
2 20 T          // High threshold
2 Q             // Start detection
```

3. Smooth Control Signal:
```mint
2 3 F           // Exponential filter
2 10 T          // Medium threshold
2 S             // Sample and filter
```

Potential Enhancements:

1. Butterworth filter
2. Moving average
3. FFT analysis
4. Adaptive thresholds
5. Band-pass filtering
6. Auto-calibration




//////////////////////////////////////////////////////////////////
## Pin state change detection (9)
A pin state change detection system that can track changes, timing, and patterns across multiple pins.

```mint
// Pin State Change Detection System
// Uses array c for previous states
// Uses array d for change counters
// Uses array e for last change timestamps
// Uses array f for debounce settings
// Uses array g for pattern detection
// Uses array h for timing measurements
// Uses array i for change history (4 deep per pin)

:I                     // Initialize change detection
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] c!    // Previous states
  [ 0 0 0 0 0 0 0 0 ] d!    // Change counters
  [ 0 0 0 0 0 0 0 0 ] e!    // Timestamps
  [ 50 50 50 50            // Default 50ms debounce
    50 50 50 50 ] f!
  [ 0 0 0 0 0 0 0 0 ] g!    // Pattern buffer
  [ 0 0 0 0 0 0 0 0 ] h!    // Timing buffer
  [ 0 0 0 0 0 0 0 0 ] i!    // History buffer
;

:T                     // Get current timestamp
  #FF /I              // Read timer port
;

:D                     // Set debounce time for pin
  t! p!               // Store pin and time
  p H h!              // Get hash
  t h f ?!            // Store debounce time
;

:H                     // Update history buffer
  s! p!               // Store pin and state
  p H h!              // Get hash
  h i ? 4 { #0F &     // Shift history left
  s |                 // Add new state
  h i ?!              // Store updated history
;

:C                     // Check for state change
  p!                  // Store pin number
  p H h!              // Get hash
  T t!               // Get current time
  p #80 /I #01 & s!  // Read current state
  s h c ? = /F = (   // If state changed
    t h e ? - y!     // Calculate time since last change
    y h f ? > (      // If beyond debounce time
      s h c ?!       // Update previous state
      t h e ?!       // Update timestamp
      h d ? 1 + h d ?! // Increment change counter
      s p H          // Store in history
      p P            // Process change
    )
  )
;

:P                     // Process state change
  p!                  // Store pin
  p H h!              // Get hash
  h i ? #0F &         // Get history
  h g ? = (           // If matches pattern
    p M              // Call pattern match handler
  )
  h c ? /T = (        // If changed to high
    p R              // Call rise handler
  ) /E (
    p F              // Call fall handler
  )
;

// Example handlers
:R                     // Rising edge handler
  `Pin ` p .
  ` rose at ` T . /N
;

:F                     // Falling edge handler
  `Pin ` p .
  ` fell at ` T . /N
;

:M                     // Pattern match handler
  `Pattern match on pin ` p .
  ` pattern: ` p H h! h g ? , /N
;

:A                     // Set pattern to detect
  n! p!               // Store pin and pattern
  p H h!              // Get hash
  n h g ?!            // Store pattern
;

:W                     // Watch for timing pattern
  t! p!               // Store pin and target time
  p H h!              // Get hash
  T w!               // Start time
  /U (               // Loop until pattern found
    p C              // Check for change
    T w - t = (      // If target time reached
      h i ? #0F &    // Get history
      h g ? = (      // If pattern matches
        `Pattern found in ` t . `ms` /N
        /B          // Break loop
      )
    )
  )
;

:S                     // Show pin status
  `Pin Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    `Pin ` /i .
    ` State: ` h c ? .
    ` Changes: ` h d ? .
    ` Last: ` h e ? .
    ` History: ` h i ? , /N
  )
;

:Z                     // Interrupt handler
  8 (                // For each pin
    /i C            // Check each pin for changes
  )
;

// Advanced pattern detection
:B                     // Define bit pattern
  m! t! p!            // Store pin, timing, pattern
  p H h!              // Get hash
  m h g ?!            // Store pattern
  t h h ?!            // Store timing
;

:V                     // Verify bit pattern
  p!                  // Store pin
  p H h!              // Get hash
  h i ? #0F &         // Get history
  h g ? = (           // If pattern matches
    h h ? y!         // Get expected timing
    T h e ? - y = (  // If timing matches
      /T            // Return true
    ) /E (
      /F            // Return false
    )
  ) /E (
    /F              // Return false
  )
;

// Edge timing measurement
:E                     // Measure edge timing
  d! p!               // Store pin and duration
  p H h!              // Get hash
  0 c!               // Clear counter
  T t!               // Start time
  /U (               // Loop
    p C              // Check for change
    c d < /W         // While count < duration
    c 1 + c!         // Increment counter
  )
  `Edge timings for pin ` p . `:` /N
  d (                // For each edge
    /i . `: ` h h ? /i ? . `ms` /N
  )
;

// Advanced analysis functions
:L                     // Analyze change frequency
  p!                  // Store pin
  p H h!              // Get hash
  h d ? 0 > (         // If changes exist
    T h e ? - y!     // Get time since first change
    y 0 > (          // If time valid
      h d ? 1000 *   // Changes per second
      y /           // Divide by time in ms
      `Frequency: ` . ` changes/sec` /N
    )
  )
;

:Q                     // Reset statistics
  p!                  // Store pin
  p H h!              // Get hash
  0 h d ?!            // Clear counter
  0 h e ?!            // Clear timestamp
  0 h i ?!            // Clear history
;

// Example usage:
// I              // Initialize
// 2 50 D         // Set 50ms debounce on pin 2
// 2 #0101 A      // Set pattern to detect
// 2 C            // Check for changes
// 2 1000 E       // Measure edge timing for 1 second
// 2 L            // Analyze frequency
// S              // Show status
// 2 Q            // Reset pin 2 statistics

// Pattern monitoring example
:X                     // Monitor for specific sequence
  p!                  // Store pin
  [ 0 0 0 0 ] q!     // Create sequence buffer
  /U (               // Loop forever
    p C              // Check for change
    p H h! h c ? r!  // Get current state
    // Shift buffer
    q 0 ? q 1 ?!
    q 1 ? q 2 ?!
    q 2 ? q 3 ?!
    r q 3 ?!
    // Check sequence
    q 0 ? 1 = 
    q 1 ? 0 = &
    q 2 ? 1 = &
    q 3 ? 1 = & (
      `Sequence detected on pin ` p . /N
    )
  )
;

// Timing analysis
:Y                     // Measure duty cycle
  p!                  // Store pin
  0 h! 0 l!          // Clear high/low times
  T t!               // Start time
  1000 (             // Measure for 1 second
    p #80 /I #01 & ( // If pin high
      h 1 + h!       // Increment high time
    ) /E (
      l 1 + l!       // Increment low time
    )
    1 @              // 1ms delay
  )
  `Duty cycle: `
  h 100 * h l + / .  // Calculate percentage
  `%` /N
;

```

This state change detection system provides:

1. Core Features:
   - Debounced change detection
   - Edge timing measurement
   - Pattern recognition
   - Change history tracking
   - Frequency analysis

2. Key Functions:
   - C: Check for changes
   - D: Set debounce time
   - A: Set pattern to detect
   - W: Watch for pattern
   - E: Measure edge timing
   - L: Analyze frequency
   - Y: Measure duty cycle

Example Usage:

```mint
// Basic setup
I               // Initialize
2 50 D          // Set 50ms debounce on pin 2

// Pattern detection
2 #0101 A       // Set pattern to detect
2 W             // Watch for pattern

// Analysis
2 1000 E        // Measure edges for 1 second
2 L             // Show frequency
2 Y             // Measure duty cycle
```

Advanced Features:

1. Pattern Detection:
   - Bit patterns
   - Timing patterns
   - Sequence detection
   - Pattern matching

2. Timing Analysis:
   - Edge timing
   - Duty cycle
   - Frequency
   - Interval measurement

3. History Tracking:
   - 4-deep history per pin
   - Change counting
   - Timestamp recording
   - Pattern matching

Applications:

1. Button Handling:
   - Double click detection
   - Long press detection
   - Combo detection
   - Press timing

2. Protocol Decoding:
   - Serial data
   - Infrared signals
   - Encoded messages
   - Timing verification

3. Sensor Monitoring:
   - Motion detection
   - Event counting
   - State tracking
   - Timing analysis

Example Patterns:

1. Double Click Detection:
```mint
2 #0101 A       // Set pattern
2 500 W         // Watch for pattern within 500ms
```

2. Long Press Detection:
```mint
2 E             // Measure edge timing
```

3. Duty Cycle Analysis:
```mint
2 Y             // Measure duty cycle
```

Potential Enhancements:

1. Multi-pin patterns
2. Variable timing windows
3. Pattern queuing
4. Event logging
5. Statistical analysis
6. Trigger actions


//////////////////////////////////////////////////////////////////

##  Pin timing functions (10)

A pin timing system that can handle various timing functions, including pulse generation, timing measurements, and scheduled pin operations.

```mint
// Pin Timing Functions System
// Uses array t for timing periods
// Uses array u for timing counters
// Uses array v for timing modes
// Uses array w for next scheduled events
// Uses array x for timing patterns
// Uses array y for interval measurements
// Uses array z for timing flags

:I                     // Initialize timing system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] t!    // Timing periods
  [ 0 0 0 0 0 0 0 0 ] u!    // Counters
  [ 0 0 0 0 0 0 0 0 ] v!    // Modes (0=off,1=pulse,2=toggle,3=schedule)
  [ 0 0 0 0 0 0 0 0 ] w!    // Next events
  [ 0 0 0 0 0 0 0 0 ] x!    // Patterns
  [ 0 0 0 0 0 0 0 0 ] y!    // Intervals
  [ /F /F /F /F /F /F /F /F ] z!  // Flags
;

:T                     // Get current timestamp
  #FF /I              // Read timer port
;

:P                     // Generate pulse
  d! p!               // Store pin and duration
  p H h!              // Get hash
  1 h v ?!            // Set pulse mode
  d h t ?!            // Store duration
  T h w ?!            // Store start time
  p 1 W              // Set pin HIGH
;

:G                     // Toggle pin at interval
  i! p!               // Store pin and interval
  p H h!              // Get hash
  2 h v ?!            // Set toggle mode
  i h t ?!            // Store interval
  T h w ?!            // Store next toggle time
;

:S                     // Schedule pin change
  s! t! p!            // Store pin, time, state
  p H h!              // Get hash
  3 h v ?!            // Set schedule mode
  t h w ?!            // Store trigger time
  s h x ?!            // Store target state
;

:M                     // Measure pulse width
  p!                  // Store pin
  p H h!              // Get hash
  T t!               // Store start time
  /U (               // Wait for edge
    p #80 /I #01 & /T = /W
  )
  T u!               // Store rise time
  /U (               // Wait for next edge
    p #80 /I #01 & /F = /W
  )
  T u - .            // Print pulse width
;

:D                     // Measure duty cycle
  p!                  // Store pin
  0 h! 0 l!          // Clear high/low times
  T t!               // Start time
  1000 (             // Measure for 1 second
    p #80 /I #01 & ( // If pin high
      h 1 + h!       // Increment high time
    ) /E (
      l 1 + l!       // Increment low time
    )
    1 @              // 1ms delay
  )
  `Duty cycle: `
  h 100 * h l + / .  // Calculate percentage
  `%` /N
;

:F                     // Frequency measurement
  p!                  // Store pin
  0 c!               // Clear counter
  T t!               // Store start time
  1000 (             // Count for 1 second
    p #80 /I #01 & s!
    s r = /F = (     // If state changed
      c 1 + c!       // Increment counter
    )
    s r!             // Store state
    1 @              // 1ms delay
  )
  c 2 / .            // Print frequency (cycles/sec)
;

:Z                     // Interrupt handler
  8 (                // For each pin
    /i H h!          // Get hash
    h v ? 1 = (      // Pulse mode
      T h w ? - h t ? >= ( // If duration exceeded
        /i 0 W       // Set pin LOW
        0 h v ?!     // Clear mode
      )
    )
    h v ? 2 = (      // Toggle mode
      T h w ? >= (   // If interval reached
        /i #80 /I #01 & /F = /i W  // Toggle pin
        T h t ? + h w ?!  // Set next toggle
      )
    )
    h v ? 3 = (      // Schedule mode
      T h w ? >= (   // If time reached
        /i h x ? W   // Set scheduled state
        0 h v ?!     // Clear mode
      )
    )
  )
;

// Advanced timing patterns
:X                     // Generate timing pattern
  p! n!               // Store pin and pattern count
  [                  // Create pattern array
    n (              // For each pattern element
      `Duration (ms): ` /K 48 - #0A * y!  // Get duration
      `State (0/1): ` /K 48 - x!         // Get state
      y x            // Push duration and state
    )
  ] q!              // Store pattern
  /U (              // Run pattern
    q /S (          // For each pattern element
      /i 2 * q ? p W  // Set state
      /i 2 * 1 + q ? @  // Wait duration
    )
  )
;

:W                     // Write pin with timing
  v! p!               // Store value and pin
  p H h!              // Get hash
  h z ? /T = (        // If timing active
    T h y ? <= (      // If within timing window
      p v W          // Write value
    )
  )
;

:B                     // Blink pattern
  d! n! p!            // Store pin, repeats, delay
  p H h!              // Get hash
  /T h z ?!           // Set timing active
  n (                // Repeat n times
    p 1 W            // Set HIGH
    d 2 / @          // Wait half period
    p 0 W            // Set LOW
    d 2 / @          // Wait half period
  )
  /F h z ?!          // Clear timing
;

:L                     // Show timing status
  `Timing Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h v ? 0 > (      // If timing active
      `Pin ` /i .
      ` Mode: ` h v ? .
      ` Period: ` h t ? .
      ` Next: ` h w ? . /N
    )
  )
;

// Example patterns
:A                     // Fast blink
  p!                  // Store pin
  5 (                // 5 times
    p 1 W            // HIGH
    100 @            // 100ms
    p 0 W            // LOW
    100 @            // 100ms
  )
;

:C                     // Morse code dot
  p!                  // Store pin
  p 1 W              // HIGH
  100 @              // 100ms
  p 0 W              // LOW
  100 @              // 100ms
;

:E                     // Morse code dash
  p!                  // Store pin
  p 1 W              // HIGH
  300 @              // 300ms
  p 0 W              // LOW
  100 @              // 100ms
;

// Example usage:
// I              // Initialize
// 2 1000 P       // 1 second pulse on pin 2
// 2 500 G        // Toggle pin 2 every 500ms
// 2 1000 1 S     // Schedule pin 2 HIGH in 1000ms
// 2 M            // Measure pulse width on pin 2
// 2 D            // Measure duty cycle on pin 2
// 2 F            // Measure frequency on pin 2
// 2 3 100 B      // Blink pin 2 3 times with 100ms period
// L              // Show timing status

// Custom timing sequence
:K                     // Complex timing sequence
  p!                  // Store pin
  p C                 // Dot
  p C                 // Dot
  p E                 // Dash
  p C                 // Dot
  500 @              // Word space
;

// PWM with variable timing
:Y                     // Variable duty cycle PWM
  d! p!               // Store pin and duty cycle
  /U (               // Run forever
    p 1 W            // HIGH
    d @              // On time
    p 0 W            // LOW
    100 d - @        // Off time
  )
;

// Interval measurement
:N                     // Measure interval between events
  p!                  // Store pin
  T t!               // Start time
  /U (               // Wait for first edge
    p #80 /I #01 & /T = /W
  )
  /U (               // Wait for second edge
    p #80 /I #01 & /T = /W
  )
  T t - .            // Print interval
;

```

This pin timing system provides:

1. Core Functions:
   - P: Generate pulse
   - G: Toggle at interval
   - S: Schedule changes
   - M: Measure pulse width
   - D: Measure duty cycle
   - F: Measure frequency

2. Advanced Features:
   - X: Custom patterns
   - B: Blink patterns
   - Y: Variable PWM
   - N: Interval measurement

Example Usage:

```mint
// Basic timing
I               // Initialize
2 1000 P        // 1 second pulse
2 500 G         // 500ms toggle

// Measurements
2 M             // Measure pulse
2 D             // Measure duty cycle
2 F             // Measure frequency

// Patterns
2 3 100 B       // Blink 3 times
2 K             // Run morse sequence
```

Applications:

1. Signal Generation:
   - Pulse generation
   - PWM control
   - Pattern generation
   - Morse code

2. Timing Measurement:
   - Pulse width
   - Frequency
   - Duty cycle
   - Intervals

3. Scheduled Operations:
   - Delayed actions
   - Timed sequences
   - Periodic events
   - Pattern playback

Example Patterns:

1. Simple Blink:
```mint
2 3 100 B       // Blink 3 times, 100ms period
```

2. Morse Code:
```mint
2 K             // Send morse pattern
```

3. Custom PWM:
```mint
2 75 Y          // 75% duty cycle PWM
```

Features:

1. Timing Control:
   - Precise delays
   - Interval timing
   - Schedule events
   - Pattern generation

2. Measurements:
   - Width/duration
   - Frequency
   - Duty cycle
   - Intervals

3. Pattern Support:
   - Custom sequences
   - Repeating patterns
   - Complex timing
   - Variable duty cycles

Potential Enhancements:

1. Phase control
2. Multiple synchronized outputs
3. Pattern memory
4. Timing triggers
5. Event sequences
6. Time-based state machines


//////////////////////////////////////////////////////////////////

## Adding time stamping

A timestamping system that interfaces with a real-time clock (RTC) chip via SPI, enabling accurate time tracking and event timestamping.

```mint
// Timestamping System with RTC via SPI
// Uses ports: #A0=SPI data, #A1=SPI clock, #A2=CS
// RTC registers: 00=seconds, 01=minutes, 02=hours
//               03=day, 04=date, 05=month, 06=year
// Uses array t for timestamp storage (8 per event)
// Uses array u for event types
// Uses array v for event counters
// Uses array w for circular buffer indexes

:I                     // Initialize timestamp system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] t!    // Timestamp buffer
  [ 0 0 0 0 0 0 0 0 ] u!    // Event types
  [ 0 0 0 0 0 0 0 0 ] v!    // Event counters
  [ 0 0 0 0 0 0 0 0 ] w!    // Buffer indexes
;

:O                     // SPI write byte
  d!                   // Store data byte
  8 (                 // 8 bits
    d #80 & #A0 /O    // Output MSB to data line
    #01 #A1 /O        // Clock pulse
    0 #A1 /O 
    d 1 { d!          // Shift left
  )
;

:N                     // SPI read byte
  0 r!                // Clear result
  8 (                 // 8 bits
    r 1 { r!          // Shift left
    #A0 /I #80 & (    // Read data bit
      r 1 | r!        // Set bit if high
    )
    #01 #A1 /O        // Clock pulse
    0 #A1 /O
  )
  r                   // Return byte
;

:R                     // Read RTC register
  a!                  // Store address
  0 #A2 /O            // CS low
  a O                 // Send address
  N n!                // Read data
  1 #A2 /O            // CS high
  n                   // Return data
;

:W                     // Write RTC register
  d! a!               // Store address and data
  0 #A2 /O            // CS low
  a #80 | O           // Send address with write bit
  d O                 // Send data
  1 #A2 /O            // CS high
;

:S                     // Setup RTC
  #80 #0E W           // Enable write
  #00 #0F W           // Clear oscillator stop bit
  0 0 W               // Clear seconds
  #7F #0E W           // Disable write
;

:T                     // Read current time
  0 R s!              // Read seconds
  1 R m!              // Read minutes
  2 R h!              // Read hours
  3 R w!              // Read day
  4 R d!              // Read date
  5 R n!              // Read month
  6 R y!              // Read year
;

:C                     // Create timestamp entry
  e! p!               // Store pin and event type
  p H h!              // Get hash
  [ T                // Get current time
    s m h w d n y     // Store all components
  ] q!               // Save to array
  h w ? i!           // Get buffer index
  q i 8 * t + !      // Store in timestamp buffer
  e i u + !          // Store event type
  h v ? 1 + h v ?!    // Increment event counter
  h w ? 1 + 31 % h w ?!  // Update buffer index
;

:P                     // Print timestamp
  i!                  // Store index
  i 8 * t + n!        // Get timestamp address
  n 6 + ? `.` n 5 + ? `.`  // Year.Month.
  n 4 + ? ` `              // Date
  n 3 + ? `:` n 2 + ? `:` // Hours:Minutes:
  n 1 + ? `.` n ? /N      // Seconds
;

:D                     // Dump all timestamps for pin
  p!                  // Store pin number
  p H h!              // Get hash
  h v ? 0 > (         // If events exist
    `Events for pin ` p . `:` /N
    h v ? 32 > (      // If more than buffer size
      32             // Limit to buffer size
    ) /E (
      h v ?          // Use actual count
    ) n!
    n (              // For each event
      /i p P         // Print timestamp
      /i u + ? e!    // Get event type
      ` Type: ` e . /N
    )
  )
;

:L                     // Log pin change
  s! p!               // Store pin and state
  p H h!              // Get hash
  s /T = (            // If rising edge
    1 p C            // Log as rise event
  ) /E (
    2 p C            // Log as fall event
  )
;

:Z                     // Interrupt handler
  8 (                // For each pin
    /i H h!          // Get hash
    /i #80 /I s!     // Read pin
    s h c ? = /F = ( // If state changed
      s h c ?!       // Update state
      s /i L         // Log change
    )
  )
;

// Event filtering and analysis
:F                     // Filter events by type
  t! p!               // Store pin and type
  p H h!              // Get hash
  `Events type ` t . ` for pin ` p . `:` /N
  32 (               // Check buffer
    /i u + ? t = (   // If matching type
      /i p P         // Print timestamp
    )
  )
;

:A                     // Analyze time between events
  p!                  // Store pin
  p H h!              // Get hash
  `Time analysis for pin ` p . `:` /N
  0 a! 0 x!          // Clear min/max
  h v ? 1 - (        // For each pair
    /i 8 * t + n!    // Get first timestamp
    /i 1 + 8 * t + m! // Get next timestamp
    // Calculate difference in seconds
    m ? n ? -        // Seconds
    m 1 + ? n 1 + ? - 60 * + // Minutes
    m 2 + ? n 2 + ? - 3600 * + d!
    d a + a!         // Add to total
    d x > (          // Check max
      d x!
    )
    d y < (          // Check min
      d y!
    )
  )
  `Min: ` y .
  ` Max: ` x .
  ` Avg: ` a h v ? 1 - / . /N
;

// Example usage:
// I              // Initialize
// S              // Setup RTC
// T              // Read current time
// 2 1 L          // Log rising edge on pin 2
// 2 D            // Show all events for pin 2
// 2 1 F          // Show rising edges for pin 2
// 2 A            // Analyze timing for pin 2

// Time setting helper
:E                     // Set RTC time
  #80 #0E W           // Enable write
  s 0 W               // Set seconds
  m 1 W               // Set minutes
  h 2 W               // Set hours
  w 3 W               // Set day
  d 4 W               // Set date
  n 5 W               // Set month
  y 6 W               // Set year
  #7F #0E W           // Disable write
;

// Pattern detection with timestamps
:M                     // Monitor for time pattern
  d! p!               // Store pin and duration
  p H h!              // Get hash
  T t!                // Get start time
  /U (                // Loop forever
    p #80 /I s!      // Read pin
    s h c ? = /F = ( // If changed
      T t - d > (    // If duration exceeded
        /B           // Break loop
      )
      s h c ?!       // Update state
      s p L          // Log change
    )
  )
;

// Statistical functions
:B                     // Calculate time between edges
  p!                  // Store pin
  p H h!              // Get hash
  [ 0 0 0 0 0 0 0 0 ] r!  // Create results array
  h v ? 1 - (         // For each pair of events
    /i 8 * t + n!     // Get first timestamp
    /i 1 + 8 * t + m! // Get next timestamp
    // Calculate difference
    m ? n ? -         // Seconds diff
    r /i ?!           // Store in results
  )
  r                   // Return results array
;

```

This timestamping system provides:

1. Core Features:
   - SPI interface to RTC
   - Accurate time tracking
   - Event logging
   - Circular buffer storage

2. Key Functions:
   - T: Read current time
   - C: Create timestamp
   - L: Log pin change
   - D: Dump timestamps
   - F: Filter events
   - A: Analyze timing

Example Usage:

```mint
// Basic setup
I               // Initialize
S               // Setup RTC

// Event logging
2 1 L           // Log rising edge on pin 2
2 0 L           // Log falling edge on pin 2

// Analysis
2 D             // Show all events
2 1 F           // Show rising edges
2 A             // Analyze timing
```

Features:

1. Time Management:
   - Real-time clock interface
   - Time setting/reading
   - Date tracking
   - Millisecond resolution

2. Event Logging:
   - Edge detection
   - Event types
   - Circular buffer
   - Timestamp storage

3. Analysis Tools:
   - Event filtering
   - Timing analysis
   - Pattern detection
   - Statistical functions

Applications:

1. Data Logging:
   - Event recording
   - Signal timing
   - State changes
   - Duration measurement

2. Monitoring:
   - System events
   - Pin changes
   - Time patterns
   - Sequence detection

3. Debugging:
   - Event timing
   - State changes
   - Signal analysis
   - Pattern verification

Example Operations:

1. Set Time:
```mint
// Set to 12:30:00 on Jan 1, 2024
0 30 12 1 1 1 24 E
```

2. Monitor Events:
```mint
2 1000 M        // Monitor pin 2 for 1 second
```

3. Analyze Patterns:
```mint
2 B             // Calculate edge timing
```

Potential Enhancements:

1. Temperature compensation
2. Alarm functions
3. Calendar events
4. Time synchronization
5. Event categorization
6. Pattern matching


//////////////////////////////////////////////////////////////////
## Adding edge-triggered interrupts

Some approaches for detecting and analyzing different trigger types:

Break down on the types of triggers we need to detect:

1. Edge Triggers:
- Rising edge (transition from low to high)
- Falling edge (transition from high to low)
- Both edges (either transition)

2. Level Triggers:
- High level (signal stays high)
- Low level (signal stays low)

Basic algorithm approach:

```
For edge detection:
1. Store previous state of pin
2. Read current state of pin
3. Compare states:
   - Rising edge = (prev=0 AND current=1)
   - Falling edge = (prev=1 AND current=0)

For level detection:
1. Read current state
2. Apply timeout/debounce:
   - High level = signal stays at 1 for X time
   - Low level = signal stays at 0 for X time

For practical implementation:
- Sample port at regular intervals
- Use debounce timing to avoid noise
- Consider setup time and hold time requirements
```

An enhanced edge-triggered interrupt system that supports multiple edge types, priority levels, and sophisticated interrupt handling.

```mint
// Edge-Triggered Interrupt System
// Uses array c for edge types (0=none, 1=rising, 2=falling, 3=both)
// Uses array d for interrupt priorities (0=highest)
// Uses array e for debounce timers
// Uses array f for interrupt counters
// Uses array g for last trigger times
// Uses array h for interrupt queue
// Uses array i for interrupt masks

:I                     // Initialize interrupt system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] c!    // Edge types
  [ 0 0 0 0 0 0 0 0 ] d!    // Priorities
  [ 50 50 50 50            // Default 50ms debounce
    50 50 50 50 ] e!
  [ 0 0 0 0 0 0 0 0 ] f!    // Counters
  [ 0 0 0 0 0 0 0 0 ] g!    // Trigger times
  [ 0 0 0 0 0 0 0 0 ] h!    // Queue (4 events per pin)
  [ /T /T /T /T /T /T /T /T ] i!  // Interrupt masks
;

:T                     // Get current timestamp
  #FF /I              // Read timer port
;

:E                     // Enable edge interrupt
  p! t! e!            // Store pin, type, edge
  p H h!              // Get hash
  e h c ?!            // Store edge type
  t h d ?!            // Store priority
  /T h i ?!           // Enable interrupts
  0 h f ?!            // Clear counter
  T h g ?!            // Store initial time
;

:D                     // Disable interrupts for pin
  p!                  // Store pin
  p H h!              // Get hash
  0 h c ?!            // Clear edge type
  /F h i ?!           // Disable interrupts
;

:B                     // Set debounce time
  t! p!               // Store pin and time
  p H h!              // Get hash
  t h e ?!            // Store debounce time
;

:Q                     // Add to interrupt queue
  v! p!               // Store pin and value
  p H h!              // Get hash
  h h ? 4 { #0F &     // Shift queue left
  v |                 // Add new value
  h h ?!              // Store updated queue
;

:P                     // Process interrupt by priority
  p!                  // Store pin
  p H h!              // Get hash
  h d ? 0 = (         // Priority 0 (highest)
    p A              // Call handler A
  )
  h d ? 1 = (         // Priority 1
    p B              // Call handler B
  )
  h d ? 2 = (         // Priority 2
    p C              // Call handler C
  )
;

:C                     // Check debounce
  p!                  // Store pin
  p H h!              // Get hash
  T h g ? -          // Time since last interrupt
  h e ? >            // Compare with debounce time
;

:Z                     // Main interrupt handler
  8 (                // For each pin
    /i H h!          // Get hash
    h i ? /T = (     // If interrupts enabled
      /i #80 /I v!   // Read current value
      h c ? 1 = v /T = & ( // Rising edge
        /i C (        // If debounce expired
          v /i Q      // Queue the event
          T h g ?!    // Update timestamp
          h f ? 1 + h f ?!  // Increment counter
          /i P        // Process interrupt
        )
      )
      h c ? 2 = v /F = & ( // Falling edge
        /i C (        // If debounce expired
          v /i Q      // Queue the event
          T h g ?!    // Update timestamp
          h f ? 1 + h f ?!  // Increment counter
          /i P        // Process interrupt
        )
      )
      h c ? 3 = (    // Both edges
        /i C (        // If debounce expired
          v /i Q      // Queue the event
          T h g ?!    // Update timestamp
          h f ? 1 + h f ?!  // Increment counter
          /i P        // Process interrupt
        )
      )
    )
  )
;

// Example interrupt handlers
:A                     // High priority handler
  `High priority interrupt on pin ` p .
  ` Value: ` p #80 /I .
  ` Count: ` p H h! h f ? . /N
;

:B                     // Medium priority handler
  `Medium priority interrupt on pin ` p .
  ` Queue: ` p H h! h h ? , /N
;

:C                     // Low priority handler
  `Low priority interrupt on pin ` p .
  ` Time: ` T . /N
;

:S                     // Show interrupt status
  `Interrupt Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h c ? 0 > (      // If interrupts configured
      `Pin ` /i .
      ` Type:` h c ? .
      ` Pri:` h d ? .
      ` Count:` h f ? .
      ` Queue:` h h ? , /N
    )
  )
;

// Advanced edge pattern detection
:M                     // Monitor for edge pattern
  n! p!               // Store pin and pattern
  p H h!              // Get hash
  0 t!               // Clear timer
  /U (               // Loop until pattern found
    h h ? n = (      // If pattern matches
      `Pattern found after ` t . `ms` /N
      /B            // Break loop
    )
    1 t + t!        // Increment timer
  )
;

// Edge timing analysis
:Y                     // Analyze edge timing
  p!                  // Store pin
  p H h!              // Get hash
  0 m! 0 x!          // Clear min/max
  h f ? 1 > (        // If multiple interrupts
    [ h f ? 1 - (    // Create timing array
      /i 1 + g ? /i g ? - t!  // Calculate interval
      t              // Store timing
    ) ] q!           // Save to array
    // Calculate statistics
    q /S (           // For each timing
      /i q ? s!      // Get timing
      s m < (        // Update min
        s m!
      )
      s x > (        // Update max
        s x!
      )
    )
    `Edge Timing Analysis:` /N
    `Min: ` m .
    ` Max: ` x .
    ` Avg: ` q 0 ?
    q /S 1 - (       // Sum remaining
      /i 1 + q ? +
    )
    q /S / . /N      // Calculate average
  )
;

// Pattern matching with edge timing
:N                     // Define timing pattern
  t! p! n!            // Store pattern, pin, timing
  p H h!              // Get hash
  [ n t ] r!          // Store pattern and timing
  /U (               // Monitor continuously
    h h ? n = (      // If pattern matches
      T h g ? - t = (  // If timing matches
        `Pattern matched` /N
        p P          // Process interrupt
      )
    )
  )
;

// Interrupt masking control
:K                     // Mask interrupts
  m! p!               // Store pin and mask
  p H h!              // Get hash
  m h i ?!            // Store mask
;

// Example usage:
// I              // Initialize
// 2 1 1 E        // Pin 2: Rising edge, high priority
// 3 2 2 E        // Pin 3: Falling edge, medium priority
// 4 3 0 E        // Pin 4: Both edges, highest priority
// 2 100 B        // Set 100ms debounce on pin 2
// S              // Show status
// 2 #0101 M      // Monitor pin 2 for pattern
// 2 Y            // Analyze edge timing
// 2 /F K         // Mask interrupts on pin 2

// Real-world example: Button debounce
:R                     // Setup button with debounce
  p!                  // Store pin
  p 1 0 E            // Rising edge, highest priority
  p 50 B             // 50ms debounce
  `Button on pin ` p .
  ` configured` /N
;

// Example: Edge counter with timeout
:O                     // Count edges with timeout
  t! p!               // Store pin and timeout
  p 3 1 E            // Both edges, high priority
  0 c!               // Clear counter
  T s!               // Start time
  /U (               // Loop
    T s - t > /B     // Break if timeout
    p H h! h f ? c!  // Get count
    `Count: ` c . /N // Show count
    100 @            // Wait 100ms
  )
  p D                // Disable interrupts
;

// Example: Edge sequence detector
:X                     // Detect edge sequence
  p!                  // Store pin
  p 3 0 E            // Both edges, highest priority
  [ 1 0 1 1 ] q!     // Define sequence
  /U (               // Monitor continuously
    p H h! h h ? #0F & // Get last 4 edges
    q 0 ? q 1 ? q 2 ? q 3 ? | | | = ( // Compare sequence
      `Sequence detected` /N
      500 @          // Wait before restarting
    )
  )
;

```

This edge-triggered interrupt system provides:

1. Core Features:
   - Multiple edge types
   - Priority levels
   - Debouncing
   - Queue system
   - Masking control

2. Key Functions:
   - E: Enable interrupts
   - D: Disable interrupts
   - B: Set debounce
   - K: Mask interrupts
   - Y: Analyze timing
   - M: Pattern monitoring

Example Usage:

```mint
// Basic setup
I               // Initialize
2 1 1 E         // Pin 2: Rising edge, high priority
2 50 B          // 50ms debounce

// Pattern detection
2 #0101 M       // Monitor for pattern
2 Y             // Analyze timing

// Button setup
2 R             // Configure button
2 1000 O        // Count edges for 1 second
```

Advanced Features:

1. Edge Detection:
   - Rising edge
   - Falling edge
   - Both edges
   - Pattern matching

2. Interrupt Management:
   - Priority levels
   - Queuing
   - Masking
   - Debouncing

3. Analysis Tools:
   - Timing analysis
   - Pattern detection
   - Sequence matching
   - Event counting

Applications:

1. Button Handling:
   - Debounced input
   - Multi-click detection
   - Long press detection
   - Combo detection

2. Sensor Monitoring:
   - Edge counting
   - Timing measurement
   - Pattern detection
   - Event sequencing

3. Protocol Decoding:
   - Bit patterns
   - Timing verification
   - Sequence detection
   - Error checking

Example Patterns:

1. Button Click Counter:
```mint
2 R             // Setup button
2 1000 O        // Count clicks for 1 second
```

2. Edge Sequence Detection:
```mint
2 X             // Start sequence detector
```

3. Timing Analysis:
```mint
2 Y             // Analyze edge timing
```

Potential Enhancements:

1. Nested interrupts
2. Edge counting with rotation
3. Complex pattern matching
4. Time-based patterns
5. Multi-pin patterns
6. Interrupt chaining


//////////////////////////////////////////////////////////////////
## Implementing pattern detection

A pattern detection system that can recognize various types of pin patterns including timing patterns, sequential patterns, and multi-pin combinations.

```mint
// Pin Pattern Detection System
// Uses array p for pattern definitions
// Uses array q for pattern timings
// Uses array r for current state buffers
// Uses array s for pattern match counters
// Uses array t for timing windows
// Uses array u for multi-pin patterns
// Uses array v for pattern flags

:I                     // Initialize pattern system
  [ 0 0 0 0 0 0 0 0 ] a!    // States/modes array
  [ /F /F /F /F /F /F /F /F ] b!  // Pin tracking array
  [ 0 0 0 0 0 0 0 0 ] p!    // Pattern definitions
  [ 0 0 0 0 0 0 0 0 ] q!    // Pattern timings
  [ 0 0 0 0 0 0 0 0 ] r!    // State buffers
  [ 0 0 0 0 0 0 0 0 ] s!    // Match counters
  [ 100 100 100 100        // Default 100ms window
    100 100 100 100 ] t!
  [ 0 0 0 0 0 0 0 0 ] u!    // Multi-pin patterns
  [ /F /F /F /F /F /F /F /F ] v!  // Pattern flags
;

:T                     // Get current timestamp
  #FF /I              // Read timer port
;

:C                     // Update state buffer
  s! p!               // Store pin and state
  p H h!              // Get hash
  h r ? 4 { #0F &     // Shift buffer left
  s |                 // Add new state
  h r ?!              // Store updated buffer
;

:P                     // Define simple pattern
  n! p!               // Store pin and pattern
  p H h!              // Get hash
  n h p ?!            // Store pattern
  100 h t ?!          // Default timing window
  /T h v ?!           // Enable pattern detection
;

:W                     // Set timing window
  w! p!               // Store pin and window
  p H h!              // Get hash
  w h t ?!            // Store window
;

:M                     // Define timed pattern
  t! n! p!            // Store pin, pattern, timing
  p H h!              // Get hash
  n h p ?!            // Store pattern
  t h q ?!            // Store timing
  /T h v ?!           // Enable pattern detection
;

:B                     // Define multi-pin pattern
  m! n! p!            // Store pins, pattern, mask
  p H h!              // Get hash
  n h p ?!            // Store pattern
  m h u ?!            // Store pin mask
  /T h v ?!           // Enable pattern detection
;

:D                     // Check pattern match
  p!                  // Store pin
  p H h!              // Get hash
  h r ? #0F &         // Get current buffer
  h p ? = (           // If pattern matches
    h s ? 1 + h s ?!  // Increment counter
    p A              // Call match handler
  )
;

:E                     // Check timed pattern
  p!                  // Store pin
  p H h!              // Get hash
  h r ? #0F &         // Get current buffer
  h p ? = (           // If pattern matches
    T y!             // Store current time
    y h q ? - z!     // Calculate elapsed time
    z h t ? <= (     // If within window
      h s ? 1 + h s ?!  // Increment counter
      p B            // Call timed handler
    )
  )
;

:F                     // Check multi-pin pattern
  p!                  // Store pin
  p H h!              // Get hash
  0 m!               // Clear match mask
  h u ? n!           // Get pin mask
  8 (                // Check each pin
    n #01 & (        // If pin in mask
      /i #80 /I #01 & m { m! // Build state
    )
    n 1 } n!        // Shift mask
  )
  m h p ? = (        // If pattern matches
    h s ? 1 + h s ?!  // Increment counter
    p C              // Call multi-pin handler
  )
;

// Example pattern handlers
:A                     // Simple pattern handler
  `Pattern match on pin ` p .
  ` count: ` p H h! h s ? . /N
;

:B                     // Timed pattern handler
  `Timed pattern on pin ` p .
  ` in ` z . `ms` /N
;

:C                     // Multi-pin pattern handler
  `Multi-pin pattern match pin ` p .
  ` mask: ` p H h! h u ? , /N
;

:Z                     // Main interrupt handler
  8 (                // For each pin
    /i H h!          // Get hash
    h v ? /T = (     // If pattern detection enabled
      /i #80 /I v!   // Read current value
      v /i C         // Update buffer
      h u ? 0 = (    // Single pin pattern
        h q ? 0 = (  // Simple pattern
          /i D       // Check pattern
        ) /E (       // Timed pattern
          /i E       // Check timed pattern
        )
      ) /E (         // Multi-pin pattern
        /i F         // Check multi-pin pattern
      )
    )
  )
;

// Pattern library examples
:L                     // Double click pattern
  p!                  // Store pin
  #0101 p P          // Set pattern (up-down-up-down)
  p 500 W            // 500ms window
;

:N                     // Long press pattern
  p!                  // Store pin
  #0011 p P          // Set pattern (up-up-down-down)
  p 1000 W           // 1s window
;

:O                     // Triple click pattern
  p!                  // Store pin
  #010101 p P        // Set pattern
  p 750 W            // 750ms window
;

:Q                     // Alternating pins pattern
  p!                  // Store pin
  #0A0A p P          // Alternating pattern
  #03 p B            // Pins 0 and 1
;

:S                     // Show pattern status
  `Pattern Status:` /N
  8 (                // For each pin
    /i H h!          // Get hash
    h v ? /T = (     // If enabled
      `Pin ` /i .
      ` Pattern: ` h p ? ,
      ` Count: ` h s ? .
      h q ? 0 > (    // If timed
        ` Window: ` h t ? .
      )
      h u ? 0 > (    // If multi-pin
        ` Mask: ` h u ? ,
      )
      /N
    )
  )
;

// Advanced pattern functions
:X                     // Complex sequence detector
  p!                  // Store pin
  [                  // Create sequence array
    #01 #02 #04 #08  // Rising sequence
  ] q!
  4 (                // Check each part
    /i q ? p P       // Set pattern
    p 250 W          // 250ms per part
    /U (             // Wait for match
      p D            // Check pattern
      h s ? 0 > /B   // Break if matched
    )
  )
;

:Y                     // Rhythm pattern detector
  p!                  // Store pin
  [ 100 200 100 400 ] t!  // Rhythm timing array
  #0101 p P          // Set basic pattern
  /U (               // Monitor continuously
    p H h!           // Get hash
    h r ? #0F &      // Get current pattern
    4 (              // Check each timing
      /i t ? @       // Wait timing
      p #80 /I C     // Sample and update
    )
    h p ? = (        // If pattern matches
      `Rhythm matched` /N
    )
  )
;

// Example usage:
// I              // Initialize
// 2 L            // Setup double click on pin 2
// 3 N            // Setup long press on pin 3
// 4 O            // Setup triple click on pin 4
// 2 3 Q          // Setup alternating pattern on pins 2,3
// S              // Show status
// 2 X            // Start complex sequence detector
// 2 Y            // Start rhythm detector

// Morse code pattern detector
:R                     // Define morse code pattern
  p!                  // Store pin
  [ // Define timing patterns for dots and dashes
    100 100          // Dot timing (on/off)
    300 100          // Dash timing (on/off)
  ] m!
  /U (               // Monitor continuously
    p #80 /I v!      // Read pin
    v /T = (         // If high
      T t!           // Start timing
      /U (           // Wait for low
        p #80 /I /T = /W
      )
      T t - y!       // Calculate duration
      y 200 > (      // If long press
        `Dash` /N    // Dash detected
      ) /E (
        `Dot` /N     // Dot detected
      )
    )
    100 @           // Wait between samples
  )
;

```

This pattern detection system provides:

1. Core Features:
   - Simple patterns
   - Timed patterns
   - Multi-pin patterns
   - Rhythm detection
   - Sequence detection

2. Key Functions:
   - P: Define pattern
   - W: Set timing window
   - M: Define timed pattern
   - B: Define multi-pin pattern
   - X: Complex sequence
   - Y: Rhythm detection

Example Usage:

```mint
// Basic patterns
I               // Initialize
2 L             // Double click detection
3 N             // Long press detection
4 O             // Triple click detection

// Advanced patterns
2 3 Q           // Alternating pin pattern
2 X             // Complex sequence
2 Y             // Rhythm detection
2 R             // Morse code detection
```

Pattern Types:

1. Simple Patterns:
   - Click sequences
   - State changes
   - Basic combinations

2. Timed Patterns:
   - Double clicks
   - Long press
   - Hold-release

3. Multi-Pin Patterns:
   - Pin combinations
   - Alternating pins
   - Synchronized changes

Applications:

1. User Interface:
   - Button combinations
   - Gesture detection
   - Secret knocks
   - Command input

2. Protocol Decoding:
   - Morse code
   - Serial data
   - Custom protocols
   - Timing verification

3. Sensor Analysis:
   - Motion patterns
   - Trigger sequences
   - Event detection
   - Timing analysis

Example Patterns:

1. Button Patterns:
```mint
2 L             // Double click
2 N             // Long press
2 O             // Triple click
```

2. Advanced Patterns:
```mint
2 X             // Complex sequence
2 Y             // Rhythm detection
2 R             // Morse code
```

Features:

1. Pattern Matching:
   - Exact matching
   - Timing windows
   - Multi-pin combinations
   - Sequence detection

2. Timing Control:
   - Window settings
   - Rhythm detection
   - Sequence timing
   - Pattern timing

3. Analysis Tools:
   - Pattern counting
   - Timing analysis
   - Match detection
   - Status monitoring


//////////////////////////////////////////////////////////////////

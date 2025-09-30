# Attribution Implementation for Presence Simulation

## Overview
This implementation adds proper attribution to state changes made by the Presence Simulation component, allowing users to see in the logbook and history that changes were made by the "Presence Simulation" system user rather than appearing as anonymous changes.

## Changes Made

### 1. System User Creation
- Added a system user named "Presence Simulation" that is created during component setup
- The system user is created with admin privileges to ensure it can control all necessary entities
- User is stored in `hass.data[DOMAIN]["system_user"]` for access throughout the component

### 2. Context Attribution 
- All service calls now include a `Context` object with the system user ID
- This applies to all entity control operations including:
  - Light controls (turn_on/turn_off with brightness and color attributes)
  - Cover controls (open/close/position/tilt)
  - Media player controls (play/stop)
  - Input select controls
  - Generic homeassistant turn_on/turn_off commands
  - Scene creation and restoration

### 3. System User Management
- System user is reused if it already exists (checked by name and system_generated flag)
- Proper cleanup when component is removed - system user is deleted if no other presence simulation entries exist
- Graceful error handling if user creation or removal fails

## Technical Details

### Code Changes
1. **Import Added**: `from homeassistant.core import Context`
2. **System User Creation**: Added in `async_setup_entry()`
3. **Context Usage**: All `hass.services.async_call()` calls now include `context=context` parameter
4. **Cleanup**: Enhanced `async_remove_entry()` to remove system user when appropriate

### Key Functions Modified
- `async_setup_entry()`: System user creation and storage
- `update_entity()`: Context creation and usage for all service calls
- `stop_presence_simulation()`: Context usage for scene restoration
- `handle_presence_simulation()`: Context usage for scene creation
- `async_remove_entry()`: System user cleanup

## Benefits
1. **Clear Attribution**: Logbook entries will show "Presence Simulation" as the cause of state changes
2. **Better Debugging**: Easier to identify which changes were made by the simulation
3. **Audit Trail**: Proper user context for security and tracking purposes
4. **Home Assistant Best Practices**: Follows recommended patterns for system integrations

## Usage
After installing this updated version, all state changes made by the presence simulation will appear in the logbook with "Presence Simulation" as the user context, making it clear why entities changed state during simulation periods.

## Testing
To verify the implementation is working:
1. Start a presence simulation
2. Check the logbook during simulation 
3. Entity state changes should show "Presence Simulation" as the initiator
4. Check Developer Tools > Services to see the system user exists
5. Remove the integration and verify the system user is cleaned up
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Delphi Event Bus (DEB) is a publish/subscribe framework for Delphi that enables decoupled communication between application components through events and channels. It supports both interface-typed events and named-channel messages with flexible thread delivery modes.

## Architecture

### Core Components

The framework consists of four main source files in the `/source` directory:

- **EventBus.pas**: Public API interfaces (`IEventBus`) and attribute definitions
- **EventBus.Core.pas**: Core implementation including `TEventBus` and the global event bus factory
- **EventBus.Subscribers.pas**: Subscriber method encapsulation and subscription management
- **EventBus.Helpers.pas**: RTTI utilities and interface type caching for performance

### Key Design Patterns

1. **Two Event Types**:
   - Interface-typed events: User-defined interfaces posted with optional context strings
   - Named-channel messages: String messages routed by channel name

2. **Thread Modes**: Subscribers can specify delivery in:
   - Main thread (`TThreadMode.Main`)
   - Posting thread (`TThreadMode.Posting`)
   - Background thread (`TThreadMode.Background`)
   - Async mode (`TThreadMode.Async`)

3. **Attribute-Based Subscriptions**:
   - `[Subscribe]` attribute for interface events
   - `[Channel('name')]` attribute for channel messages
   - Methods must have exactly one parameter (interface or string)

4. **Global Instance**: `GlobalEventBus` provides a default singleton instance

## Building and Testing

### Compilation

To compile the project using the compiler-agent:
```delphi
// Use the "compiler-agent" for all Delphi compilation tasks
```

### Running Tests

The project uses DUnitX testing framework. Test files are in `/tests`:
- Main test project: `DEBDUnitXTests.dpr`
- Test project file: `DEBDUnitXTests.dproj`

Tests can be run in three modes:
1. **Console mode** (default): Define `CONSOLE_TESTRUNNER`
2. **GUI mode**: Define `GUI_TESTRUNNER`
3. **TestInsight mode**: Define `TESTINSIGHT`

## Sample Applications

Four sample applications demonstrate different use cases:

1. **AccessRemoteData**: Remote data access patterns
2. **Analytics**: Event analytics and logging
3. **WeatherStation**: Multiple forms communicating via events (FMX)
4. **VCLMessaging**: VCL forms messaging example

## Development Guidelines

### Event Definition
```delphi
IMyEvent = interface(IInterface)
  ['{GUID}']
  // Event properties/methods
end;
```

### Subscriber Implementation
```delphi
[Subscribe(TThreadMode.Main)]
procedure OnMyEvent(AEvent: IMyEvent);

[Channel('MY_CHANNEL')]
procedure OnChannelMessage(AMsg: string);
```

### Registration Pattern
```delphi
// Register for events
GlobalEventBus.RegisterSubscriberForEvents(Self);

// Register for channels
GlobalEventBus.RegisterSubscriberForChannels(Self);

// Always unregister in destructor
GlobalEventBus.UnregisterForEvents(Self);
```

### Thread Safety

- The framework is thread-safe using `TMultiReadExclusiveWriteSynchronizer`
- Background tasks use a dedicated thread pool (Delphi 28.0+)
- Subscriber methods should handle their own synchronization if needed

## Important Notes

- Framework requires Delphi 2010 or higher
- Interface-based events (v2.0+) replaced the older TObject-based approach
- Methods with invalid signatures raise `EInvalidSubscriberMethod`
- The `TInterfaceHelper` caches RTTI for performance - call `RefreshCache` after loading packages dynamically
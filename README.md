# Synapse

**A modular C#/.NET desktop application for collaborative sessions and lab monitoring.**

Synapse brings messaging, a shared whiteboard, screen sharing, file comparison, cloud storage and tool updates into one Windows application. Users can create or join a session and move between these capabilities through a shared dashboard.

This is a **collaborative college software engineering project**. The complete application was developed by the project team. My contribution focused on the **Chat/Content module's MVVM application logic, state management, integration and unit testing**.

[Original team repository](https://github.com/GroupProjectSE2024/SoftwareEngineering2024) · [My portfolio](https://satyadev-subudhi-profile.vercel.app/) · [LinkedIn](https://www.linkedin.com/in/satyadev-subudhi-365325244/)

## Application overview

| Capability | What it provides | Source |
|---|---|---|
| Session dashboard | Create/join flows, participant information and session status | [Dashboard](Dashboard/) and [desktop views](UXModule/Views/) |
| Chat and messaging | Public and private messages, dynamic message search, matched-text highlighting, deletion and emotes | [Content](Content/) |
| Shared whiteboard | Drawing and text tools, shape manipulation, snapshots and undo/redo | [WhiteboardGUI](WhiteboardGUI/) |
| Screen sharing | Client-side capture and processing, with server-side handling and display of shared screens | [Screenshare](Screenshare/) |
| File comparison | Directory/file metadata comparison and summary generation | [FileCloner](FileCloner/) |
| Cloud storage | Upload, download, list, update and delete operations through a cloud service and serverless API | [Cloud Module](Cloud%20Module/) |
| Tool updates and loading | Tool metadata comparison, updates and assembly-loading support for analysis tools | [Updater](Updater/), [ViewModel](ViewModel/) and [ToolInterface](ToolInterface/) |
| Shared communication | Networking infrastructure used by application modules | [Networking](Networking/) |

These are application-level capabilities implemented across the team. They are separate from my individual contribution described below.

## My contribution: Chat/Content module

My work focused on the behaviour behind messaging: processing actions, maintaining application state and connecting the chat module to the wider application. Frontend layout and the underlying networking module were outside my implementation scope.

### Dynamic message search

I implemented the ViewModel search logic behind the application's search-as-you-type interaction. The UI forwards changes in the search query to the ViewModel, which updates the results and exposes matching text for highlighting.

- Case-insensitive matching against message content.
- An observable result collection that the view can display.
- Separate properties for the text before, within and after a match.
- Restoration of original message text when leaving search.

The goal was to make earlier messages easy to find while keeping search behaviour explicit in the application logic.

**Code:** [MainViewModel.cs](Content/ChatViewModel/MainViewModel.cs)  
**UI integration reference:** [ChatPage.xaml.cs](UXModule/Views/ChatPage.xaml.cs)

### Public/private messaging and emotes

I worked on the messaging functionality supporting public broadcasts, messages to a selected recipient, message deletion and emote handling. These interactions required coordinating message content, recipient selection and state updates within the chat module.

**Related module code:** [ChatClient.cs](Content/ChatClient.cs), [ChatServer.cs](Content/ChatServer.cs) and [ChatMessage.cs](Content/ChatMessage.cs)

### MVVM, state management and concurrency

- Used MVVM commands and property-change notifications to connect application logic with the view.
- Worked with observable collections for messages and search results.
- Implemented multithreading and synchronization locks to coordinate access to shared application state, including client-list updates.
- Contributed to module integration and unit testing.

The search and messaging work gave me practical experience with the relationship between user interactions, changing state and concurrent application behaviour.

### Testing

The repository contains chat tests for message state, property-change notifications, deletion, case-insensitive search, highlighting and restoration of original content.

**Start here:** [MainViewModelTests.cs](TestProject/Content/MainViewModelTests.cs) and [Content tests](TestProject/Content/).

## How the application works

1. **Sign in and enter a session.** The desktop application includes a Google OAuth sign-in flow and options to create or join a session.
2. **Establish the session connection.** Host/server and client components use the networking layer to exchange module data. Session information and participants appear in the dashboard.
3. **Choose a collaboration tool.** Users access messaging, whiteboard, screen sharing, file comparison and update/tool-loading interfaces from the application shell.
4. **Process actions through module logic.** ViewModels and module services handle commands and update the state consumed by desktop views. The exact division varies by module.
5. **Use cloud operations where configured.** The cloud client calls serverless endpoints for storage operations; those features depend on the required service configuration.

For the chat module, the interaction path is:

```text
User action in the chat view
        |
        v
Chat ViewModel: commands, message state and search results
        |
        +--> Local search/highlighting and property updates
        |
        +--> Chat client/server logic --> Shared networking layer
```

Search is handled against the messages available to the ViewModel. Sending messages involves the chat client/server components and the shared networking infrastructure.

## Technology stack

| Area | Technologies and patterns |
|---|---|
| Desktop application | C#, .NET 8, Windows Presentation Foundation (WPF), XAML |
| Application structure | MVVM, commands, observable collections, property-change notifications |
| Concurrency | Multithreading and synchronization locks |
| Communication | Client-server architecture; networking project includes gRPC and Protocol Buffers |
| Cloud integration | Azure Blob Storage and serverless HTTP APIs |
| Authentication | Google OAuth integration |
| Testing | MSTest, xUnit, NUnit and Moq are referenced across the test projects |

## Repository structure

```text
Content/               Chat models, client/server logic and ChatViewModel
Dashboard/             Session and participant-related components
Networking/            Shared communication infrastructure
Screenshare/           Capture, processing and screen-sharing components
WhiteboardGUI/         Whiteboard models, services and ViewModels
FileCloner/            File metadata, comparisons and summaries
Cloud Module/          Cloud client library and serverless storage API
Updater/               Tool/file updates and assembly loading
ViewModel/             Dashboard and updater ViewModels
ToolInterface/         Tool interfaces
UXModule/              WPF application shell and views
Synapse/               Windows packaging project
TestProject/           Module tests, including Content tests
TestCases/             Networking tests
UXModule.sln           Integrated Visual Studio solution
```

## Getting started

### Requirements

- Windows, because the desktop application targets `net8.0-windows` and uses WPF.
- .NET 8 SDK.
- Visual Studio with the **.NET desktop development** workload, if using the IDE.
- NuGet access to restore the packages referenced by the projects.
- Working Google OAuth and cloud-service configuration for the features that depend on them.

### Open the application

1. Clone this repository using its GitHub **Code → HTTPS** URL.
2. Open `UXModule.sln` in Visual Studio.
3. Resolve the cloud-library reference described below and restore NuGet packages.
4. Set **`UXModule/UXModule.csproj`** as the startup project.
5. Build and run the desktop application. Use separate application instances to exercise the host/client session flow, with a reachable host address and the appropriate network access.

After resolving the required references and configuration, the equivalent desktop-project commands are:

```powershell
dotnet restore .\UXModule\UXModule.csproj
dotnet build .\UXModule\UXModule.csproj
dotnet run --project .\UXModule\UXModule.csproj
```

### Setup notes for the current source snapshot

- **Use the nested desktop project.** The solution references `UXModule/UXModule.csproj`. The separate root-level `UXModule.csproj` contains references to development folders outside this repository and should not be used as the startup project.
- **Resolve `SECloud.dll`.** The desktop and main test projects reference a root-level `SECloud.dll`, which is absent from the reviewed `master` snapshot. The cloud library source is available at `Cloud Module/SECloud/SECloud/SECloud.csproj`. Build that library and supply its assembly at the referenced location, or replace the binary references with project references before building the dependent projects.
- **Cloud and sign-in require configuration.** Check the OAuth setup and cloud endpoints used by the application. Availability of the original hosted services has not been verified.
- **Packaging is separate from running the desktop application.** The solution also includes a Windows packaging project, which may require additional Visual Studio components and signing configuration.


## Running tests

Once the project references and dependencies are resolved, use Visual Studio Test Explorer or run:

```powershell
dotnet test .\TestProject\TestProject.csproj
dotnet test .\TestCases\TestCases.csproj
```

## Project history and attribution

The original application belongs to the collaborative [SoftwareEngineering2024 team project](https://github.com/GroupProjectSE2024/SoftwareEngineering2024). Contributor history and existing author attribution should remain intact.

This README describes the `master` source snapshot at commit [`7eccd5c`](https://github.com/GroupProjectSE2024/SoftwareEngineering2024/commit/7eccd5cd443679e10c70f7276907954c962497bb). Its tip is newer than the inspected `Package` and dry-run branches; it is the basis for this documentation, not a claim of a verified release.

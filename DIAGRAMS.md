# Architecture Diagrams

## 1. Library Dependencies

```mermaid
graph TD
    exe[gameplayfootball]

    exe --> gamelib
    exe --> hidlib
    exe --> menulib
    exe --> datalib
    exe --> leaguelib
    exe --> blunted2

    gamelib --> blunted2
    hidlib --> blunted2
    menulib --> blunted2
    datalib --> blunted2
    leaguelib --> blunted2

    subgraph blunted2 [blunted2 aggregate]
        baselib
        frameworklib
        scenelib
        systemsgraphicslib
        systemsaudiolib
        systemsphysicslib
        managerslib
        utilslib
        gui2lib
        loaderslib
        typeslib
    end
```

---

## 2. Runtime Boot & Game Loop

```mermaid
flowchart TD
    main["main()"] --> Init["blunted::Initialize(config)\n— create managers\n— register systems\n— set up scenes"]
    Init --> SM["SystemManager\nregisters:\nGraphicsSystem\nAudioSystem\nPhysicsSystem"]
    Init --> TM["TaskManager\nspawns WorkerThreads"]
    Init --> Sched["Scheduler\nregisters TaskSequences"]

    main --> RegisterTasks["register TaskSequences\n(game, menu, graphics, …)"]
    RegisterTasks --> Run["blunted::Run()\nScheduler::Run() loop"]

    Run --> Seq["TaskSequence ticked\nevery N ms"]
    Seq --> Get["IUserTask::GetPhase()\nor ISystemTask StartFrame"]
    Seq --> Process["IUserTask::ProcessPhase()\nor worker thread"]
    Seq --> Put["IUserTask::PutPhase()\nor ISystemTask PutFrame"]

    Get --> GameTask
    Process --> GameTask
    Put --> GameTask

    GameTask --> Match
    GameTask --> MenuTask
```

---

## 3. Manager Singletons

```mermaid
graph LR
    subgraph Managers [Singleton Managers]
        SystemManager
        TaskManager
        SceneManager
        ResourceManagerPool
        EnvironmentManager
        UserEventManager
    end

    SystemManager -->|"registers/owns"| GraphicsSystem
    SystemManager -->|"registers/owns"| AudioSystem
    SystemManager -->|"registers/owns"| PhysicsSystem

    TaskManager -->|"owns pool of"| WorkerThread
    TaskManager -->|"dispatches Command to"| WorkerThread

    SceneManager -->|"holds"| Scene2D
    SceneManager -->|"holds"| Scene3D

    ResourceManagerPool -->|"typed managers for"| Texture
    ResourceManagerPool -->|"typed managers for"| VertexBuffer
    ResourceManagerPool -->|"typed managers for"| Surface
```

---

## 4. Systems Architecture

```mermaid
graph TD
    subgraph Interfaces
        ISystem
        ISystemTask
        ISystemScene
        ISystemObject
    end

    ISystem --> GraphicsSystem
    ISystem --> AudioSystem
    ISystem --> PhysicsSystem

    ISystemTask --> GraphicsTask
    ISystemTask --> AudioTask
    ISystemTask --> PhysicsTask

    ISystemScene --> GraphicsScene
    ISystemScene --> AudioScene
    ISystemScene --> PhysicsScene

    ISystemObject --> GraphicsObject
    ISystemObject --> AudioObject
    ISystemObject --> PhysicsObject

    GraphicsSystem -->|"owns"| GraphicsTask
    GraphicsSystem -->|"owns"| OpenGLRenderer3D["OpenGLRenderer3D\n(IRenderer3D)"]
    GraphicsSystem -->|"creates"| GraphicsScene

    PhysicsSystem -->|"owns"| PhysicsTask
    PhysicsSystem -->|"wraps"| ODEPhysics["ODEPhysics\n(IPhysicsWrapper)"]
```

---

## 5. Scene Graph

```mermaid
graph TD
    IScene --> Scene
    Scene --> Scene2D
    Scene --> Scene3D

    Scene3D -->|"contains"| Node
    Node -->|"children"| Node
    Node -->|"attached"| Geometry
    Node -->|"attached"| Camera
    Node -->|"attached"| Light
    Node -->|"attached"| Sound

    Geometry --> Resource
    Resource --> Texture
    Resource --> VertexBuffer
    Resource --> Surface

    subgraph SceneObjects [Scene Objects: e_ObjectType]
        Geometry
        Camera
        Light
        Sound
    end
```

---

## 6. Match Simulation

```mermaid
graph TD
    GameTask -->|"owns"| Match
    Match -->|"owns 2x"| Team
    Match -->|"owns"| Ball
    Match -->|"owns"| Referee
    Match -->|"owns"| Officials

    Match -->|"holds"| AnimCollection
    Match -->|"holds ring-buffer of"| MentalImage

    Team -->|"owns N"| Player
    Team -->|"owns"| TeamAIController

    Player -->|"extends"| PlayerBase
    PlayerOfficial -->|"extends"| PlayerBase

    PlayerBase -->|"has-a"| HumanoidBase
    HumanoidBase --> Humanoid

    PlayerBase -->|"has-a"| IController
    IController --> PlayerController
    PlayerController --> HumanController
    PlayerController --> ElizaController
    IController --> RefereeController

    ElizaController -->|"selects"| Strategy
    Strategy --> DefaultDefenseStrategy
    Strategy --> DefaultMidfieldStrategy
    Strategy --> DefaultOffenseStrategy
    Strategy --> GoalieDefaultStrategy
    Strategy --> CelebrationStrategy

    Match -->|"reads"| AIfunctions["AIfunctions\n(free functions)"]
    MentalImage -->|"snapshot of"| Match
```

---

## 7. Humanoid Animation

```mermaid
graph TD
    HumanoidBase -->|"extended by"| Humanoid

    HumanoidBase -->|"holds"| AnimCollection
    AnimCollection -->|"holds"| Animation

    HumanoidBase -->|"skeleton of"| Joint
    Joint -->|"skinned by"| WeightedBone
    WeightedBone -->|"deforms"| WeightedVertex

    Humanoid -->|"drives"| Node["scene3d::Node\n(bone nodes)"]
    Humanoid -->|"uploads to"| Geometry["Geometry\n(skinned mesh)"]
```

---

## 8. HID & Input

```mermaid
graph TD
    IHIDevice --> Keyboard
    IHIDevice --> Gamepad

    UserEventManager -->|"polls SDL events"| IHIDevice

    HumanController -->|"reads"| IHIDevice
    GameTask -->|"passes controllers to"| Match
    Match -->|"passes controllers to"| HumanController
```

---

## 9. GUI Class Hierarchy

```mermaid
graph TD
    subgraph Core
        Gui2View --> Gui2Frame
        Gui2Frame --> Gui2Page
        Gui2Frame --> Gui2Dialog
        Gui2Frame --> Gui2Menu
        Gui2View --> Gui2Button
        Gui2View --> Gui2Grid
        Gui2View --> Gui2Image
        Gui2View --> Gui2Text
        Gui2View --> Gui2Slider
        Gui2View --> Gui2Root
    end

    subgraph MainMenuPages
        Gui2Page --> IntroPage
        Gui2Page --> OutroPage
        Gui2Page --> MainMenuPage
        Gui2Page --> ControllerSelectPage
        Gui2Page --> SettingsPage
        Gui2Page --> CreditsPage
        Gui2Page --> TeamSelectPage
    end

    subgraph InGameHUD
        Gui2Page --> GamePage
        Gui2Page --> GameOverPage
        Gui2Page --> PhaseMenuPage
        Gui2View --> Gui2ScoreBoard
        Gui2View --> Gui2Radar
        Gui2View --> Gui2TacticsDebug
        Gui2View --> Gui2PlayerHUD
        Gui2View --> Gui2ReplayMenu
    end

    subgraph LeaguePages
        Gui2Page --> LeaguePage
        Gui2Page --> LeagueCalendarPage
        Gui2Page --> LeagueStandingsPage
        Gui2Page --> LeagueManagementPage
        Gui2Page --> LeagueInboxPage
    end

    IUserTask --> Gui2Task
    Gui2Task --> MenuTask
```

---

## 10. Data Layer

```mermaid
graph TD
    Database["Database (SQLite)"] -->|"loaded into"| MatchData
    Database -->|"loaded into"| PlayerData
    Database -->|"loaded into"| TeamData

    MatchData -->|"passed to"| Match
    PlayerData -->|"passed to"| PlayerBase
    TeamData -->|"passed to"| Team

    subgraph Loaders
        AseLoader["AseLoader (3D models)"]
        ImageLoader["ImageLoader (textures)"]
        WavLoader["WavLoader (audio)"]
    end

    AseLoader -->|"produces"| Geometry
    ImageLoader -->|"produces"| Texture
    WavLoader -->|"produces"| AudioBuffer
```

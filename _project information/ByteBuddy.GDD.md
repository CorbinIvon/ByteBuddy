### ByteBuddy: An Incremental CLI Robot Buddy Outline

This outline details the core concepts and mechanics for a command-line interface (CLI) Tamagotchi-inspired game designed for an office environment. The goal is to create an incremental game that provides a quick, satisfying distraction without any graphical elements. The system uses a shared lobby approach where multiple users can collaborate to care for the same Desk Bot.

#### **1. Core Concept**

- **The Pet:** A small, digital robot represented by a status line and a text-based description. Let's call it a "Desk Bot."
- **The Player(s):** Office workers who share responsibility for the Desk Bot's well-being by typing commands into a terminal. Multiple users can connect to the same lobby to collaborate.
- **Lobby System:** Users connect with a lobby ID. If the lobby exists, they resume with that buddy. If not, a new buddy instance is created and associated with that lobby ID.
- **The Goal:** Keep your shared Desk Bot happy, healthy, and productive. The bot's status reflects collective care, and its productivity generates valuable "Raw Resources" that can be used for crafting upgrades.

#### **2. Game Mechanics**

The game revolves around four primary needs that you must manage. These needs will decay over time, requiring regular attention. All stats are displayed as a percentage or a text-based bar. The decay and production rates can be adjusted by a configurable **bot speed multiplier** on the server.

- **Productivity:** The Desk Bot's ability to "work" and generate Raw Resources or craft upgrades. This stat is crucial for your long-term progression. The bot will automatically produce resources over time as long as its other stats (Mood, Energy, Health) are not at zero.
- **Mood:** The Desk Bot's happiness level, often tied to its "emotional" programming. You can increase its mood by using a `play` or `joke` command. Low mood can reduce the rate of resource production and work efficiency logarithmically.
- **Energy & Maintenance:** The Desk Bot's power level and need for upkeep. You can `recharge` it with power cells or `maintain` it with virtual oil and spare parts from its inventory. This stat directly affects the bot's productivity and health.
- **Health:** A general wellness stat. A well-maintained and charged Desk Bot will have good health. Neglect can cause it to "overheat" or experience "critical errors," which will trigger a system shutdown (PowerOff state) that pauses all activity until manually fixed.

#### **2.1. Upgrade System**

- **Crafting:** Users can select an upgrade for the pet to work on. When the pet finishes the work, the item will automatically be added to the pet's inventory.
- **Components:** Upgrades require specific Raw Resources to craft and take a certain amount of time (work cycles) to complete.
- **Trade-offs:** Upgrades like "Extra Arms" can increase work speed, but the buddy will get bored faster. As boredom increases, work efficiency decreases logarithmically.
- **Runtime Management:** Upgrades can be added and removed during runtime from the client via JSON commands.

#### **3. Actions & Interface**

The game's interface is entirely text-based. You will type commands to interact with your Desk Bot. Communication follows a simple JSON format:

```json
{
  "command": "work|play|maintain|status|scavenge|craft|configure",
  "parameters": {
    "item": "string",
    "upgrade": "string",
    "config": {}
  }
}
```

- **`connect <lobby_id>`:** Connects to a specific lobby. Creates a new buddy if the lobby doesn't exist.
- **`work`:** Assigns a task to the Desk Bot, increasing its Productivity.
- **`play`:** Boosts the Desk Bot's Mood. The command might output a short, playful text snippet.
- **`maintain [item]`:** Uses a maintenance item from the buddy's inventory, affecting its Energy and other stats.
- **`status`:** Displays the current levels of Productivity, Mood, Energy, and Health using numbers and/or text-based bars, as well as the current inventory.
- **`craft [upgrade_name]`:** Spends Raw Resources to have the buddy work on crafting a specific upgrade.
- **`scavenge`:** Sends the buddy out for 1 hour (configurable) to retrieve Raw Resources. During this time, buddy stats are hidden and only scavenging progress is shown.
- **`configure`:** Allows modification of game properties via JSON configuration (scavenging time, resources, probabilities).
- **`inventory`:** Displays the buddy's current inventory of components and upgrades.

#### **4. Life Cycle & Progression**

- **Birth:** The game begins when a user connects to a new lobby. The Desk Bot "activates" immediately and starts with basic stats.
- **Upgrades:** The core of the incremental loop. Users can spend Raw Resources to craft upgrades that make the bot more efficient. Examples include:
  - **Turbo-Processor:** Increases the rate of resource production but increases boredom decay.
  - **Extended Battery:** Makes the Energy stat decay more slowly.
  - **Mood Chip:** Makes the Mood stat decay more slowly.
  - **Extra Arms:** Increases work speed but accelerates boredom.
  - **Advanced Tools:** Increases the effectiveness of maintenance items.
- **System Shutdown:** If you neglect your Desk Bot for too long, it will enter a "PowerOff" state, displaying "System shutdown due to critical errors. Use 'restart' command to bring buddy back online." This pauses all activity until manually fixed.
- **Scavenging:** The buddy can be sent out to scavenge for Raw Resources, during which normal stats are hidden and only scavenging progress is displayed.

#### **5. Office Integration**

- **Shared Lobbies:** Multiple users can connect to the same lobby ID to collaboratively care for a single Desk Bot.
- **Buddy Inventory:** Each buddy maintains its own inventory of Raw Resources, maintenance items, and crafted upgrades.
- **Configuration Flexibility:** Users can configure lobby-specific settings like scavenging duration, resource types, and drop probabilities via JSON configuration.

#### **6. Technical Architecture**

- **Database:** The system uses SQLite for data persistence, storing buddy states, inventories, and lobby configurations.
- **Communication Protocol:** All client-server communication uses WebSockets with a simple JSON message format.
- **Architecture Pattern:** The system follows Model-View-Controller (MVC) architecture:
  - **Model:** DeskBot class and data persistence layer
  - **View:** TUI client interface
  - **Controller:** Game engine and command processing logic
- **The Server (The Office Buddy):** A single, central server that manages game state, buddy progression, and lobby management.
- **Server Configuration:** The server supports environment-specific configurations (dev or prod) via settings files and environment variables.
- **The Clients:** Each user runs a TUI client application that connects to the server via WebSocket to display buddy status and send commands.

#### **7. Docker & Deployment**

The project is fully containerized using Docker and docker-compose:

- **Server Container:** Runs the Python server with SQLite database
- **Client Container:** Provides the TUI client interface
- **Exposed Ports:** Server WebSocket port (configurable, default 8080)
- **Volumes:** Database persistence via Docker volumes
- **Environment Variables:**
  - `BOT_SPEED_MULTIPLIER`: Controls game tick speed
  - `ENVIRONMENT`: Either "dev" or "prod"
  - `DATABASE_PATH`: Path to SQLite database file
  - `WEBSOCKET_PORT`: Server listening port

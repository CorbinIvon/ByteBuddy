Reference [[1. Project Planning]]

The **planning phase** is about understanding the project's goal and requirements. This is where you answer questions like:

**What is the project's purpose?** What problem does it solve?

- A collaborative Tamagotchi-inspired CLI game where multiple users can care for shared digital pets through lobby-based connections.

**Who is the target audience?**

- IT professionals who use a CLI and don't care much for graphics as much as they would story.

**What features are necessary?** What features are just "nice to have"?
Must haves:

- Run on a server and tick every second.
- Server WebSocket with simple JSON message format
- Dynamic Curses TUI Client
- SQLite database for persistence
- Lobby-based buddy sharing system
- MVC architecture implementation
- Crafting and upgrade system
- Scavenging mechanic with configurable timers
- Buddy inventory management
- System shutdown/restart for neglected buddies
  Nice to have:
- Graphical Client

**What technologies and languages will be used?** (e.g., Python, JavaScript, Java, etc.)

- Python for both server and client
- SQLite for database persistence
- WebSockets for real-time communication
- JSON for message formatting
- Docker for containerization

**How will the project be deployed or used?**

- The project will be deployed via docker-compose with separate containers for server and client components. The server will use SQLite for persistence and expose WebSocket connections. Only single environment deployments (either dev or prod) are supported.

**Project Specifications**

1. The server runs on a dedicated host, exposing WebSocket connections on a configurable port. All communication uses a simple JSON message format for commands and responses.
2. Clients establish WebSocket connections using lobby IDs. If a lobby exists, the client resumes with that buddy; if not, a new buddy instance is created.
3. The system follows MVC (Model-View-Controller) architecture with clear separation between game logic, data persistence, and user interface.
4. SQLite database handles all persistence including buddy states, inventories, lobby configurations, and upgrade progress.
5. The game includes crafting, scavenging, and upgrade mechanics with configurable timers and resource management.
6. System shutdown/PowerOff state pauses buddy activity when neglected, requiring manual restart.

---

This stage often involves creating a **project specification document** or a simple list of requirements. Think of it as creating a blueprint for a house before you ever lay a foundation.

---

This initial phase is about defining the project's core purpose and requirements. You'll provide the high-level vision, and the AI will help you refine it.

- **Your Input (Required):**
  - **Project Name:** ByteBuddy
  - **Project Goal:** Tamagotchi inspired game project
  - **Key Features:** A bulleted list of the essential functionalities the project must have.
    - Server + tick + WebSocket + SQLite database
    - TUI Client with MVC architecture
    - Lobby-based buddy sharing system
    - Crafting and upgrade system
    - Scavenging mechanic
    - JSON configuration system
  - **Target Audience:** tui bros
  - **Technical Stack:** List the programming languages, frameworks, and technologies you intend to use (e.g., "Python, Django, SQLite").
    - Python, WebSockets, SQLite, JSON, Docker
- **AI's Role Prompt (Assistance):**
  - The AI will help you brainstorm, identify potential challenges, and suggest features you might have missed. It can act as a sounding board for your ideas.
- **Prompt Example:** "Based on my project plan above, can you identify any potential challenges or missing features?

[[2. Project Outlining]]

Reference [[2. Project Outlining]]

Once the plan is in place, the **outlining phase** focuses on breaking the project down into smaller, manageable pieces. This is where you determine the main components and their relationships. A common way to do this is to create a **high-level design** or **system architecture**. This might include:

- Listing the key **modules** or **classes** needed.
- Defining the major **functions** and **methods** within each module.
- Considering how different parts of the code will interact.

### **1. Server Architecture (MVC Pattern)**

This module handles all core game logic, state management, and communication with clients following MVC architecture.

**Controller Layer:**

- **`Server` Class:** The primary entry point and controller for the back-end.
  - **`__init__`**: Initializes the WebSocket server and instances of the `GameEngine` and `DatabaseManager`.
  - **`start_server`**: Starts the WebSocket listener and periodic timer for game ticks.
  - **`handle_client_connection`**: Asynchronous method that accepts WebSocket connections and processes JSON commands.
  - **`broadcast_update`**: Sends updated buddy state to connected clients after game ticks or commands.
  - **`process_lobby_connection`**: Handles lobby ID connections, creating or resuming buddy instances.

**Model Layer:**

- **`GameEngine` Class:** Core game logic and rule processing.

  - **`__init__`**: Initializes with database reference and configurable `bot_speed_multiplier`.
  - **`game_loop_tick`**: Called every second to apply stat decay to all active buddies.
  - **`process_client_command`**: Processes JSON commands (work, play, maintain, craft, scavenge, configure).
  - **`handle_scavenging`**: Manages scavenging timers and resource collection.
  - **`handle_crafting`**: Processes upgrade crafting progress and completion.
  - **`apply_shutdown_logic`**: Triggers buddy PowerOff state when neglected.
  - **`calculate_decay`**: Helper method for stat decay calculations using logarithmic efficiency curves.

- **`DatabaseManager` Class:** SQLite persistence layer.
  - **`get_buddy_state(lobby_id)`**: Retrieves buddy state for a specific lobby.
  - **`update_buddy_state(lobby_id, state)`**: Saves buddy state to SQLite database.
  - **`get_buddy_inventory(lobby_id)`**: Retrieves buddy's inventory and upgrades.
  - **`update_buddy_inventory(lobby_id, inventory)`**: Updates inventory in database.
  - **`get_lobby_config(lobby_id)`**: Retrieves lobby-specific configuration settings.
  - **`update_lobby_config(lobby_id, config)`**: Updates configurable game properties.

### **2. Game Data Model (Shared Components)**

This module defines the data structure for the Desk Bot and its progression, located in the Shared folder.

- **`DeskBot` Class:** A representation of a lobby's virtual pet (shared between server and client).

  - **`__init__(lobby_id)`**: Sets the bot's lobby ID and initializes core attributes: `productivity`, `mood`, `energy`, `health`, `raw_resources`, and `inventory`.
  - **`apply_decay`**: Called by GameEngine tick to reduce stats over time. Checks for PowerOff conditions.
  - **`perform_action(command, parameters)`**: Updates bot stats based on JSON commands.
  - **`start_scavenging(duration)`**: Initiates scavenging mode with specified duration.
  - **`complete_scavenging()`**: Completes scavenging and adds resources to inventory.
  - **`start_crafting(upgrade_name, resources_required)`**: Begins crafting an upgrade.
  - **`complete_crafting()`**: Finishes crafting and adds upgrade to inventory.
  - **`apply_upgrade_effects()`**: Applies active upgrade modifiers to stats and efficiency.
  - **`calculate_work_efficiency()`**: Calculates current work efficiency using logarithmic boredom curves.
  - **`trigger_shutdown()`**: Puts buddy into PowerOff state.
  - **`restart_buddy()`**: Brings buddy back online from PowerOff state.

- **`Inventory` Class:** Manages buddy's items and upgrades.
  - **`add_resource(resource_type, amount)`**: Adds Raw Resources to inventory.
  - **`consume_resources(requirements)`**: Uses resources for crafting.
  - **`add_upgrade(upgrade)`**: Adds completed upgrade to inventory.
  - **`remove_upgrade(upgrade_name)`**: Removes upgrade from active list.
  - **`get_active_upgrades()`**: Returns list of currently active upgrades.

### **3. Client Architecture (MVC Pattern)**

This module handles the TUI and user input following MVC pattern.

**View Layer:**

- **`TUIClient` Class:** The main TUI application and view controller.
  - **`__init__`**: Initializes TUI framework (Textual) and WebSocket client.
  - **`connect_to_server(lobby_id)`**: Establishes WebSocket connection with lobby ID.
  - **`display_tui()`**: Renders the text-based interface showing buddy stats, inventory, and command line.
  - **`display_scavenging_status()`**: Shows scavenging progress when buddy is out.
  - **`display_crafting_status()`**: Shows current crafting progress and queue.
  - **`prompt_lobby_id()`**: Initial screen for lobby ID input.

**Controller Layer:**

- **`ClientController` Class:** Handles user input and command processing.
  - **`listen_for_updates()`**: Asynchronous method waiting for WebSocket messages.
  - **`process_server_message(message)`**: Processes JSON messages from server and updates view.
  - **`send_command(command, parameters)`**: Sends JSON commands to server.
  - **`handle_user_input()`**: Processes user commands and validates input.
  - **`parse_json_config(config_string)`**: Parses configuration JSON for lobby settings.

### **4. JSON Message Format**

Simple JSON structure for all client-server communication:

**Client to Server Commands:**

```json
{
  "command": "connect|work|play|maintain|craft|scavenge|configure|status|inventory|restart",
  "lobby_id": "string",
  "parameters": {
    "item": "string",
    "upgrade": "string",
    "duration": "number",
    "config": {}
  }
}
```

**Server to Client Responses:**

```json
{
  "type": "status_update|scavenging_progress|crafting_progress|error",
  "buddy_state": {
    "productivity": 75,
    "mood": 60,
    "energy": 80,
    "health": 90,
    "is_scavenging": false,
    "is_crafting": false,
    "is_shutdown": false
  },
  "inventory": {
    "raw_resources": 150,
    "upgrades": ["Extra Arms", "Mood Chip"],
    "maintenance_items": ["Oil", "Power Cell"]
  },
  "progress": {
    "scavenge_eta": 0,
    "craft_progress": 25,
    "craft_item": "Turbo-Processor"
  },
  "message": "string"
}
```

### **5. How the Code Interacts (MVC Flow)**

1. **Connection**: The `TUIClient` prompts for lobby ID and establishes WebSocket connection to `Server`.
2. **Lobby Management**: Server checks if lobby exists. If not, creates new DeskBot instance in SQLite database.
3. **Game Loop**: Server runs `GameEngine.game_loop_tick()` every second, updating all active buddy states.
4. **State Persistence**: All buddy states, inventories, and configurations are stored in SQLite database.
5. **Client Updates**: Server broadcasts JSON status updates to connected clients for their lobby.
6. **User Commands**: Client sends JSON commands through WebSocket to server.
7. **Command Processing**: Server's `GameEngine.process_client_command()` updates buddy state and database.
8. **Scavenging Flow**: When scavenging, normal stats are hidden and only progress timer is displayed.
9. **Crafting Flow**: Users select upgrades to craft; buddy works on them over time; completed items go to inventory.
10. **Configuration**: Users can send JSON config to modify lobby-specific game settings.

For example, a web application might have modules for a database, user authentication, and a front-end interface. You're creating a map of the project's landscape.

---

Now, you'll break down your plan into a logical, high-level structure. The AI will assist in organizing your thoughts into a clear hierarchy.

- **Your Input (Required):**
  - **High-Level Components:** List the major modules, classes, or functions needed to achieve your project's goal. For example, "User Authentication Module," "Database Management," or "Front-End Interface."
- **AI's Role (Assistance):**
  - The AI can help you formalize these components and propose a logical file and folder structure. It can also suggest how different parts of the code should interact.
- **Prompt Example:** "Given these high-level components for my project, can you propose a logical directory structure for **\[Your Chosen Language/Framework]**? The components are: **\[List of your components]**."

---

### **6. Proposed Directory Structure**

This is the updated file and folder hierarchy separating server, client, shared components, and Docker configuration.

```
ByteBuddy/
├── docker/
│   ├── Dockerfile.server      # Server container configuration
│   ├── Dockerfile.client      # Client container configuration
│   └── docker-compose.yml     # Multi-container orchestration
├── server/
│   ├── __init__.py           # Makes 'server' a Python package
│   ├── main.py               # Contains 'Server' class and application entry point
│   ├── controllers/
│   │   ├── __init__.py
│   │   ├── game_engine.py    # GameEngine controller class
│   │   └── websocket_handler.py # WebSocket connection handling
│   ├── models/
│   │   ├── __init__.py
│   │   └── database_manager.py # DatabaseManager and SQLite operations
│   └── config/
│       └── settings.py       # Server configuration (bot_speed_multiplier, etc.)
├── client/
│   ├── __init__.py           # Makes 'client' a Python package
│   ├── main.py               # TUIClient application entry point
│   ├── views/
│   │   ├── __init__.py
│   │   ├── tui_interface.py  # TUI display logic
│   │   └── command_interface.py # Command input handling
│   └── controllers/
│       ├── __init__.py
│       └── client_controller.py # Client-side command processing
├── shared/
│   ├── __init__.py           # Makes 'shared' a Python package
│   ├── models/
│   │   ├── __init__.py
│   │   ├── deskbot.py        # DeskBot class (shared between server/client)
│   │   └── inventory.py      # Inventory management class
│   └── utils/
│       ├── __init__.py
│       └── message_format.py # JSON message formatting utilities
├── data/
│   └── buddies.db            # SQLite database file
├── .env.example              # Environment variables template
├── .gitignore
├── README.md                 # Project setup and usage instructions
└── requirements.txt          # Python dependencies
```

### **7. Docker & Deployment Configuration**

#### **7.1. Environment Variables**

The system supports single environment deployments (dev or prod) through environment variables:

```bash
# .env file
ENVIRONMENT=dev              # or 'prod'
BOT_SPEED_MULTIPLIER=1.0    # Game tick speed modifier
DATABASE_PATH=/data/buddies.db
WEBSOCKET_PORT=8080
WEBSOCKET_HOST=0.0.0.0
SCAVENGING_DEFAULT_DURATION=3600  # 1 hour in seconds
```

#### **7.2. Docker Configuration**

**Server Dockerfile (docker/Dockerfile.server):**

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY server/ ./server/
COPY shared/ ./shared/
EXPOSE 8080
VOLUME ["/data"]
CMD ["python", "-m", "server.main"]
```

**Client Dockerfile (docker/Dockerfile.client):**

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY client/ ./client/
COPY shared/ ./shared/
CMD ["python", "-m", "client.main"]
```

**Docker Compose (docker/docker-compose.yml):**

```yaml
version: "3.8"
services:
  bytebuddy-server:
    build:
      context: ..
      dockerfile: docker/Dockerfile.server
    ports:
      - "8080:8080"
    volumes:
      - buddy_data:/data
    environment:
      - ENVIRONMENT=${ENVIRONMENT:-dev}
      - BOT_SPEED_MULTIPLIER=${BOT_SPEED_MULTIPLIER:-1.0}
      - DATABASE_PATH=/data/buddies.db
      - WEBSOCKET_PORT=8080
    restart: unless-stopped

  bytebuddy-client:
    build:
      context: ..
      dockerfile: docker/Dockerfile.client
    environment:
      - SERVER_HOST=bytebuddy-server
      - SERVER_PORT=8080
    depends_on:
      - bytebuddy-server
    stdin_open: true
    tty: true
    profiles: ["client"]

volumes:
  buddy_data:
```

#### **7.3. Deployment Instructions**

**Development Deployment:**

```bash
# Start server only
docker-compose up bytebuddy-server

# Start client (separate terminal)
docker-compose --profile client up bytebuddy-client

# Or run client locally
python -m client.main
```

**Production Deployment:**

```bash
# Set production environment
export ENVIRONMENT=prod
export BOT_SPEED_MULTIPLIER=0.5

# Deploy server
docker-compose up -d bytebuddy-server
```

### **8. Additional Recommendations**

Based on the project scope and requirements, here are additional suggestions:

#### **8.1. Essential Files to Add**

- **README.md:** Project overview, setup instructions, and usage examples
- **requirements.txt:** Python dependencies (websockets, textual, sqlite3)
- **.env.example:** Template for environment configuration
- **.gitignore:** Exclude database files, Docker volumes, and Python cache

#### **8.2. Configuration Considerations**

- **Single Environment:** System supports either dev or prod deployment, not both simultaneously
- **Database Portability:** SQLite file should be volume-mounted for data persistence
- **WebSocket Security:** Consider adding authentication tokens for production use

#### **8.3. Example Commands and Usage**

Include in README.md:

```bash
# Connect to lobby
connect myteam-lobby-01

# Basic commands
status
work
play
maintain Oil

# Crafting system
craft Extra Arms
inventory

# Scavenging
scavenge

# Configuration
configure {"scavenging_duration": 1800, "resource_multiplier": 2.0}
```

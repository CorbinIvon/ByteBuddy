Reference [[1. Project Planning]]

The **planning phase** is about understanding the project's goal and requirements. This is where you answer questions like:

**What is the project's purpose?** What problem does it solve?

- Simply put, I thought it would be a fun little project to share with a co-worker.

Reference [[1. Project Planning]]

**Major Updates & Features:**

- The server uses SQLite for all persistence.
- The project follows a Model-View-Controller (MVC) approach.
- The Buddy has its own inventory (formerly "Supply Closet").
- Upgrades require components, time to craft, and can be added/removed at runtime via JSON. Upgrades affect stats and crafting speed, e.g., "Extra Arms" increases work speed but increases boredom rate, reducing efficiency logarithmically.
- The Buddy can be sent out to "Scavenge" for 1 hour to retrieve generic "Raw Resources" for crafting.
- System failure is a "shutdown" (PowerOff) until a user logs in and fixes the Buddy, pausing all activity.
- Only a single environment (dev or prod) is supported.
- Docker and docker-compose are used for deployment, with SQLite persisted via volume.
- The client can run a configure command and send JSON to modify game properties and logic for the lobby.
- On connection, the user enters a lobby ID. If it exists, resumes with that Buddy; otherwise, creates a new Buddy and associates it with the lobby. Multiple users can care for the same Buddy; clients are anonymous.
  **What features are necessary?** What features are just "nice to have"?

- Run on a server and tick every second.
- Server Websocket
- Dynamic Curses TUI Client
  Nice to have:

**What technologies and languages will be used?** (e.g., Python, JavaScript, Java, etc.)

**How will the project be deployed or used?**

- The project will be deployed via docker-compose. The project files will be compiled and deployable.
  This stage often involves creating a **project specification document** or a simple list of requirements. Think of it as creating a blueprint for a house before you ever lay a foundation.

---

This initial phase is about defining the project's core purpose and requirements. You'll provide the high-level vision, and the AI will help you refine it.

- **Your Input (Required):**
  - **Project Name:** ByteBuddy
  - **Project Goal:** Tamagotchi inspired game project
    - Server + tick + websocket
    - TUI Client
  - **Target Audience:** tui bros
  - **Technical Stack:** List the programming languages, frameworks, and technologies you intend to use (e.g., "Python, Django, SQLite").
    - Python
- **AI's Role Prompt (Assistance):**
  - The AI will help you brainstorm, identify potential challenges, and suggest features you might have missed. It can act as a sounding board for your ideas.
- **Prompt Example:** "Based on my project plan above, can you identify any potential challenges or missing features?

[[2. Project Outlining]]

Reference [[2. Project Outlining]]

Once the plan is in place, the **outlining phase** focuses on breaking the project down into smaller, manageable pieces. This is where you determine the main components and their relationships. A common way to do this is to create a **high-level design** or **system architecture**. This might include:

- Listing the key **modules** or **classes** needed.
- Defining the major **functions** and **methods** within each module.
- Considering how different parts of the code will interact.

### **1. Server Architecture**

This module handles all core game logic, state management, and communication with the clients.

- **`Server` Class:** The primary entry point for the back-end.
  - **`__init__`**: Initializes the WebSocket server and instances of the `GameEngine` and `DatabaseManager`.
  - **`start_server`**: A method that starts the WebSocket listener and a separate, periodic timer for the `GameEngine`'s `tick`.
  - **`handle_client_connection`**: An asynchronous method that accepts new WebSocket connections and listens for incoming commands from a specific client.
  - **`broadcast_update`**: A method that sends the updated `DeskBot` state to the client(s) after a game tick or a successful command.
- **`GameEngine` Class:** The brain of the game, responsible for managing all game rules and progression.
  - **`__init__`**: Initializes with a reference to the `DatabaseManager` and the configurable `bot_speed_multiplier`.
  - **`game_loop_tick`**: A method called every second by the server. It iterates through all active Desk Bots in the database and applies the decay to their stats.
  - **`process_client_command`**: Receives a command from the `Server` (e.g., `work`, `play`, `maintain`). It then calls the appropriate method on the target `DeskBot` object and updates its state in the database.
  - **`calculate_decay`**: A private helper method to determine how much each stat should decrease per tick, using the `bot_speed_multiplier`.
- **`DatabaseManager` Class:** Handles all data persistence. This is crucial for managing the shared leaderboard and supply closet.
  - **`get_bot_state(user_id)`**: Retrieves the current state of a specific Desk Bot.
  - **`update_bot_state(user_id, new_state)`**: Saves the new state of a Desk Bot to the database.
  - **`get_leaderboard_data`**: Queries the database for all Desk Bots and returns a sorted list based on Floppy Disk count.
  - **`update_supply_closet`**: Adds or removes items from the shared inventory.

### **2. Game Data Model**

This module defines the data structure for the Desk Bot and its progression.

- **`DeskBot` Class:** A representation of a single player's virtual pet.
  - **`__init__(name)`**: Sets the bot's name and initializes its core attributes: `productivity`, `mood`, `energy`, `health`, and `floppy_disks`.
  - **`apply_decay`**: A method called by the `GameEngine`'s `tick` to reduce the bot's stats over time. It will check if any stats have hit zero and apply consequences (e.g., stopping floppy disk production).

### **3. Client Architecture**

This module handles the TUI and user input.

- **`TUIClient` Class:** The main application for the user's terminal.
  - **`connect_to_server`**: Establishes the WebSocket connection.
  - **`display_tui`**: The method that renders the text-based interface, showing the bot's stats, the command line, and other info.
  - **`listen_for_updates`**: An asynchronous method that waits for messages from the server.
  - **`process_server_message`**: Receives a WebSocket message (a JSON object) and updates the local state, triggering a redraw of the TUI.

### **4. How the Code Interacts**

1. **Connection**: The `TUIClient` establishes a WebSocket connection to the `Server`.
2. **State Update**: The `game_loop_tick` uses the `DatabaseManager` to retrieve and update the stats for all `DeskBot` instances.
3. **Broadcast**: After updating, the `Server`'s `broadcast_update` method sends the new state for a specific bot back to its connected `TUIClient`.
4. **Display**: The `TUIClient`'s `listen_for_updates` receives the message and calls `process_server_message` to update the display using `display_tui`.
5. **User Input**: The user types a command. The `TUIClient`'s `send_command` sends a message to the `Server`.
6. **Command Processing**: The `Server`'s `handle_client_connection` receives the command and passes it to the `GameEngine`'s `process_client_command` to apply the changes to the `DeskBot` in the database.

For example, a web application might have modules for a database, user authentication, and a front-end interface. You're creating a map of the project's landscape.

---

Now, you'll break down your plan into a logical, high-level structure. The AI will assist in organizing your thoughts into a clear hierarchy.

- **Your Input (Required):**
  - **High-Level Components:** List the major modules, classes, or functions needed to achieve your project's goal. For example, "User Authentication Module," "Database Management," or "Front-End Interface."
- **AI's Role (Assistance):**
  - The AI can help you formalize these components and propose a logical file and folder structure. It can also suggest how different parts of the code should interact.
- **Prompt Example:** "Given these high-level components for my project, can you propose a logical directory structure for **\[Your Chosen Language/Framework]**? The components are: **\[List of your components]**."

---

This is a logical file and folder hierarchy for your project, which separates the server, client, and shared components.

```
bytebuddy/
├── server/
│   ├── __init__.py         # Makes 'server' a Python package
│   ├── main.py             # Contains the 'Server' class and starts the application
│   ├── game_engine.py      # Contains the 'GameEngine' class
│   └── database_manager.py # Contains the 'DatabaseManager' class
├── client/
│   └── main.py             # Contains the 'TUIClient' class and runs the client
├── game/
│   ├── __init__.py         # Makes 'game' a Python package
│   └── deskbot.py          # Contains the 'DeskBot' class and related logic
├── shared/
│   ├── __init__.py         # Makes 'shared' a Python package
│   └── models.py           # For shared data models or message formats (e.g., DeskBot attributes)
├── config/
├── data/
│   └── game.db             # Placeholder for the database file
├── .gitignore
├── Dockerfile              # Instructions to build the Docker image
└── docker-compose.yml      # Instructions to run the server and client with Docker
```

### Office Tamagotchi: An Incremental CLI Robot Buddy Outline

This outline details the core concepts and mechanics for a command-line interface (CLI) Tamagotchi-inspired game designed for an office environment. The goal is to create an incremental game that encourages friendly competition and provides a quick, satisfying distraction without any graphical elements.

#### **1. Core Concept**

- **The Pet:** A small, digital robot represented by a status line and a text-based description. Let's call it a "Desk Bot."
- **The Player:** The office worker who is responsible for the Desk Bot's well-being by typing commands into a terminal.
- **The Goal:** Keep your Desk Bot happy, healthy, and productive. The bot's status reflects your care, and its productivity generates valuable "Floppy Disks" that can be used for upgrades.

#### **2. Game Mechanics**

The game revolves around four primary needs that you must manage. These needs will decay over time, requiring regular attention. All stats are displayed as a percentage or a text-based bar. The decay and production rates can be adjusted by a configurable **bot speed multiplier** on the server.

- **Productivity:** The Desk Bot's ability to "work" and generate Floppy Disks. This stat is crucial for your long-term progression. The bot will automatically produce Floppy Disks over time as long as its other stats (Mood, Energy, Health) are not at zero.
- **Mood:** The Desk Bot's happiness level, often tied to its "emotional" programming. You can increase its mood by using a `play` or `joke` command. Low mood can reduce the rate of Floppy Disk production.
- **Energy & Maintenance:** The Desk Bot's power level and need for upkeep. You can `recharge` it with power cells or `maintain` it with virtual oil and spare parts. This stat directly affects the bot's productivity and health.
- **Health:** A general wellness stat. A well-maintained and charged Desk Bot will have good health. Neglect can cause it to "overheat" or experience "critical errors," which can stop all Floppy Disk production.

#### **3. Actions & Interface**

The game's interface is entirely text-based. You will type commands to interact with your Desk Bot.

- **`work`:** Assigns a task to the Desk Bot, increasing its Productivity.
- **`play`:** Boosts the Desk Bot's Mood. The command might output a short, playful text snippet.
- **`maintain [item]`:** Gives the Desk Bot a maintenance item from your inventory, affecting its Energy and other stats.
- **`status`:** Displays the current levels of Productivity, Mood, Energy, and Health using numbers and/or text-based bars, as well as your current Floppy Disk balance.
- **`upgrade [upgrade_name]`:** Spends Floppy Disks to permanently improve your Desk Bot's abilities.
- **`leaderboard`:** Displays the top-performing Desk Bots and their owners in a simple, ranked text list based on total Floppy Disks earned.

#### **4. Life Cycle & Progression**

- **Birth:** The game begins with a "starter kit." The Desk Bot "activates" after a certain amount of time or after you perform a set number of tasks.
- **Upgrades:** The core of the incremental loop. You can use Floppy Disks to purchase upgrades that make the bot more efficient. Examples include:
  - **Turbo-Processor:** Increases the rate of Floppy Disk production.
  - **Extended Battery:** Makes the Energy stat decay more slowly.
  - **Mood Chip:** Makes the Mood stat decay more slowly.
  - **Advanced Tools:** Increases the effectiveness of maintenance items.
- **Consequences:** If you neglect your Desk Bot for too long, a status check might display a message like "Your Desk Bot has entered sleep mode due to low power" or "System failure. Floppy Disk production has stopped." This forces you to start over with a new starter kit, losing all upgrades.

#### **5. Office Integration**

- **Shared "Supply Closet":** A common, shared inventory where players can leave maintenance items and power cells for others. This is managed by a shared, text-based list.
- **Leaderboard:** A central, text-based list that showcases the top Desk Bots based on a combined score of their stats.
- **Notifications:** Simple, non-intrusive text-based notifications that pop up in the terminal when your Desk Bot needs urgent attention.

#### **6. Technical Architecture**

- **The Server (The Office Buddy):** This will be a single, central server that manages the game's state. It will be responsible for tracking all players' Desk Bots, updating their stats over time, and managing the shared leaderboard and "Supply Closet."
- **Server Configuration:** The server will be fully configurable via a settings file or command-line arguments. This includes a **`bot_speed_multiplier`** that can be adjusted to change how quickly all stats (productivity, mood, energy) decay. A higher multiplier would make the game more challenging by requiring more frequent check-ins.
- **The Clients:** Each office worker will have a client application (likely a web app or browser tab running a terminal-like interface). The client will connect to the server to display their Desk Bot's current status and send text commands back to the server. Since the point is to check in once a day, the clients will primarily be for these quick daily interactions.

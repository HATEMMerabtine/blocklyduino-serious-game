
# BlocklyDuino Custom Learning Environment

## 1. Project Overview

This project is a major evolution of the classic [BlocklyDuino](https://github.com/BlocklyDuino/BlocklyDuino), transforming it into a structured, level-based learning environment ideal for integration into educational platforms and gamified tools like **GDevelop**.

While preserving the core functionality of BlocklyDuino, this fork introduces a new pedagogical approach. It guides learners progressively with structured toolboxes, integrated help, and scene-safe GDevelop interoperability.

---

## 2. Key Enhancements

### 2.1 Level-Based Architecture

The environment is structured into four distinct learning levels. Each level is accessed via a URL like `/level/1`, which dynamically loads a unique configuration to guide the learning process:

-   **Level-Specific Toolboxes (`toolboxXML`)**: To prevent overwhelming the learner, only the blocks required for the current level's task are shown. This helps learners stay focused and master concepts incrementally.
-   **Pre-Defined Workspace Layouts (`levelXML`)**: Some levels preload blocks onto the workspace, giving learners a structured starting point.
-   **Targeted Documentation & Help**: Each level has its own documentation and hints directly related to the concepts being introduced.

### 2.2 Enhanced UI/UX

The interface has been modernized and optimized for a better learning experience:

-   **Modern Styling**: A new layout using Flexbox, gradients, and improved typography.
-   **Responsive Design**: Fully usable on both desktop and mobile devices.
-   **Simplified Controls**: The interface is streamlined, showing only task-relevant buttons like **"Upload to Arduino"**, **"Reset"**, **"Documentation"**, and **"Help"**.

### 2.3 Learning Tools

#### Documentation Panel
A toggleable sidebar features an accordion-style menu that provides detailed information for each level, including block descriptions and Arduino code examples.

#### Modal Help System
A modal window provides level-specific hints. Help for the current level is automatically displayed on page load to guide the learner.

### 2.4 Integration with GDevelop

The environment is designed for robust communication with a parent GDevelop application via `postMessage`:

-   **Code Transfer (`sendCodeToGDevelop`)**: The **"Upload to Arduino"** button sends the generated C++ code to the parent window for validation.
-   **Click Tracking (`computeClickCount`)**: The total number of user clicks is sent to the parent window for analytics or in-game logic.
-   **Bi-directional Feedback (`showNotification`)**: The environment can receive messages from GDevelop to display non-blocking visual notifications (e.g., "Correct!" or "Incorrect, try again!"), providing immediate feedback.

---

## 3. How to Use

-   **Navigate to a Level**: Start by accessing the URL for the level you want to play (e.g., `.../level/1`).
-   **Build Your Logic**: Use the visual blocks from the toolbox to construct the solution to the level's puzzle.
-   **Get Assistance**: If you're stuck, click the **Documentation** button to learn about the available blocks or the **Help** button to see hints for the current level.
-   **Submit Your Code**: Once you believe you have the correct solution, press the **Upload to Arduino** button to send your code to the host GDevelop application for checking.
-   **Start Over**: To clear the workspace and reload the level from its initial state, press the **Reset** button.

---

## 4. GDevelop Integration Guide

### Best Practice: Scene-Specific, Unified Event Listeners

To ensure a bug-free experience, it is critical that each GDevelop scene manages its own message listener. A listener created in Level 1 must be **destroyed** when you leave the scene to prevent it from conflicting with the listener in Level 2.

The most efficient method is to use a **unified listener** for each level that handles all message types (`blocklyCode`, `clickCount`, etc.) and stores data in unique scene variables (e.g., `BlocklyCode1`, `ClickCount1`).

**Follow this two-step pattern for every level:**

1.  **Create a "Setup" Event:** Use a JavaScript event with the condition `At the beginning of the scene` to create and add the listener.
2.  **Create a "Cleanup" Event:** In the event that triggers the scene change (e.g., a "Next Level" button), add a JavaScript action to remove the listener. This action **must** come before the "Change Scene" action.

---

### **Complete Code for Level 1**

**1. "Setup" Event (Condition: `At the beginning of the scene`)**

```javascript
// --- LEVEL 1: SETUP UNIFIED LISTENER ---

console.log("GDevelop Level 1: Initializing listeners...");

// Define ONE handler function for ALL Level 1 messages
window.handleLevel1Messages = function(event) {
  if (!event.data || !event.data.type) return; // Ignore invalid messages

  const sceneVariables = runtimeScene.getVariables();
  const iframe = document.getElementById("myIframe"); // Ensure your iframe object has this ID

  // --- Handle Blockly Code ---
  if (event.data.type === "blocklyCode") {
    let blocklyCode = event.data.code.replace(/\s+/g, '');
    console.log("L1 Received code:", blocklyCode);
    
    // Store code in a unique scene variable for this level
    sceneVariables.get("BlocklyCode1").setString(blocklyCode);
    console.log("Stored code in 'BlocklyCode1'.");
    
    // Validate the code against the "correct" scene variable
    const correctCode = sceneVariables.get("correct").getAsString();
    let feedbackMessage = (correctCode === blocklyCode) ? "Correct!" : "Incorrect, try again!";
    
    // Send feedback back to the iframe
    if (iframe && iframe.contentWindow) {
      iframe.contentWindow.postMessage({ type: "blocklyFeedback", message: feedbackMessage }, "*");
    }
  }

  // --- Handle Click Count ---
  if (event.data.type === "clickCount") {
    let clickCount = event.data.clicks;
    console.log("L1 Received clicks:", clickCount);
    // Store in a unique scene variable for this level
    sceneVariables.get("ClickCount1").setNumber(clickCount);
    console.log("Stored clicks in 'ClickCount1'.");
  }
};

// Add the listener using a unique flag for this level
if (!window.level1ListenerActive) {
  window.addEventListener("message", window.handleLevel1Messages, false);
  window.level1ListenerActive = true;
  console.log("Level 1 Unified Listener ADDED.");
}
```

**2. "Cleanup" Event (Action in "Next Level" button, *before* "Change Scene")**

```javascript
// --- LEVEL 1: CLEANUP UNIFIED LISTENER ---

if (window.level1ListenerActive) {
  window.removeEventListener("message", window.handleLevel1Messages, false);
  window.level1ListenerActive = false;
  console.log("Level 1 Unified Listener REMOVED.");
}
```

---

### **Adapting for Other Levels (Level 2, 3, 4)**

To implement this for other levels, simply copy the code and change the unique identifiers. For **Level 2**, your code would use:

-   **Handler function:** `window.handleLevel2Messages`
-   **Listener flag:** `window.level2ListenerActive`
-   **Scene variables:** `"BlocklyCode2"`, `"ClickCount2"`
-   **Console logs:** `console.log("L2 Received code...")`

This pattern is essential for a stable and scalable project.

---

## 5. Level Breakdown

Here is a summary of the tools and tasks for each level as defined in the environment:

### Level 1: Basic Output
-   **Objective**: Learn to control a digital output.
-   **Toolbox Blocks**: `Digital Write`.
-   **Hints**: "Use DigitalWrite to set port 13 to HIGH to turn on the LED."

### Level 2: Analog Input & Conditional Logic
-   **Objective**: Read an analog sensor and use an `if-else` condition.
-   **Pre-loaded Blocks**: `variables_set` (for `sensor_value` and `temp`), `variables_get`.
-   **Toolbox Blocks**: `controls_if`, `logic_compare`, `inout_analog_read`, `inout_digital_write`, `inout_highlow`, `math_number`.
-   **Hints**: Involves reading a temperature sensor, comparing it to a threshold, and controlling an output based on the result.

### Level 3: Digital Input & Servos
-   **Objective**: Read a digital input (like a PIR sensor) and control a servo motor.
-   **Pre-loaded Blocks**: `variables_set` and `variables_get` for `pir_value`.
-   **Toolbox Blocks**: `controls_if`, `logic_compare`, `inout_digital_read`, `inout_digital_write`, `inout_highlow`, `servo_move`, `servo_read_degrees`, `math_number`.
-   **Hints**: Guides the user to read a motion sensor, move a servo to different positions based on the sensor's state, and signal an output pin.

### Level 4: Combining Inputs & Nested Logic
-   **Objective**: Use multiple sensors (analog and digital) and nested conditional logic.
-   **Pre-loaded Blocks**: `variables_set` and `variables_get` for `pir_value` and `light`.
-   **Toolbox Blocks**: `controls_if`, `logic_compare`, `inout_digital_read`, `inout_analog_read`, `inout_digital_write`, `inout_highlow`, `math_number`.
-   **Hints**: "Use two blocks of if-else condition, one for the light and inside it one for the motion sensor." The task involves checking for low light and then checking for motion to control multiple outputs.

---

## 6. Legacy BlocklyDuino Overview

> For users of the original BlocklyDuino, the following functionalities are retained and available.

-   Visual drag-and-drop Arduino code blocks
-   Generation of Arduino-compatible C++ code
-   Grove sensor block support
-   URL-based loading of examples

### Try Online

-   [Smart in the Dark - Custom Version - Level 1](https://blocklyduino-smart-in-the-dark-sg.netlify.app/level/1)
-   [Smart in the Dark - Custom Version - Level 2](https://blocklyduino-smart-in-the-dark-sg.netlify.app/level/2)
-   [Blink Example](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/blink.xml)
-   [Servo + Potentiometer](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/servo_potentio.xml)
-   [LED Color Button](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/click_color.xml)

---

## 7. Running Locally

Clone the repository and open:

```bash
blockly/apps/blocklyduino/index.html
```

Or serve it via a local server:

```bash
python arduino_web_server.py --port=COM3
```

Access it on [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

---

## 8. Authors & Credits

### Original Authors

Fred Lin ([@gasolin](https://github.com/gasolin))
Thanks to: Neil Fraser, Q.Neutron, Dale Low, Seeeduino, Arduino, contributors of Blockly, and inspired projects like [ArduBlock](https://github.com/taweili/ardublock).

### Enhanced & Maintained By

### Contributors


Hatem Merabtine ([@HATEMMerabtine](https://github.com/HATEMMerabtine))  
Lead developer of the level-based learning environment and GDevelop integration.
**Project Supervisor**  
Dr. Rida Mezghache ([@Rida-Mezghache](https://github.com/Rida-Mezghache))  
Provided conceptual design, feature guidance, and documentation strategy.

---

## 9. License

Licensed under the [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)

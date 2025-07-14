
# BlocklyDuino Custom Learning Environment

## 1. Project Overview

This project is a **major evolution** of the classic [BlocklyDuino](https://github.com/BlocklyDuino/BlocklyDuino), turning it into a **structured, level-based learning environment** ideal for integration into educational platforms and gamified tools like **GDevelop**.

While preserving the core functionality of BlocklyDuino (a web-based visual programming tool for Arduino), this fork introduces a new **pedagogical approach** by guiding learners progressively, with structured toolboxes, integrated help, and GDevelop interoperability.

---

## 2. Key Enhancements

### 2.1 Level-Based Architecture

Each level is accessed via a URL like `/level/1`, which dynamically loads:

- **Level-Specific Toolboxes (`toolboxXML`)**  
  Only the blocks required for a given level are shown, helping learners stay focused.
  
- **Pre-Defined Workspace Layouts (`levelXML`)**  
  Levels can preload blocks, giving learners a strong starting point.

---

### 2.2 Enhanced UI/UX

The interface has been **modernized** and optimized:

- **Modern Styling**: New layout using Flexbox, gradients, typography, and shadows.
- **Responsive Design**: Fully usable on both desktop and mobile.
- **Simplified Controls**: Only task-relevant buttons are shown (e.g., no "Load XML").

---

### 2.3 Learning Tools

#### Documentation Panel
- Toggleable sidebar with collapsible accordion structure.
- Includes block descriptions and Arduino code examples.

#### Modal Help System
- Level-specific hints shown in a modal.
- Encourages exploration without giving direct answers.

---

### 2.4 Integration with GDevelop

The environment can communicate with a GDevelop application via `postMessage`:

- **Code Transfer (`sendCodeToGDevelop`)**  
  The **"Upload to Arduino"** button sends generated code to the parent window.

- **Click Tracking (`computeClickCount`)**  
  Sends total user clicks to the parent window—useful for analytics or in-game logic.

- **Feedback System (`showNotification`)**  
  Non-blocking, visual notifications when events like code upload succeed.

---

## 3. How to Use

1. Navigate to:  
   `https://your-domain.com/level/1`

2. Use the toolbox to build the Arduino logic for the current level.

3. Click **Documentation** to explore available blocks.

4. Use the **Help** button to view hints.

5. Press **Upload to Arduino** to send the code to your host app.

6. Press **Reset** to reload the level.

---

## 4. GDevelop Integration Snippets

### 4.1 Receiving Blockly Code in GDevelop

```javascript
function removeWhitespace(str) {
  return str.replace(/\s+/g, '');
}

if (!window.blocklyCodeListenerAdded) {
  window.addEventListener("message", function (event) {
    if (event.data && event.data.type === "blocklyCode") {
      var blocklyCode = event.data.code;
      blocklyCode = removeWhitespace(blocklyCode);
      console.log("Received Blockly Code:", blocklyCode);

      const sceneVariables = runtimeScene.getVariables();
      sceneVariables.get("BlocklyCode").setString(blocklyCode);

      console.log("Stored in scene variable 'BlocklyCode'");
    }
  }, false);
  window.blocklyCodeListenerAdded = true;
}
```

### 4.2 Receiving the Click Count

```javascript
if (!window.clickCountListenerAdded) {
  window.addEventListener("message", function (event) {
    if (event.data && event.data.type === "clickCount") {
      var clickCount = event.data.clicks;
      console.log("Click Count:", clickCount);

      const sceneVariables = runtimeScene.getVariables();
      sceneVariables.get("ClickCount").setNumber(clickCount);

      console.log("Stored in scene variable 'ClickCount'");
    }
  }, false);
  window.clickCountListenerAdded = true;
}
```

---

## 5. Legacy BlocklyDuino Overview

> For users of the original BlocklyDuino, the following functionalities are retained and available.

- Visual drag-and-drop Arduino code blocks  
- Generation of Arduino-compatible C++ code  
- Grove sensor block support  
- URL-based loading of examples  

### Try Online

- [Smart in the Dark - Custom Version](https://blocklyduino-smart-in-the-dark-sg.netlify.app/)
- [Blink Example](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/blink.xml)  
- [Servo + Potentiometer](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/servo_potentio.xml)  
- [LED Color Button](http://blocklyduino.github.io/BlocklyDuino/blockly/apps/blocklyduino/index.html?url=examples/click_color.xml)  

---

## 6. Running Locally

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

## 7. Authors & Credits

### Original Authors  
Fred Lin ([@gasolin](https://github.com/gasolin))  
Thanks to: Neil Fraser, Q.Neutron, Dale Low, Seeeduino, Arduino, contributors of Blockly, and inspired projects like [ArduBlock](https://github.com/taweili/ardublock).

### Enhanced & Maintained By

### Contributors

- **[Hatem Merabtine]** – Main developer; implemented BlocklyDuino integration, level-based system, and other features.
- **[Dr. Rida Mezghache]** – Project supervisor; provided concept design, guidance on features, and documentation planning.

---

## 8. License

Licensed under the [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)

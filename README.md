# P5.js Serial Data Visualizer

This web application connects to serial devices using the Web Serial API, collects incoming numerical data (comma-separated values), and provides various real-time and post-collection visualizations using the p5.js library.

## Functionality

The core purpose of this application is to provide a flexible platform for visualizing data streamed from serial devices, such as Arduino boards or other microcontrollers equipped with sensors. It allows users to:

1.  **Connect:** Establish a connection to a compatible serial device detected by the browser.
2.  **Collect:** Record data streams for a specified duration or indefinitely.
3.  **Visualize Live:** View the incoming data in real-time using one of several selectable visualization modes.
4.  **Visualize Collected Data:** After collection stops, view the entire dataset using different visualization techniques.
5.  **Control:** Adjust visualization parameters (like speed, opacity, line height for specific modes) using UI controls or mapped sensor inputs.
6.  **Persist Settings:** Save UI preferences (durations, selected modes, control values) locally for convenience across sessions.
7.  **Debug:** View the raw incoming serial data stream directly on the canvas.
8.  **Fullscreen:** View the visualization in an immersive fullscreen mode.

## Key Features

*   **Web Serial API Integration:** Connects directly to serial ports via compatible browsers (Chrome, Edge).
*   **Auto-Reconnect:** Remembers the last used serial port (if only one was granted permission) and attempts to reconnect automatically on page load.
*   **Flexible Data Collection:**
    *   Set collection duration in seconds.
    *   Option to collect data indefinitely until manually stopped.
*   **Multiple Live Visualizations:**
    *   **Centered Strips:** Represents the latest data point as colored strips horizontally centered.
    *   **Particle Emitters:** Uses sensor values to control the rate and color of particles emitted from points on the canvas.
    *   **Woven Tapestry:** Creates a vertical tapestry effect using historical data.
    *   **Woven Tapestry (Overlay):** Draws the latest data as a horizontal line that scrolls vertically, overlaying previous lines with adjustable speed, opacity, and line height.
    *   **None:** Shows only collection progress without a specific live visualization.
*   **Multiple Post-Collection Visualizations:**
    *   **Line Graph:** Classic plot of sensor values over time.
    *   **Circular Plot:** Represents the last data point as concentric circles.
    *   **Radial Bars:** Plots data points radially from the center.
    *   **Stacked Bars:** Shows the proportional contribution of each sensor over time, normalized to the canvas height.
    *   **Histogram:** Displays the frequency distribution of values for the first sensor stream.
    *   **Heatmap:** Visualizes sensor values over time using color intensity in a grid.
*   **Real-time Control:**
    *   The "Woven Tapestry (Overlay)" mode's speed, opacity, and weft height can be controlled by incoming sensor values (mapped to indices 10, 11, and 12 respectively).
    *   UI sliders allow manual control and reflect sensor input when the overlay mode is active.
*   **Dynamic Data Handling:** Adapts to varying numbers of data points per line received from different serial devices by padding shorter lines or truncating longer ones during collection based on the first valid data received.
*   **Settings Persistence:** Uses `localStorage` to save and load user preferences for collection duration, indefinite mode, selected live/final visualizations, overlay slider values, and debug mode state.
*   **Debug Mode:** A toggle allows displaying the raw, comma-separated numerical data received from the serial port directly on the canvas instead of a visualization.
*   **Fullscreen Mode:** Clicking the canvas toggles a fullscreen view, hiding UI elements and resizing the canvas to fit the screen.

## Code Structure (`index.html`)

The application is contained within a single HTML file, leveraging Tailwind CSS for styling and p5.js for graphics.

1.  **HTML (`<body>`)**:
    *   Main container (`#mainContentContainer`): Holds all visible content.
    *   Title (`#mainTitle`).
    *   Controls container (`.controls`): Contains buttons (`#connectButton`, `#startButton`, `#stopButton`), inputs (`#collectDuration`, `#collectIndefinitely`), dropdowns (`#liveVizSelect`, `#vizSelect`), overlay sliders (`#overlayControlsContainer`), and the debug toggle (`#debugModeCheckbox`).
    *   Status indicators (`#status`, `#errorDisplay`, `#dataCount`).
    *   Canvas container (`#canvasContainer`): Where the p5.js canvas is rendered.

2.  **CSS (`<style>` & Tailwind)**:
    *   Basic styles for elements.
    *   Status indicator styling based on connection/collection state.
    *   Tailwind utility classes for layout and appearance.

3.  **JavaScript (`<script>`)**:
    *   **Global Variables:** Store state information (e.g., `port`, `isCollecting`, `collectedData`, `currentVisualization`, UI settings like `overlayDrawSpeed`).
    *   **UI Element References:** Constants linking JS variables to HTML elements by ID.
    *   **Utility Functions:** `updateStatus`, `showError`, `hideError`, `updateDataCount`, `updateButtonStates`, `saveSettings`, `loadSettings`, `updateUIDependentElements`.
    *   **Serial Communication:**
        *   `connectSerial()`: Handles connection requests and auto-reconnect logic.
        *   `disconnectSerial()`: Closes the port and cleans up resources.
        *   `readLoop()`: Continuously reads data from the serial port.
        *   `processIncomingData()`: Buffers incoming data chunks and splits them into lines.
        *   `parseAndStoreData()`: Parses lines, validates data, handles padding/truncation, stores data in `collectedData`, updates `absoluteLatestData`, and processes sensor inputs for overlay controls.
    *   **Data Collection:**
        *   `startCollection()`: Initializes collection state, starts timers (if applicable), and prepares the p5 canvas.
        *   `stopCollection()`: Stops collection, clears timers, updates status.
    *   **p5.js Sketch (`sketch` function):**
        *   `setup()`: Creates the canvas, sets color mode, initializes overlay position if needed, attaches the fullscreen click listener.
        *   `draw()`: The main rendering loop. Checks `isCollecting` state and `isDebugMode`. Calls the appropriate live visualization logic (via `switch` statement) or the selected final visualization function (from `visualizations` object) using `collectedData`.
        *   `visualizations` Object: Contains functions for each *post-collection* visualization type (e.g., `lines`, `stackedBars`). Live visualizations are handled directly within the `draw()` loop's switch statement.
        *   `Particle` Class: Used by the 'particleEmitters' visualization.
        *   `windowResized()`: Handles canvas resizing, adjusting for fullscreen mode.
        *   `clearCanvas()`: Helper to clear the canvas.
    *   **Event Listeners:** Attached to UI elements (buttons, selects, inputs, checkboxes) to trigger actions like connecting, starting/stopping collection, changing visualizations, saving settings, and handling fullscreen changes (`fullscreenchange` listener on `document`).
    *   **Initialization (`DOMContentLoaded`):** Executes when the page is loaded. Calls `loadSettings`, attempts `tryAutoConnectOnLoad`, adds the `fullscreenchange` listener, initializes p5.js (`new p5(sketch)`), and sets initial button states. 
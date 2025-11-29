# Maltilabeler – User Guide for Researchers

## 1. Introduction

Maltilabeler is a desktop tool that helps you:

- **Look at scientific images** (for example, microscopy images).
- **Draw precise markings** (rectangles and polygons) to show important structures.
- **Save these markings as high‑quality annotation data** for AI models.
- **Let AI suggest markings for you**, then check and correct them.

You do **not** need programming knowledge to use this tool.  
You mainly:

- Open a project with images.
- Draw or adjust shapes on the image.
- Optionally ask the AI to create a “first draft” of shapes.
- Save your work so the AI team can train or improve models later.

---

## 2. Interface Overview

When you open a project, the main window has four important areas.

### 2.1 Top Menu Bar

At the very top you will see menus such as:

- **File**
  - **Save Annotations**: saves all current markings for all images in the project.
  - **Export Processed Image…**: saves a copy of the current image with the visual adjustments applied.
  - **Add Images to Project…**: adds more image files into the current project.

- **Settings**
  - **Label Settings…**: open a dialog to edit label names and keyboard shortcuts.

- **View**
  - **Sharpen Image**, **Binarize Image…**, **Edge Detection…**: extra visual filters (optional; most users will use the sliders in the left panel instead).

- **AI**
  - **AI Center**: opens the AI Operation Center for training and running the AI.

You do not need to use every menu item; the basic work can be done from the left panel and the canvas.

### 2.2 Left Panel

The left side of the window contains several blocks:

- **Tools**
  - **Rectangle**: tool to draw rectangular boxes.
  - **Polygon**: tool to draw outlines with multiple points.

- **Image Adjustment Panel**
  - **Basic Adjustments**
    - **Brightness** slider: makes the whole image brighter or darker.
    - **Contrast** slider: increases or decreases the difference between light and dark areas.
  - **Detail Enhancement (CLAHE)**
    - Sliders that can make faint structures easier to see.
  - **Sharpening (Unsharp Mask)**
    - Sliders that make edges look clearer and crisper.
  - **Save Processed Image** button:
    - Saves the image exactly as you see it (with your current adjustment settings).

- **Labels**
  - A group of radio buttons (for example “ring (1)”, “core (2)”, or other names defined by your team).
  - Only one label can be active at a time.
  - The selected label is used for new shapes and for re‑labeling selected shapes.
  - At the bottom is a button **“⚙️ Label Settings…”** to change the list of labels (usually done together with your AI team).

- **Annotation List & Navigation**
  - A list that shows one line per shape on the current image (with index, label, and type).
  - **Previous (←)** button: moves to the previous image in the project.
  - **Next (→)** button: moves to the next image.
  - **AI Center** button: opens the AI Operation Center (same as the AI menu).

### 2.3 Image Canvas (Center Area)

The large area in the middle shows:

- The current image.
- Your rectangles and polygons drawn on top.

You can:

- **Zoom**: hold **Ctrl** and roll the **mouse wheel**.
- **Pan (move the view)**:
  - Press and hold the **middle mouse button**, then drag, or
  - Hold **Alt** and left‑click drag.

Shapes on the canvas can be:

- Created (with the Rectangle/Polygon tools).
- Selected and moved.
- Resized or reshaped.
- Deleted.

### 2.4 Status Bar

At the bottom, the status bar shows short messages:

- For example: “Saved project …”, “No annotations to save”, or simple warnings.

---

## 3. Basic Workflow

### Step 1 – Create or Open a Project and Load Images

1. **Start the program**  
   Run the Maltilabeler application. A **launcher window** appears.

2. **Choose project type**
   - To start from a folder of raw images:
     - Click **“Create New Project…”**.
     - Choose the folder that contains your images.
     - Choose where to store the new project folder.
     - Give the project a name.
   - To continue an existing project:
     - Click **“Open Existing Project…”** and select the project folder.

3. **Open the main window**  
   After you confirm, the main annotation window opens and shows the first image of the project.

### Step 2 – Manual Annotation (Rectangle / Polygon Tools)

First, choose your **label** and **tool**:

1. In the **Labels** box:
   - Click the label that matches the structure you want to mark (for example, “ring” or “core”).
   - If a label shows a shortcut in parentheses (e.g. “ring (1)”),
     you can press that key to select it as well.

2. In the **Tools** box:
   - Click **Rectangle** to draw rectangular boxes.
   - Click **Polygon** to draw free‑form outlines.

#### 2.1 Drawing Rectangles

Use the **Rectangle** tool to draw a box around a structure:

1. Click once near one side of the structure.
2. Drag the mouse to draw a line that becomes one side of the rectangle.
3. Release the mouse. A preview appears.
4. Move the mouse to adjust the thickness of the rectangle.
5. Click again to confirm the rectangle.

You can then:

- Click the rectangle to **select** it.
- Drag it to **move** it.
- Drag small squares at its corners and edges to **resize** it.
- Select it and press **Delete** or **Backspace** to remove it.

#### 2.2 Drawing Polygons (Free‑Form Outlines)

Use the **Polygon** tool to trace a shape with multiple points:

1. Click the first point on the boundary of the structure.
2. Continue clicking around the boundary to add more points.
3. A dashed line shows the current outline.
4. When you return near the starting point or finish the outline:
   - **Double‑click** to close and confirm the polygon (or line).

You can then:

- Click the polygon to **select** it.
- Drag it to **move** the entire shape.
- Drag small squares (handles) at the vertices to **adjust** the outline.
- **Right‑click** on a vertex to remove it (if there are enough points left).
- Select it and press **Delete** or **Backspace** to remove the shape.

#### 2.3 Changing Labels of Existing Shapes

To change the label of a shape:

1. Click the shape in the **annotation list** or on the **canvas** to select it.
2. In the **Labels** box, click the desired label.
3. The shape’s label is updated.

### Step 3 – Adjusting the Image to Make Structures Easier to See

Use the **Image Adjustment Panel** on the left to make details clearer.  
These changes only affect how the image looks on screen (and saved processed images); they do not change the raw data.

Typical adjustments:

- **Brightness**
  - Slide right to make the image lighter.
  - Slide left to make it darker.

- **Contrast**
  - Slide right to make light areas lighter and dark areas darker.
  - This can help separate structures from the background.

- **Detail Enhancement (CLAHE)**
  - Use these sliders to bring out **faint or hidden details**.
  - If the image starts to look too harsh or noisy, reduce the effect.

- **Sharpening**
  - Makes edges crisper so boundaries are easier to see.
  - Use gentle changes; very strong sharpening can look unnatural.

The image preview updates as you move the sliders.  
When you are happy with the appearance and want a copy of the adjusted image:

- Click **“Save Processed Image”** in the panel.
- Choose a file name and folder; a new image file is created with the current look.

### Step 4 – Saving and Exporting Your Work

To save all your markings:

1. Open the **File** menu.
2. Click **“Save Annotations”**.

This updates the project’s internal annotation file.  
It is good practice to save:

- After annotating several images.
- Before closing the application.

To export a visually adjusted image:

- Use either:
  - **View → Sharpen / Binarize / Edge Detection** followed by **File → Export Processed Image…**, or
  - The **“Save Processed Image”** button in the Image Adjustment Panel.

You do not need to worry about where data files are stored; your AI team can handle that.  
Just remember to **save annotations** regularly.

---

## 4. AI Assistance Features

### 4.1 Optional: AI Brush (SAM Tool)

If your version of Maltilabeler includes an extra **AI‑assisted brush** or **SAM tool**, it can help you draw outlines faster:

1. Switch to the AI brush / SAM tool (the exact button name may vary; ask your AI team if you are unsure).
2. On the canvas:
   - **Left‑click** inside the object you care about to tell the AI “this is part of the object”.
   - (Optionally) **Right‑click** on background areas to tell the AI “this is not part of the object”.
3. A soft colored region appears as a suggestion from the AI.
4. When you are satisfied with the suggested region:
   - **Double‑click** to confirm it.
   - The region becomes a normal polygon annotation that you can edit like any other shape.

You stay **in control**:

- You can still adjust, move, or delete the AI‑created shapes.
- You can combine AI‑assisted shapes with manually drawn rectangles and polygons.

### 4.2 Training an AI Model in the AI Center

The **AI Center** collects all AI‑related actions in one place.

To open it:

- Click **AI → AI Center** in the menu, or
- Click the **“AI Center”** button in the left panel.

In the AI Center, switch to the **Training Mode** tab.

1. **Select data**
   - Click **“Browse…”** next to **Master CSV File** and choose the annotation table prepared for training.
   - Click **“Browse…”** next to **Original Image Folder** and choose the folder with the original images.

2. **Check training settings**
   - **Base Model**, **Epochs**, and **Image Size** are usually set by your AI specialist.
   - In most cases, you can leave these values as they are.

3. **Start training**
   - Click **“Start Training”**.
   - The **Training Log** box at the bottom shows text messages as the process runs.
   - When training is finished, a message appears (for example: “Training process has finished.”).

The trained model file can then be used later in the **Analysis Mode** tab.

### 4.3 Running AI Analysis and Reviewing Results

In the AI Center, switch to the **Analysis Mode** tab.

1. **Choose the trained model**
   - Click **“Browse…”** next to **Trained Model (.pt):**
   - Select the model file produced during training.

2. **Choose images to analyze**
   - Click **“Browse…”** next to **Target Image/Folder for Analysis:**
   - Select either:
     - A single image file, or
     - A folder containing multiple images.

3. **Start analysis**
   - Click **“Start Analysis”**.
   - The log area will show messages as the AI processes the image(s).
   - When the analysis finishes, a **Review Window** opens automatically.

4. **Review and correct AI suggestions**
   In the Review Window:

   - The analyzed image is displayed.
   - AI‑suggested shapes are drawn on top.
   - You can:
     - Move, resize, or reshape them with the same tools as in the main window.
     - Delete them and draw new ones if needed.

5. **Save corrections for future training**
   - When you are satisfied with the corrections, click:
     - **“Save Corrections to Retraining CSV”**.
   - This saves a separate file that your AI team can use to improve the model later.
   - After saving, the Review Window closes.

---

## 5. Useful Shortcuts

Below is a list of helpful keyboard and mouse shortcuts.  
You can use the tool without memorizing them, but they can make work faster once you get used to them.

- **[View / inspection]**
  - **Hold Space**: temporarily hide colored shapes and show only the original image (“X‑ray mode”).
  - **Release Space**: show the shapes again.
  - **Ctrl + Mouse Wheel**: zoom in/out on the image.
  - **Middle Mouse Button drag** (or **Alt + Left‑click drag**): move (pan) the image.

- **[Drawing and editing]**
  - **Esc**:
    - While drawing a new rectangle or polygon: cancel the current drawing.
  - **Double‑click** with **Polygon** tool:
    - Finish the polygon or line.
  - **Arrow keys (↑ ↓ ← →)** with a shape selected:
    - Move the selected shape slightly in the chosen direction.
  - **Delete / Backspace** with a shape selected:
    - Delete the selected shape.

- **[Labels]**
  - If a label shows a key in parentheses (for example, “ring (1)”):
    - Press that key to select the label quickly.

You can always continue to work using only the mouse if you prefer; shortcuts are just there to speed up common actions.

---

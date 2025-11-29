# Maltilabeler: Human–AI Collaborative Annotation Toolkit

[![Status](https://img.shields.io/badge/status-research--grade-blue.svg)](./)
[![GUI](https://img.shields.io/badge/UI-PyQt6-41cd52.svg)](./)
[![Python](https://img.shields.io/badge/python-3.x-blue.svg)](./)
[![Build](https://img.shields.io/badge/packaging-PyInstaller-green.svg)](./)

---

## 1. Project Purpose: Human–AI Collaboration in Scientific Research

Maltilabeler is a desktop annotation toolkit designed for **scientific imaging workflows**, where domain experts and AI models must collaborate tightly rather than compete.

The core concept is:

> **“Human–AI co-creation of high‑quality labels for research data.”**

Instead of hiding the model behind a black box, Maltilabeler exposes:

- **Robust manual tools** for precise annotation.
- **Configurable pre-processing and visualization** to make subtle structures visible.
- **Integrated training & inference pipelines** that keep AI models close to the annotation loop.
- **Feedback-oriented review tools** so researchers can correct model outputs and feed them back into retraining datasets.

The application is engineered to be:

- **Production-hardened** (error handling, logging, validation, thread safety).
- **Extensible** (plugin architecture, AI-center, SAM integration).
- **Research-friendly** (CSV/YOLO dataset export, analysis plugins, retraining workflows).
- **Internationalized** (Qt `tr()` and runtime language switching).

---

## 2. Main Features (from an Engineering Perspective)

### 2.1 Application Lifecycle & Robustness

- **Central application manager (`ApplicationManager` in [app.py])**
  - Owns the `QApplication`, translators, main window, and project manager.
  - Handles startup flow:
    - Initialize Qt and logging.
    - Run a **launcher dialog** to either create or open a project.
    - Create and validate a `ProjectManager` instance.
    - Construct the `MainWindow`.
    - Trigger initial project/image load in a controlled way.
  - Encapsulates:
    - Comprehensive **exception handling** (including fatal errors).
    - Centralized **logging** to both file and console.
    - **Cleanup** logic for main window, project manager, translators, and resources.

- **Internationalization (i18n)**
  - Uses `QTranslator`, `QLocale`, and Qt translation files (`translations/*.qm`).
  - User-facing strings are wrapped via `self.tr(...)` in UI classes.
  - `ConfigManager` stores the language code and `ApplicationManager` configures translators accordingly.
  - Qt’s own dialogs are localized using Qt system translation packs.

- **Production-style logging & error dialogs**
  - Global logging configuration ([maltilabeler.log]) with timestamps, module names, and source locations.
  - Graceful fallback to console output if GUI error dialogs cannot be shown.
  - Detailed logging of initialization steps, resource loading, and cleanup.

---

### 2.2 Project Management Layer

- **`ProjectManager` ([logic/project_manager.py])**
  - Inherits from `QObject` and communicates with the UI via Qt signals:
    - `project_loaded(project_name: str)`
    - `image_changed(image_path: str, annotations: list)`
    - `status_updated(message: str)`
    - `error_occurred(message: str)`
  - Manages:
    - Project paths (`project_path`, `images_path`, `output_path`, `annotations_csv_path`).
    - Image list and current index.
    - Per-image annotation cache (`session_annotations`).
    - Per-image image-adjustment parameters.

- **Validation and safety**
  - Extensive checks for:
    - Project folder validity and structure (presence of `images/`).
    - File system permissions and path lengths (Windows path limit awareness).
    - Supported image extensions and non-empty files.
    - Maximum counts for images and annotations per image.
  - Uses `QMutex` / `QMutexLocker` to make critical sections thread-safe.

- **CSV I/O (annotation persistence)**
  - Saves annotation sessions to CSV using **pandas**:
    - Validates annotation structure before writing.
    - Handles backup/restore if a save fails.
    - Serializes annotation geometry into JSON strings for robustness.
  - Designed for interoperability with downstream ML training pipelines.

- **Project creation workflow**
  - Class method [create_new_project(details)]:
    - Validates input paths and project name.
    - Creates a project folder tree (`images/`, [output/]).
    - Safely copies and filters source images.
    - Cleans up on failure.
  - Lazy imports of heavy modules (`shutil`, `pandas`) to keep startup light.

---

### 2.3 Annotation & Interaction Layer

- **`DrawingItem` ([logic/drawing_item.py])**
  - A custom `QGraphicsItem` that:
    - Holds an internal list of annotations (dictionaries with type/label/geometry).
    - Tracks selection state (`selected_item_index` property with propagation to the UI).
    - Validates annotation structures and geometry before use.
  - Maintains strict **geometric validation**:
    - Checks polygon vertex structures and coordinates versus the image bounds.
    - Ensures oriented boxes (OBB) have coherent parameters and minimal sizes.

- **Tool architecture ([logic/tools/])**
  - `BaseTool` as an abstract base for all annotation tools:
    - Defines the template methods for mouse events and paint operations.
  - Concrete tools:
    - **SelectionTool**: Selection, dragging, editing of existing annotations.
    - **OBBTool**: Creation/adjustment of oriented bounding boxes.
    - **PolygonTool**: Polygon and line-based annotations (including 2-point “lines” reused by plugins).
    - **SAMTool**: SAM-based interactive segmentation driven by point prompts.
  - The `DrawingItem` keeps a registry of tools and switches modes (`set_tool_mode`) without leaking UI concerns into tool implementations.

- **User interaction quality**
  - Handles hover events, keyboard shortcuts, and cursor feedback.
  - Enforces coordinate clamping and validity to avoid invalid geometry.
  - Provides x-ray modes, preview overlays, and handle-based resizing/vertex editing.

---

### 2.4 Main Window & UI Composition

- **`MainWindow` ([ui/main_window.py])**
  - Central hub that binds `ProjectManager`, `DrawingItem`, `ZoomableView`, and plugins.
  - Uses:
    - **QMutex** for UI state during long operations.
    - A consistent left-panel layout (tools, image adjustment, labels, navigation, AI shortcuts) and a main graphics view.
  - Key UI components:
    - **Tool box** with radio buttons for OBB vs Polygon vs (optionally) SAM mode.
    - **ImageAdjustmentPanel** (see below).
    - **Labels group**: dynamically populated from `ConfigManager` with radio buttons and keyboard shortcuts.
    - Annotation list and navigation buttons for cycling through images.
  - Connects signals:
    - From `ProjectManager` (`project_loaded`, `image_changed`, `status_updated`).
    - From canvas (`canvas_selection_changed`) to sync list selection.
    - From label radios and keyboard shortcuts to update annotation labels.

- **Configuration-driven label system**
  - `ConfigManager` ([logic/config_manager.py]) manages:
    - A JSON config ([config.json]) with label definitions and keyboard shortcuts.
    - Persistent language preference (`language`).
  - `MainWindow.update_ui_from_config()`:
    - Rebuilds label radio buttons at runtime from config.
    - Binds per-label keyboard shortcuts for fast expert workflows.
    - Adds a “Label Settings…” button that opens a dedicated label editor dialog.

- **Image adjustment pipeline**
  - `ImageAdjustmentPanel` ([ui/image_adjustment_panel.py]) exposes a GUI for:
    - Brightness/contrast.
    - CLAHE.
    - Unsharp masking and related enhancements.
  - The panel emits `parameters_changed` and `save_requested` signals:
    - `MainWindow` calls [apply_image_adjustments()] in [logic/image_processing.py].
    - Logic is separated cleanly from UI: panel is purely parameter/UI, processing is in [logic/].

---

### 2.5 Image Processing & Computer Vision Utilities

- **[logic/image_processing.py]**
  - Encapsulates image processing as **pure functions on `QPixmap`**:
    - Unsharp masking.
    - Adaptive binarization.
    - Edge detection.
    - CLAHE.
    - Combined color pipeline using LAB space with brightness/contrast + CLAHE + unsharp mask.
  - Uses **OpenCV (`cv2`) + NumPy**:
    - Efficient conversion between `QImage` and `numpy` arrays.
    - Strict handling of image formats and strides.
  - Design choices:
    - No stateful global pipeline; functions are composable utilities.
    - Lazy imports for heavy libraries inside functions to keep import costs low.
    - Returns **new `QPixmap` objects** instead of mutating inputs, improving safety.

- **`ZoomableView` ([logic/zoomable_view.py])**
  - A custom `QGraphicsView` implementation:
    - Smooth zooming and panning for large, high-resolution scientific images.
    - Keeps aspect ratio and scene bounds under control for reliable usability.

---

### 2.6 AI Center: Training & Inference Flows

- **[AICenterWindow] ([ui/ai_center_window.py])**
  - A tabbed dialog that centralizes:
    - **Training Mode**: dataset creation + YOLO training orchestration.
    - **Analysis Mode**: running inference on a trained YOLO model and reviewing results.
  - Uses QThreads and worker objects to keep the UI responsive:
    - [TrainingWorker] (dataset preparation + external training command).
    - [AnalysisWorker] (model inference and OBB result extraction).

- **Training pipeline ([logic/training_manager.py])**
  - [TrainingWorker]:
    - Consumes a master CSV and an image folder produced by Maltilabeler.
    - Builds a YOLO-oriented dataset structure (images/labels split).
    - Writes **OBB-style** label files from the stored annotation geometries.
    - Generates a `data.yaml` for YOLO, including the class name mapping.
    - Calls the `yolo` CLI in a subprocess for training, streaming logs back to the UI.
  - Design highlights:
    - Separation between **dataset preparation** and **training command orchestration**.
    - Clear communication via Qt signals (`log_updated`, `finished`).
    - Uses PIL for robust image-size handling and path validation.

- **Inference pipeline ([logic/analysis_manager.py])**
  - [AnalysisWorker]:
    - Lazily imports **Ultralytics YOLO** and PyTorch only when needed.
    - Loads a user-specified model file and runs prediction on a target image.
    - Extracts oriented bounding boxes (OBB) and class labels into a serializable structure.
    - Emits both logs and `analysis_complete(image_path, boxes_list)` back to the UI.

- **Review & correction ([ui/review_window.py])**
  - [ReviewWindow]:
    - Reuses the same `ZoomableView` + `DrawingItem` combination used in the main UI.
    - Displays model predictions as editable annotations.
    - Offers a single-purpose button to **append corrected annotations** to a dedicated **retraining CSV** (`output/retraining_data.csv`).
  - This creates an explicit **human-in-the-loop feedback loop**:
    - Train → Predict → Human corrects → Append corrections → Retrain.

---

### 2.7 SAM Integration (Segment Anything)

- **[SAMManager]([logic/sam_manager.py])**
  - Manages:
    - Model checkpoint download (if missing).
    - Device selection (CUDA vs CPU) without impacting application startup.
    - Lazy import of **Segment Anything** and PyTorch.
  - Exposes:
    - [set_image(image_path)] to compute embeddings once per image.
    - [predict(points, labels)] to obtain masks from point prompts.

- **[SAMTool] ([logic/tools/sam_tool.py])**
  - A `BaseTool` subclass that:
    - Collects positive/negative prompt points on the canvas.
    - Requests masks from [SAMManager].
    - Renders preview masks as filled polygons using OpenCV contour extraction.
    - On double-click, converts the preview mask into a standard polygon annotation managed by `DrawingItem`.

This is a good example of **encapsulating a heavyweight AI model behind a clean, interactive tool interface**.

---

### 2.8 Plugin Architecture & Advanced Analysis

- **Plugin interface ([logic/plugin_interface.py])**
  - Defines base plugin classes used by the application:
    - Basic `PluginInterface` with [name], [description], [initialize()], and [execute()] hooks.
    - Extended plugin base (`MaltilabelerPlugin`, used in advanced plugins) that integrates with the main window and menu structure.

- **`PluginManager` ([logic/plugin_manager.py])**
  - Dynamically discovers Python modules in [plugins/].
  - Checks for subclasses of the plugin base interface and instantiates them.
  - Provides:
    - `discover_and_load_plugins()`
    - `add_plugin_menus(menu_bar)`
    - `notify_image_loaded(image_path)` and `notify_annotation_changed(annotations)` hooks.
    - A programmatic `register_plugin(plugin_instance)` API for internal plugins.

- **Example plugins**

  - **OtolithAnalysisPlugin ([plugins/otolith_counter_plugin.py])**
    - Demonstrates how domain-specific analysis can be implemented as a plugin.
    - Uses existing polygon annotations as scale bars and analysis axes.
    - Converts geometric distances into real-world units using user-supplied calibration.
    - Exports results to CSV for downstream statistical analysis.
    - Example of a **modeless, two-step scientific workflow** embedded into the annotation environment.

  - **FeasibilityAnalyzerPlugin ([plugins/feasibility_analyzer.py])**
    - Runs a project feasibility study on an existing CSV of annotations:
      - Feature extraction using **torch + torchvision (ResNet)**.
      - Dimensionality reduction and visualization with **t‑SNE** (scikit-learn + matplotlib).
      - Learning-curve estimation using scikit-learn’s `learning_curve`.
    - Uses a background worker ([FeasibilityWorker]) with:
      - Lazy imports of heavy libraries.
      - Structured logging, progress updates, and plot generation (saved to [output/]).
    - Provides a multi-tab dialog (log, t‑SNE plot, learning curve) fully decoupled from the main UI.

These plugins showcase how the **core annotation platform can be extended with sophisticated AI and statistical analyses** without entangling them with the base application.

---

## 3. Technology Stack

### 3.1 Programming Language & Runtime

- **Python**
  - Modern CPython 3.x (uses type hints, f‑strings, and contemporary libraries).

### 3.2 GUI & Desktop Framework

- **PyQt6**
  - `PyQt6.QtWidgets`, `QtGui`, `QtCore`, `QtSvg`
  - Custom widgets:
    - `MainWindow`, [LauncherDialog], [AICenterWindow]
    - `ImageAdjustmentPanel`, various parameter dialogs (Unsharp/CLAHE/Binarization/Edge Detection)
    - [ReviewWindow]
    - `NewProjectDialog` and other project/UI helpers.
  - Uses:
    - `QGraphicsScene` / `QGraphicsItem` for canvas.
    - `QDialog`, `QTabWidget`, `QGroupBox`, `QFormLayout` for structured dialogs.
    - `QThread` + QObject workers for background AI and training tasks.
    - `QTranslator` and `QLibraryInfo` for localization.

### 3.3 Core Scientific & Image Processing Libraries

- **OpenCV (`cv2`)**
  - Image filtering, edge detection, CLAHE, geometric operations, and mask handling.
  - Interoperability with Qt via `QImage` ↔ `numpy.ndarray` conversions.

- **NumPy (`numpy`)**
  - Array manipulation for image data and AI-related intermediate representations.

- **Pillow (`PIL`)**
  - Robust image opening and size extraction in the training dataset builder.

### 3.4 Machine Learning & AI Libraries

- **Ultralytics YOLO (`ultralytics`)**
  - Training: via the `yolo` CLI (OBB mode).
  - Inference: `YOLO` class used in [AnalysisWorker] to run prediction and obtain OBB outputs.

- **PyTorch (`torch`)**
  - Device management and tensor operations for:
    - Segment Anything (SAM).
    - ResNet-based feature extraction in the Feasibility Analyzer plugin.

- **Torchvision (`torchvision`)**
  - Pretrained ResNet backbone and associated preprocessing transforms.

- **Segment Anything (`segment_anything`)**
  - `sam_model_registry`, `SamPredictor` for interactive segmentation through [SAMManager] and [SAMTool].

- **scikit-learn (`sklearn`)**
  - `TSNE` for t‑SNE visualizations.
  - `learning_curve` and `SGDClassifier` for learning-curve analysis in feasibility studies.

### 3.5 Data Science & Visualization

- **pandas**
  - Reading/writing annotation CSV files.
  - Grouping and transforming data for YOLO dataset generation and analysis plugins.

- **Matplotlib (`matplotlib`, `matplotlib.pyplot`)**
  - Generates t‑SNE scatter plots.
  - Generates learning-curve figures.
  - Uses a non-interactive backend for headless plot generation, saving to [output/].

- **PyYAML (`yaml`)**
  - Writes YOLO’s `data.yaml` configuration file from the training manager.

### 3.6 Packaging & Deployment

- **PyInstaller**
  - [maltilabeler.spec] provides a **one-file** build configuration:
    - Includes [plugins/], [translations/], and [models/] as data.
    - Declares hidden imports (PyQt6 submodules, `cv2`, `pandas`, `numpy`).
  - Target: single executable suitable for end users (e.g., researchers without Python experience).

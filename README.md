# Envista Turntable Inspection System

This repository contains the complete **C# WinForms** solution for the Envista turntable automated inspection system.
It serves as the source-of-truth for engineering changes; production binaries are published separately in [`Envista_turntable_stable`](https://github.com/tefj-fun/Envista_turntable_stable).

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Repository Structure](#repository-structure)
3. [Key Features](#key-features)
4. [Prerequisites](#prerequisites)
5. [Building & Running](#building--running)
6. [Hardware Setup](#hardware-setup)
7. [Workflow Guide](#workflow-guide)
8. [Logic Builder](#logic-builder)
9. [Hardware Integration](#hardware-integration)
10. [Development Guidelines](#development-guidelines)
11. [Release Process](#release-process)
12. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

The DemoApp is an automated visual inspection system that coordinates multiple hardware components to detect and classify defects on rotating parts:

### Core Components

- **SolVision AI Models**
  - **Attachment Detection Model** (top camera): Identifies attachment points on parts for inspection
  - **Front Attachment Model** (front camera): Additional attachment detection from front view
  - **Defect Detection Model** (front camera): Classifies defects on each attachment point

- **Hardware Integration**
  - **ComXim Turntable**: Precision rotational positioning with homing capability
  - **Huaray Cameras**: USB top camera + GigE front camera for multi-angle capture
  - **Serial Communication**: COM-based turntable control protocol

- **User Interface**
  - **Initialize Tab**: Hardware/model setup and configuration
  - **Workflow Tab**: Inspection execution, results review, and defect analysis
  - **Logic Builder**: Custom rule-based pass/fail criteria

### Key Libraries

| Library | Purpose |
|---------|---------|
| `Solvision.dll` | SolVision SDK for AI inference (ExecuteType.Dll workflow) |
| `MVSDK_Net.dll` | Huaray camera SDK for image acquisition |
| `Emgu.CV` | OpenCV wrapper for image processing and overlay rendering |
| `Newtonsoft.Json` | JSON serialization for results export |
| Custom `TurntableController` | Serial protocol implementation for turntable commands |

---

## Repository Structure

```
DemoApp/
├── DemoApp.sln                    # Visual Studio solution file (use this one)
├── build.cmd                      # Windows batch build script
├── build.ps1                      # PowerShell build script
├── build.sh                       # WSL/Linux build wrapper
├── README.md                      # This file
├── .gitignore                     # Git ignore rules
├── .gitattributes                 # Git LFS configuration
├── .vscode/                       # VS Code settings
├── .claude/                       # Claude Code configuration
└── DemoApp/                       # Main project directory
    ├── DemoApp.csproj             # C# project file
    ├── DemoApp.cs                 # Main form implementation
    ├── DemoApp.Designer.cs        # Form designer code
    ├── DemoApp.resx               # Form resources
    ├── Program.cs                 # Application entry point
    ├── LoadingForm.cs             # Splash screen
    ├── App.config                 # Application configuration
    ├── AppIcon.ico                # Application icon
    ├── Properties/                # Assembly info and resources
    ├── Dependencies/Huaray/       # Huaray camera SDK DLLs
    ├── dll/                       # SolVision native support DLLs
    ├── images/                    # UI icons and assets
    └── bin/, obj/                 # Build output (git-ignored)
```

### Important Notes

- **DO NOT USE** `DemoApp/DemoApp.sln` - this is a duplicate with incorrect paths
- Build artifacts (`bin/`, `obj/`, `Captured/`) are git-ignored
- Runtime data is saved to `Runs/` directory with timestamped folders
- The project expects to be located under the SolVision installation directory

### Recommended Installation Path

```
C:\Program Files\Solomon Technology Corp\SolVision6\Version_6.1.4\SampleCode\C_Sharp\DemoApp\
    DemoApp.sln                    # Use this solution file
    DemoApp/DemoApp.csproj
    DemoApp/Dependencies/Huaray/...
```

If installing elsewhere, update DLL hint paths in [DemoApp.csproj](DemoApp/DemoApp.csproj) (search for `..\..\..\..\Solvision.dll` references).

---

## Key Features

### Automated Inspection Workflow
1. **Top-Down Detection**: Capture overhead image to identify attachment points
2. **Rotational Capture**: Automatically rotate turntable to each attachment point
3. **Front Inspection**: Capture and analyze front view for each attachment
4. **Defect Classification**: AI-based defect detection with confidence scores
5. **Pass/Fail Logic**: Customizable rule-based evaluation system

### Data Management
- **Timestamped Runs**: Each inspection creates a dated folder with all captured images
- **JSON Export**: Attachment detection results exported for integration
- **CSV Summaries**: Defect summary reports in CSV format
- **Debug Visualization**: Cropped images and debug overlays for troubleshooting

### Advanced Features
- **Recorded Run Playback**: Re-analyze previously captured data without hardware
- **Custom Logic Rules**: Build complex AND/OR rule trees for pass/fail criteria
- **Multi-Model Support**: Separate models for attachment detection and defect classification
- **Real-time Overlay**: Visual feedback with detection boxes, labels, and confidence scores
- **Cycle Time Tracking**: Performance monitoring for production optimization

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Operating System** | Windows 10/11 (x64) |
| **IDE** | Visual Studio 2022 (v17.6+) with .NET desktop development workload |
| **.NET Framework** | .NET Framework 4.8 SDK (included with VS workload) |
| **SolVision SDK** | SolVision 6.1.4 runtime with valid license |
| **Camera Drivers** | Huaray MV SDK (USB & GigE drivers) |
| **Turntable Driver** | ComXim turntable USB/Serial driver |
| **Version Control** | Git + Git LFS (for binary asset management) |
| **GPU (optional)** | CUDA-capable NVIDIA GPU for accelerated inference |

### Pre-Build Checklist
1. Install Visual Studio 2022 with ".NET desktop development" workload
2. Install SolVision 6.1.4 and activate license
3. Install Huaray MV SDK and verify camera connectivity with "MV Viewer" tool
4. Install ComXim turntable driver and note COM port assignment
5. Verify GPU drivers are current if using CUDA acceleration

---

## Building & Running

### Visual Studio Build

1. Open [DemoApp.sln](DemoApp.sln) (root directory) in Visual Studio
2. Select build configuration:
   - **Debug | x64** - Development with full debugging symbols
   - **Release | x64** - Production build with optimizations
3. Build the solution: `Build` → `Build Solution` (or Ctrl+Shift+B)
4. Run: `Debug` → `Start Debugging` (F5) or `Start Without Debugging` (Ctrl+F5)

### Command-Line Build

#### Windows (cmd)
```cmd
build.cmd Debug x64
```

#### Windows (PowerShell)
```powershell
.\build.ps1 -Configuration Release -Platform x64
```

#### WSL/Linux
```bash
./build.sh Release x64
```

### Build Output
- Debug: `DemoApp/bin/x64/Debug/DemoApp.exe`
- Release: `DemoApp/bin/x64/Release/DemoApp.exe`
- Stable: `DemoApp/bin/x64/Stable/` (manual copy for deployment)

---

## Hardware Setup

### 1. SolVision Licensing
Ensure the machine has a valid SolVision 6.1.4 license. Check license status in SolVision software before running.

### 2. Turntable Connection
1. Connect ComXim turntable via USB
2. Install driver and note COM port (e.g., COM3)
3. Verify connection in Device Manager under "Ports (COM & LPT)"

### 3. Camera Setup

#### Top Camera (USB)
- Connect Huaray USB camera
- Launch "MV Viewer" to verify live feed
- Note camera model/serial number

#### Front Camera (GigE)
- Connect Huaray GigE camera to dedicated NIC
- Configure network adapter:
  - Static IP: 169.254.1.100
  - Subnet: 255.255.0.0
  - Camera IP: 169.254.1.10 (verify in MV Viewer)
- Verify streaming in "MV Viewer" before proceeding

### 4. GPU Configuration (Optional)
- Install latest NVIDIA drivers for CUDA support
- Verify GPU detection in SolVision software
- Monitor GPU utilization during inference for performance tuning

---

## Workflow Guide

### Initialize Tab

#### Step 1: Load AI Models
1. **Attachment Model**
   - Click "Browse" next to "Attachment .tsp Path"
   - Select trained attachment detection model (.tsp file)
   - Click "Load" - status indicator turns green on success

2. **Front Attachment Model** (optional)
   - Load secondary attachment model for front view
   - Used for additional validation if needed

3. **Defect Model**
   - Load trained defect classification model (.tsp file)
   - This model classifies defects found on each attachment point

**Status Indicators:**
- Gray: Not loaded
- Green: Successfully loaded
- Yellow/Red: Loading error (check log panel)

#### Step 2: Connect Cameras
1. Click "Refresh Cameras" to scan for devices
2. Select top camera from dropdown (USB device)
3. Click "Connect Top Camera"
4. Select front camera from dropdown (GigE device)
5. Click "Connect Front Camera"
6. Test each camera with "Capture Preview"

#### Step 3: Connect Turntable
1. Select COM port from dropdown
2. Click "Connect Turntable"
3. Click "Home Turntable" - **REQUIRED before first use**
4. Wait for homing sequence to complete (TB_END response)
5. Verify offset angle is logged

#### Step 4: Begin Workflow
Once all three components show green status:
1. Click "Begin Workflow" to switch to Workflow tab
2. System is now ready for automated inspection

### Workflow Tab

#### Running an Inspection

1. **Enter Part ID** (optional)
   - Type unique identifier in "Part ID" field
   - Used for folder naming and traceability

2. **Top Detection**
   - Click "Detect Attachments"
   - Top camera captures overhead image
   - AI model identifies attachment points
   - Overlay shows center marker and sequence numbers
   - Attachment count displayed

3. **Automatic Capture Sequence**
   - Click "Start Inspection Sequence"
   - For each attachment point:
     - Turntable rotates to position
     - Front camera captures image
     - Image saved to `Runs/<Date>/<PartID>/Front/`
   - Turntable returns home
   - All images processed with defect model
   - Results displayed in gallery

4. **Review Results**
   - **Gallery View**: Thumbnail cards with PASS/FAIL badges
   - **Overlay View**: Click any thumbnail to see detailed overlay
   - **Defect Ledger**: Table of all detections with class, confidence, area, bbox

5. **Pass/Fail Evaluation**
   - Logic rules automatically evaluated
   - Status banner shows overall result (green PASS / red FAIL)
   - Triggered rule displayed if failure detected

#### Recorded Run Mode
To re-analyze previous inspection data without hardware:
1. Check "Use Recorded Run"
2. Click "Select Folder"
3. Choose existing run directory (must contain Front/ and Top/ folders)
4. "Detect Attachments" and "Start Inspection" work with saved images

### Run Output Structure

Each inspection creates a timestamped folder:

```
Runs/
  2025-11-04_143052_PartID/          # Date_Time_PartID format
    ├── Top/
    │   ├── Top.png                  # Original top-down capture
    │   └── TopAttachments.json      # Attachment detection export
    ├── Front/
    │   ├── PartID_idx_01.png        # Front capture for each attachment
    │   ├── PartID_idx_02.png
    │   └── FrontAttachments.json    # Front attachment export (if enabled)
    ├── Front_Crop/                  # Cropped regions for each detection
    │   ├── PartID_idx_01_crop.png
    │   └── PartID_idx_01_debug_mat.png  # Debug visualization
    └── Results/
        └── DefectSummary.csv        # All defects in CSV format
```

**CSV Format:**
```csv
Run,PartID,InspectionIndex,Class,Confidence,Area,BoundingBox
2025-11-04_143052,PartID,1,Scratch,0.95,1234,"(100,200,50,75)"
```

---

## Logic Builder

The Logic Builder allows custom pass/fail criteria based on defect detection results.

### Rule Types

#### Individual Rules
- **Field**: Class Name, Confidence, Area, or Count
- **Operator**: `==`, `!=`, `>`, `>=`, `<`, `<=`
- **Value**: Comparison value (auto-suggested from current inspection data)

#### Rule Groups
- **ALL (AND)**: All child rules must be true
- **ANY (OR)**: At least one child rule must be true

### Creating Logic

1. Switch to "Logic Builder" tab
2. Right-click logic tree or use buttons:
   - "Add Rule" - Create comparison rule
   - "Add Group" - Create AND/OR group
3. Configure rule:
   - Select field (e.g., "Class Name")
   - Select operator (e.g., "==")
   - Select/enter value (e.g., "Scratch")
4. Nest rules in groups for complex logic

### Example Logic

**Scenario**: Fail if any "Crack" defects OR more than 2 "Scratch" defects

```
Root (ANY)
  ├── Rule: Class Name == "Crack"
  └── Rule: Count > 2 AND Class Name == "Scratch"
```

### Evaluation Behavior
- Logic evaluates **after** all front inspections complete
- If **any** rule/group evaluates TRUE, inspection **FAILS**
- Workflow status banner shows triggered rule
- If no rules defined, defaults to PASS

### Implementation Details
```csharp
// Evaluation entry point in DemoApp.cs
bool fail = EvaluateNode(logicRoot, CollectCurrentDefectDetections(), out LogicNodeBase triggered);

// Data collection from all front inspections
Dictionary<string, List<Detection>> detections = CollectCurrentDefectDetections();
```

---

## Hardware Integration

### Turntable Controller

The `TurntableController` class manages serial communication with the ComXim turntable.

#### Key Commands
```csharp
// Homing (required before first use)
turntableController.SendCommand("CT+HOME();");

// Rotate to absolute angle
turntableController.SendCommand("CT+START(45.0);");  // Rotate to 45 degrees

// Get current offset angle (after homing)
turntableController.SendCommand("CT+GETOFFSETANGLE();");

// Speed adjustment (1 = fastest, 5 = slowest)
turntableController.SendCommand("CT+CHANGESPEED(1);");
turntableController.SendCommand("CT+GETTBSPEED();");  // Query speed
```

#### Response Handling
```csharp
turntableController.MessageReceived += (sender, message) => {
    if (message.Contains("TB_END")) {
        // Rotation complete
    }
    if (message.Contains("CR+ERR")) {
        // Command error
    }
};
```

### Camera Context

The `CameraContext` class wraps Huaray SDK operations.

```csharp
// Initialize camera
CameraContext topCamera = new CameraContext(CameraRole.Top);
topCamera.Connect(deviceHandle);

// Capture frame
Bitmap frame = topCamera.CaptureCameraFrame();

// Cleanup
topCamera.Dispose();
```

#### Camera Roles
- `CameraRole.Top`: USB camera for overhead attachment detection
- `CameraRole.Front`: GigE camera for front defect inspection

### Angle Calculations

Turntable angle math (ported from `angle_calculation.py`):

```csharp
// Normalize angle to [-180, 180]
float NormalizeSignedAngle(float angle) {
    while (angle > 180f) angle -= 360f;
    while (angle < -180f) angle += 360f;
    return angle;
}

// Calculate rotation angle from center point
float CalculateAttachmentAngle(PointF center, PointF attachment) {
    float deltaX = attachment.X - center.X;
    float deltaY = center.Y - attachment.Y;  // Invert Y axis
    return (float)(Math.Atan2(deltaY, deltaX) * 180.0 / Math.PI);
}
```

---

## Development Guidelines

### Code Style
- **Target Framework**: .NET Framework 4.8 (no .NET 5+ APIs)
- **Naming Conventions**:
  - PascalCase for public methods/properties
  - camelCase for local variables/private fields
  - UPPER_CASE for constants
- **Formatting**: Use Visual Studio "Format Document" (Ctrl+K, Ctrl+D)

### Threading Best Practices
- Heavy operations (detection, turntable moves) run on background threads
- UI updates **must** use `BeginInvoke`:
  ```csharp
  BeginInvoke(new Action(() => {
      LBL_Status.Text = "Updated from background thread";
  }));
  ```
- Use `CancellationToken` for long-running operations

### Logging
Use `outToLog()` helper with status enum:
```csharp
outToLog("Operation started", LogStatus.Info);
outToLog("Success!", LogStatus.Success);
outToLog("Warning: Low confidence", LogStatus.Warning);
outToLog("Error: Camera disconnected", LogStatus.Error);
outToLog("Processing...", LogStatus.Progress);
```

### Resource Management
Always dispose unmanaged resources:
```csharp
using (Bitmap frame = camera.CaptureCameraFrame()) {
    // Process frame
} // Automatic disposal

Mat mat = image.ToMat();
try {
    // Process mat
} finally {
    mat.Dispose();
}
```

### Error Handling
- User-facing errors should suggest corrective actions
- Log detailed exceptions for debugging
- Gracefully degrade when hardware unavailable

### Testing Checklist
- [ ] Test with all hardware connected
- [ ] Test with recorded run (no hardware)
- [ ] Test logic builder with various rule combinations
- [ ] Test with missing models (error handling)
- [ ] Test with camera disconnection during capture
- [ ] Test turntable timeout scenarios
- [ ] Verify memory cleanup (no leaks after multiple runs)

### AI Assistant Guidelines

When using AI coding tools (GitHub Copilot, Claude Code, ChatGPT, etc.):

1. **Repository Discipline**
   - All source changes go in **this** repository
   - Never modify the binary release repository directly

2. **Code Review**
   - Carefully review all AI-generated code
   - Add explanatory comments for complex logic
   - Document assumptions and limitations

3. **Hardware Safety**
   - Extra scrutiny for turntable motion code
   - Verify camera driver interactions
   - Test asynchronous operations thoroughly

4. **Style Consistency**
   - Match existing code patterns
   - Run Format Document before committing
   - Follow naming conventions

5. **Commit Hygiene**
   - Use descriptive commit messages
   - Reference issues/tickets when applicable
   - Create pull requests for review

6. **Refactoring**
   - Discuss large refactors with team first
   - Avoid breaking existing workflows
   - Maintain backward compatibility when possible

---

## Release Process

### 1. Development Workflow
1. Create feature branch from `main`
2. Implement changes with tests
3. Commit with descriptive messages
4. Push branch and create pull request
5. Code review and approval
6. Merge to `main`

### 2. Release Build
1. Checkout `main` branch
2. Update version in `Properties/AssemblyInfo.cs`
3. Build `Release | x64` configuration
4. Test full workflow with real hardware
5. Verify no regressions

### 3. Package for Deployment
```powershell
# Copy Release build to Stable directory
Copy-Item ".\DemoApp\bin\x64\Release\*" ".\DemoApp\bin\x64\Stable\" -Recurse -Force

# Package for binary repository
Copy-Item ".\DemoApp\bin\x64\Stable\*" "..\Envista_turntable_stable\" -Recurse -Force
```

### 4. Binary Repository Update
1. Navigate to `Envista_turntable_stable` repository
2. Update README if behavior changed
3. Commit with release notes
4. Push to remote
5. Create GitHub Release/tag (e.g., `v1.2.0`)

### 5. Documentation
1. Update main README if needed
2. Update CHANGELOG.md with changes
3. Notify team via Slack/email

### 6. Rollback Plan
- Keep previous Stable build archived
- Document rollback procedure
- Test rollback before critical deployments

---

## Troubleshooting

### Build Issues

| Problem | Solution |
|---------|----------|
| `SolVision.dll` not found | Install SolVision 6.1.4 SDK; verify hint path in DemoApp.csproj |
| `MVSDK_Net.dll` missing | Install Huaray MV SDK; check `Dependencies/Huaray/Runtime/` directory |
| MSBuild not found | Install Visual Studio 2022 with .NET desktop development workload |
| Platform target mismatch | Ensure `x64` platform is selected (not AnyCPU or x86) |

### Runtime Issues

| Problem | Solution |
|---------|----------|
| Camera not detected | 1. Check USB/Ethernet connection<br>2. Test in MV Viewer<br>3. Reinstall drivers<br>4. Check Device Manager for errors |
| Turntable timeout | 1. Verify COM port in Device Manager<br>2. Check serial cable connection<br>3. Close other apps using COM port<br>4. Try `CT+GETOFFSETANGLE()` test |
| App crashes on detection | 1. Verify .tsp model compatibility<br>2. Check SolVision license status<br>3. Update GPU drivers<br>4. Check log for specific errors |
| Out of memory errors | 1. Dispose images after use<br>2. Check for memory leaks in loops<br>3. Restart application periodically<br>4. Increase pagefile size |

### Model Issues

| Problem | Solution |
|---------|----------|
| Low confidence scores | 1. Retrain model with more data<br>2. Adjust lighting conditions<br>3. Check camera focus<br>4. Verify calibration |
| Missing detections | 1. Lower confidence threshold in model<br>2. Check image quality<br>3. Verify part positioning<br>4. Add more training samples |
| False positives | 1. Increase confidence threshold<br>2. Refine training dataset<br>3. Add negative samples<br>4. Review class definitions |

### Git Issues

| Problem | Solution |
|---------|----------|
| Large file rejection | Use Git LFS: `git lfs track "*.dll"` then recommit |
| Merge conflicts | Resolve manually; test build after merging |
| Slow clone/pull | Verify Git LFS is installed and configured |

### Support Contacts

- **SolVision SDK**: Solomon Technology Corp support
- **Camera Drivers**: Huaray technical support
- **Turntable**: ComXim hardware support
- **Application**: Desktop Vision/Envista inspection team

---

## Files That Should Be Removed

The following files are unnecessary and can be deleted:

1. **`DemoApp/DemoApp.sln`** - Duplicate solution file with incorrect paths (use root `DemoApp.sln` instead)
2. **`DemoApp/DemoApp.cs.backup`** - Backup file no longer needed
3. **Old test data in `bin/Debug/Runs/` and `bin/x64/Debug/Runs/`** - Keep `Runs/` structure but remove old runs

---

## Additional Resources

- [SolVision Documentation](https://www.solomon-3d.com/download/)
- [Huaray SDK Manual](https://www.huaray.com/en/support/)
- [.NET Framework 4.8 API Reference](https://docs.microsoft.com/en-us/dotnet/api/)
- [Emgu CV Documentation](http://www.emgu.com/wiki/index.php/Main_Page)

---

**Last Updated**: 2025-11-10
**Maintainer**: Envista Inspection Team
**License**: Proprietary - Solomon Technology Corp

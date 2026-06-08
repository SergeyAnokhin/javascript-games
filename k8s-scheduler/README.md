# Kubernetes Scheduler Telemetry Simulator & Game

An interactive, gamified simulation of the Kubernetes (`kube-scheduler`) placement engine. Built entirely using vanilla HTML5, CSS3, and JavaScript, it features a sci-fi cyberpunk telemetry dashboard design, procedural audio effects, real-time YAML manifest generation, and two distinct game modes: **Sandbox Mode** (free simulation) and **Quiz-10 Mode** (placement logic puzzle).

---

## 🎮 Game Modes

### 1. Sandbox Mode (Free Simulation)
A sandbox environment for cluster exploration, capacity planning, and debugging.
* **Full Interactivity**: You can adjust all parameters of both the incoming Pod and the cluster Nodes (zones, disk preferences, taints, capacities, and background workloads) at any time.
* **Auto-Schedule Live Preview**: Toggling the **Auto-Schedule Preview** checkbox runs the scheduler instantly in the background as you adjust sliders or toggle settings:
  - **Borders & Status LEDs** update in real-time (Green for passed predicates, Red for failed predicates, Pulsing Cyan for the calculated winner node).
  - **Resource Preview Bars** (Cyan/Red segments) overlay the resource meters to show potential load if scheduled.
  - **Score Badges** display the priority points breakdown for each node.
  - **Telemetry Console** prints detailed scheduling logs instantly.
  - Unchecking the preview hides all indicators, showing basic instructions.
* **Direct Deployment**: Click **🚀 DEPLOY POD** to run the scheduling flight animation. The pod lands on the calculated winner node, permanently consuming its resources. Controls unlock instantly after flight, allowing consecutive pod deployments.

### 2. Quiz-10 Mode (Scheduling Logic Puzzle)
A turn-based logic test consisting of 10 randomized placement challenges.
* **Locked Pod Parameters**: The Pod configurator locks with a golden border and a `🔒 QUIZ PARAMETERS LOCKED` banner. Sliders, checkboxes, and buttons are set to read-only (`pointer-events: none`) but maintain full high-contrast visibility to remain readable.
* **Randomized Challenges**: Each round randomizes node specifications (capacities, background loads, disks, taints, existing workloads) and pod placement demands.
* **Strict 3-Stage Game Loop**:
  1. **Stage 1: The Guess**: All node cards are styled identically and neutrally (grey LEDs, no borders, no scores, no preview bars). Telemetry logs are suppressed to avoid giving hints. The user must analyze the specifications and click on the Node Card (or the **PENDING ZONE ⚠️** slot if no node is suitable) where they believe the pod will schedule, marking it with a `🎯 GUESS` badge.
  2. **Stage 2: Submit & Evaluation**: The user clicks **🚀 SUBMIT GUESS**. The scheduler runs, revealing green/red highlights, scores, and flight paths. The pod token flies to the node (or Pending zone), and the correctness of the guess is checked. detailed logic is printed in the logs, and chiptune sound effects play. The button changes to `➡️ NEXT QUESTION` (or `🏆 VIEW CERTIFICATE`).
  3. **Stage 3: Next Round Transition**: Clicking the button resets all card states and guess slots, generates a new random question, and enters Stage 1.
* **Rank Certificate**: Completing 10 questions shows a ranking popup based on accuracy:
  - **9-10**: 👑 `K8S CLUSTER ARCHITECT`
  - **7-8**: 🎖️ `SENIOR DEVOPS ENGINEER`
  - **5-6**: 🛠️ `K8S ADMINISTRATOR`
  - **3-4**: 👶 `JUNIOR SYSADMIN`
  - **0-2**: 🐌 `YAML COPY-PASTER`

---

## ⚙️ Kubernetes Scheduling Engine Logic

The simulator implements a realistic dual-phase scheduling workflow modeling the Kubernetes filtering (predicates) and scoring (priorities) pipeline.

### Phase 1: Filtering (Predicates)
Nodes must pass all predicate filters to be considered eligible. If a node fails, it is excluded, and the reasons are printed.
1. **Resource Fit (`PodFitsResources`)**:
   - `Node.AvailableCPU >= Pod.RequestedCPU`
   - `Node.AvailableRAM >= Pod.RequestedRAM`
   - Available capacity includes total capacity minus background load minus already scheduled pods.
2. **Node Zone Affinity (`NodePortsFit` / `NodeAffinity`)**:
   - Checks if the pod requests a specific zone (Zone-A, Zone-B, or Zone-C). If so, the node's zone must match.
3. **Taints and Tolerations (`PodToleratesNodeTaints`)**:
   - If a node is tainted with `gpu:NoSchedule`, the pod must have the toleration checked to be scheduled on that node.
4. **Pod Anti-Affinity (`PodAntiAffinity`)**:
   - If the pod has anti-affinity configured to avoid `[DB]` workloads, the node must not contain any existing or scheduled database workloads.

*If no nodes pass the filters, the Pod fails placement and is sent to the **PENDING ZONE ⚠️**.*

### Phase 2: Scoring (Priorities)
Nodes that pass all filters are scored. The node with the highest score is selected.
1. **Disk Preference (`NodeAffinityPriority`)**:
   - Matches pod's preferred disk type (SSD or HDD).
   - Matching SSD adds **+50 points**.
   - Matching HDD adds **+20 points**.
2. **Pod Affinity (`PodAffinityPriority`)**:
   - If the pod prefers to be near `[Web]` workloads, matching nodes with existing Web workloads receive **+30 points**.
3. **Balanced Allocation (`BalancedResourceAllocation` / `LeastRequested`)**:
   - Encourages balanced usage of CPU and RAM.
   - **Least Requested Score**: `Score += (RemainingCPU% + RemainingRAM%) * 15` (up to 30 pts).
   - **Balanced Score**: `Score += (1 - |RemainingCPU% - RemainingRAM%|) * 20` (up to 20 pts).
   - Adding a balanced resource configuration adds up to **+50 points** total.

### Tiebreaker Rules
If two or more nodes are tied for the highest score:
- The node with the **larger raw RAM Capacity** is selected as the winner.
- If still tied, the first node in the sequence is selected.

---

## 🎨 Visual Design & Audio Features

### Design Aesthetics
- **Cyberpunk Telemetry Dashboard**: Dark mode backdrop (`#060913`) overlaid with an active grid pattern, glowing neon borders, and scanning effects.
- **Glassmorphism Panels**: Semi-transparent frosted-glass containers (`backdrop-filter: blur(10px)`) with subtle gradient borders.
- **Dynamic Capacity Meters**: Displays CPU and RAM allocations in four distinct colored bar segments:
  - **Slate Blue**: Fixed background workloads.
  - **Purple**: Resources consumed by deployed pods.
  - **Pulsing Cyan**: Potential resources for the current pod request (preview).
  - **Flashing Red**: Allocation overflow.

### Audio Synthesizer
The simulator features a procedural chiptune engine utilizing the **Web Audio API** (no external audio assets required). Toggled via the audio icon button:
- **Tick / Click**: Sharp micro-beeps for slider moves and button selections.
- **Swoosh**: Exponential sweep down when a pod token is flying.
- **Success Tone**: Arpeggiated triangle-wave chord (G4 -> C5 -> E5 -> G5) on correct guess / successful schedule.
- **Error Tone**: Low-frequency sawtooth buzz on incorrect guess / pending placement.
- **Reset Tone**: Sine-wave sweep down on cluster reset.

---

## 📄 Real-time YAML Manifests
Generates valid, syntax-highlighted Kubernetes manifests dynamically:
* **Pod Spec**: Reflects requested CPU/RAM resources, node affinity expressions, tolerations, and pod affinities.
* **Selected Node Spec**: Reflects the node's hostname, zone labels, disk labels, taints, capacity limits, allocatable resources, and running workloads.
* **Scheduler Config**: Shows the custom scheduling profile enabling or disabling filters and priorities.
* **Copy to Clipboard**: Quick-copy button copies the selected YAML manifest with visual toast notifications.

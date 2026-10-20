# 🚦 Gate-Calibrated Traffic Vision & Acoustic Analyzer

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![TensorFlow.js](https://img.shields.io/badge/TensorFlow.js-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![Web Audio API](https://img.shields.io/badge/Web_Audio_API-38BDF8?style=flat-square)
![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-222222?style=flat-square&logo=github&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)

A lightweight, zero-backend web application that extracts **classified vehicle counts**, **perspective-independent fleet average speed**, and **acoustic honking frequency** directly from uploaded traffic videos. 

Built as a single-file HTML5 application powered by **TensorFlow.js (`COCO-SSD`)** and the **Web Audio API**, it runs entirely inside the client's browser and deploys out-of-the-box on **GitHub Pages**.

---

## ✨ Key Features

* **Arbitrary Camera Angle Support:** Works with roadside tripods, foot-overbridge (FOB) CCTV angles, elevated building views, and overhead drone footage.
* **Interactive Virtual Tripwires (Gates A & B):** Drag-and-drop or click-to-place 4 calibration points directly on the video canvas to define counting and speed-trap lines across any road orientation.
* **Three-Layer Anti-Overcounting Pipeline:**
  * **Cross-Class Non-Maximum Suppression (NMS):** Merges overlapping bounding boxes ($\text{IoU} > 0.45$) so a single vehicle is never simultaneously double-detected as both a `car` and a `truck`.
  * **2D Vector Line Intersection:** Vehicles are counted strictly when their bottom-center road contact anchor crosses **Gate A** (`A1–A2`).
  * **Spatial-Temporal Gate Lock:** Applies a localized cooldown window along the tripwire to prevent flickering or fragmented bounding boxes from triggering duplicate counts.
* **Two-Gate Time-Mean Speed Trap:** Measures exact travel time between **Gate A** and **Gate B** over a user-defined real-world distance (in meters) to compute class-wise and fleet-wide average speeds ($\text{km/h}$).
* **Automated Acoustic Honk Counter:** Decodes the video's audio stream via an offline biquad bandpass filter centered on vehicle horn frequencies ($1.8\text{--}3.5\text{ kHz}$), counting discrete honk peaks above the rolling ambient road noise floor.
* **100% Client-Side Privacy:** Video files never leave the user's device; all computer vision and acoustic signal processing execute locally in the browser.

---

## 🧠 Methodology & Architecture

### 1. Vehicle Detection & Tire-Anchor Tracking
For each video frame, objects are detected and filtered into four primary transport classes:
* **Two-Wheeler (`2W`):** Motorcycles and bicycles
* **Car (`car`):** Sedans, hatchbacks, SUVs, and vans
* **Bus (`bus`):** Public transit and institutional buses
* **Truck (`truck`):** Light, medium, and heavy commercial vehicles (LCVs/HCVs)

Rather than tracking the center of the 2D bounding box (which suffers from parallax shift on tall vehicles like buses and trucks), the tracker monitors the **bottom-center tire contact anchor**:

$$P_t = \left( x + \frac{w}{2},\; y + 0.92h \right)$$

### 2. Virtual Tripwire Intersection (Counting)
A tracked vehicle transitions from uncounted (blue bounding box) to verified/counted (green bounding box with `✓`) only when its displacement vector $\vec{P_{t-1}P_t}$ intersects the segment $\vec{A_1A_2}$ using counter-clockwise ($\text{CCW}$) orientation predicates:

$$\text{CCW}(P_{t-1}, A_1, A_2) \neq \text{CCW}(P_t, A_1, A_2) \quad \text{and} \quad \text{CCW}(P_{t-1}, P_t, A_1) \neq \text{CCW}(P_{t-1}, P_t, A_2)$$

### 3. Speed Estimation ($\text{Gate A} \leftrightarrow \text{Gate B}$)
Given a user-specified real-world distance $D$ (in meters) between **Gate A** and **Gate B**, and the exact video timestamps $t_A$ and $t_B$ when a vehicle's anchor intersects each gate, the space-mean trap speed is calculated as:

$$v_{\text{km/h}} = \frac{D}{\vert{}t_B - t_A\vert{}} \times 3.6$$

If a vehicle exits the frame or changes lanes before crossing Gate B, the system falls back to the calibrated pixel-to-meter scale inside the shaded trap zone.

### 4. Acoustic Horn Event Detection
1. The audio track is extracted into an `OfflineAudioContext`.
2. A **Biquad Bandpass Filter** ($f_0 = 2600\text{ Hz}$, $Q = 1.8$) isolates dominant fundamental and harmonic vehicle horn frequencies while attenuating low-frequency engine/tire rumble ($< 500\text{ Hz}$).
3. Short-time Root Mean Square ($\text{RMS}$) energy is computed across $50\text{ ms}$ windows.
4. Discrete honk events are logged whenever window energy exceeds $\tau \times E_{\text{median}}$ (where $E_{\text{median}}$ is the clip's median ambient noise floor and $\tau$ is the user sensitivity multiplier), subject to a $300\text{ ms}$ debounce window.

---

## 🚀 Quick Start & Deployment

### Option 1: Run Locally
No build tools, Node.js, or Python environment required.
1. Clone this repository:
   ```bash
   git clone [https://github.com/](https://github.com/)kouniks/Vehicle-Detector.git
   cd Vehicle-Detector

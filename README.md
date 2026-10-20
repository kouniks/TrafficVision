# 🚦 TrafficVision v1 — Mixed-Traffic (2W/3W Enhanced) & Acoustic Analyzer

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![TensorFlow.js](https://img.shields.io/badge/TensorFlow.js-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![Web Audio API](https://img.shields.io/badge/Web_Audio_API-38BDF8?style=flat-square)
![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-222222?style=flat-square&logo=github&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)

A client-side, zero-backend web application designed for **heterogeneous Indian traffic streams**. Upload roadside, foot-overbridge (FOB), or drone video footage to extract **gate-verified vehicle counts across 5 classes (`2W`, `3W/Auto-Rickshaw`, `Car`, `Bus`, `Truck`)**, **space-mean fleet speed**, and **acoustic honking frequency** directly in the browser.

🔗 **Live Demo:** [https://kouniks.github.io/TrafficVision/](https://kouniks.github.io/TrafficVision/)

![TrafficVision Preview](sample_capture.jpg)

---

## ✨ Key Features

* **5-Class Heterogeneous Fleet Detection:** Explicitly resolves **Two-Wheelers (`2W`)**, **Three-Wheeled Auto-Rickshaws (`3W`)**, **Cars/SUVs (`Car`)**, **Buses (`Bus`)**, and **Commercial Trucks/LCVs (`Truck`)**.
* **SAHI-Style High-Res Gate-Zone Zoom Slicing:** Overcomes standard $300 \times 300$ MobileNet resolution crushing by cropping a native-resolution Region of Interest (ROI) around the active speed-trap gates on every frame, boosting small motorcycle and scooter recall by up to $3\times$.
* **Rider-to-2W Fusion Engine:** Automatically fuses high-confidence `'person'` detections (helmeted riders) overlapping low-confidence motorcycles—or moving rapidly inside the active roadway gate zone—into unified `2W` tracks.
* **3W / Auto-Rickshaw Livery & Morphology Classifier:** Separates three-wheelers from standard sedans using depth-normalized footprint width, boxy aspect ratios ($h/w$), canopy luminance contrast, and HSV color segmentation (Indian yellow/black and green/yellow CNG auto-rickshaw liveries).
* **Interactive Virtual Tripwires (Gate A & Gate B):** Drag-and-drop 4 points (`A1, A2, B1, B2`) across any road angle. Vehicles are counted strictly once upon intersecting **Gate A**, while space-mean trap speed is computed between **Gate A** and **Gate B**.
* **Acoustic Horn Frequency Counter:** Uses the Web Audio API (`OfflineAudioContext`) with a biquad bandpass filter centered at $2.6\text{ kHz}$ ($1.8\text{--}3.5\text{ kHz}$ passband) to isolate and count discrete vehicle honking events above the rolling ambient noise floor.

---

## 🧠 Methodology & Technical Pipeline

### 1. Dual-Pass Slicing & Rider-to-2W Recovery
Standard browser object detectors downscale full $1080\text{p}$ frames into small square tensors, causing distant motorcycles to vanish and elevated camera views to detect only the rider's torso. TrafficVision runs a two-stage pass per frame:
1. **Full-Frame Pass (`mobilenet_v2`):** Captures large vehicles (`bus`, `truck`, `car`) and macroscopic trajectories.
2. **Gate-Zone Native Crop (SAHI Pass):** Extracts the bounding polygon spanning **Gate A** and **Gate B** with padding and runs a second high-resolution inference pass.
3. **Rider-Bike Spatial Fusion:** Any `'person'` detection $P$ whose horizontal overlap with a candidate motorcycle $B$ exceeds $35\%$ (or that moves inside the gate zone outside any 4W vehicle cabin with aspect ratio $1.1 \le h/w \le 2.65$) is fused into a `2W` bounding box.

### 2. 3W / Auto-Rickshaw Discrimination
Because MS-COCO lacks a native `auto-rickshaw` label, candidate compact four-wheeler boxes are evaluated across three features:
* **Perspective-Normalized Width:** Compares candidate width $w_i / (y_i / H)$ against the rolling 65th-percentile width of verified passenger cars in the scene ($w_{\text{3W}} < 0.78\,w_{\text{car}}$).
* **Livery HSV Segmentation:** Samples a $24 \times 24$ patch across the bounding box to quantify characteristic auto-rickshaw yellow ($R > 135, G > 110, B < 95$) and CNG green ($G > 1.15R, G > 1.12B$) pixel ratios alongside top-canopy luminance contrast.
* **Temporal Majority Voting:** Each tracked ID accumulates class votes across consecutive frames so single-frame lighting changes never flip a vehicle's classification.

### 3. Virtual Gate Intersection & Space-Mean Speed Trap
Rather than tracking the 2D box centroid (which shifts due to camera parallax on tall buses and trucks), the tracker locks onto the **bottom-center road tire contact anchor**:

$$P_t = \left( x + \frac{w}{2},\; y + 0.90h \right)$$

A vehicle is counted strictly when its motion vector $\vec{P_{t-1}P_t}$ intersects **Gate A** ($\vec{A_1A_2}$) via counter-clockwise ($\text{CCW}$) orientation predicates:

$$\text{CCW}(P_{t-1}, A_1, A_2) \neq \text{CCW}(P_t, A_1, A_2) \quad \text{and} \quad \text{CCW}(P_{t-1}, P_t, A_1) \neq \text{CCW}(P_{t-1}, P_t, A_2)$$

When the vehicle subsequently crosses **Gate B** ($\vec{B_1B_2}$) across a known real-world trap distance $D$ (in meters), its exact space-mean speed is recorded:

$$v_{\text{km/h}} = \frac{D}{\vert{}t_B - t_A\vert{}} \times 3.6$$

### 4. Acoustic Honk Detection ($1.8\text{--}3.5\text{ kHz}$)
1. Extracts the full audio buffer upon video upload and passes it through a Biquad Bandpass Filter ($f_0 = 2600\text{ Hz}, Q = 1.8$).
2. Computes short-time Root Mean Square ($\text{RMS}$) energy over $50\text{ ms}$ windows.
3. Registers a honk event whenever energy exceeds $2.3 \times E_{\text{median}}$ (the clip's ambient road noise floor), using a $300\text{ ms}$ debounce window to avoid double-counting sustained horn blasts.

---

## 🚀 Deployment on GitHub Pages

1. Clone or fork this repository:
   ```bash
   git clone [https://github.com/kouniks/TrafficVision.git](https://github.com/kouniks/TrafficVision.git)
   cd TrafficVision

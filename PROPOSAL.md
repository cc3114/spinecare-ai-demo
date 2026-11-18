# SpineCare AI: Project Proposal

*On-Device Browser-Based Posture Screening and Form-Gated Corrective Exercise System*

| Project Resource | Reference Link / Identifier |
| --- | --- |
| **Source Repository** | [github.com/cc3114/spinecare-ai-demo](https://github.com/cc3114/spinecare-ai-demo) |
| **Live Web Demonstration** | [cc3114.github.io/spinecare-ai-demo](https://cc3114.github.io/spinecare-ai-demo/) |

## 1. Executive Summary

SpineCare AI is a browser-based application that turns a standard smartphone into a posture-screening and corrective-exercise tool without requiring external hardware, cloud uploads, or clinical equipment. Using an on-device 33-landmark pose estimation model, the system evaluates shoulder tilt, forward-head posture, and forward-bend asymmetry against clinically informed thresholds, and subsequently guides users through five corrective exercises with form-gated repetition counting.

| Dimension | Specification |
| --- | --- |
| **Target Users** | Parents monitoring school-age children, desk-based workers, and patients performing home rehabilitation between clinical visits |
| **Core Technology** | MediaPipe Pose Landmarker (float16, 33 landmarks) running via WASM/WebGL in the browser |
| **Problem Addressed** | Postural deviations are often visually apparent but remain unquantified; clinical screening is costly, infrequent, and reliant on specialized facilities |
| **Implementation Status** | Functional proof of concept supporting 3 screening poses, 5 guided exercises, annotated image capture, local history logging, and a bilingual interface |

## 2. Problem Statement

### 2.1 The Gap Between Postural Awareness and Objective Measurement

Spinal and postural deviations generally progress gradually without immediate symptoms. Adolescents with idiopathic scoliosis often show few noticeable signs until spinal curvature becomes pronounced; office workers develop forward-head and rounded-shoulder alignment over years of prolonged screen use; and patients in post-surgical or conservative rehabilitation lose objective feedback once they leave the clinic.

In each of these groups, posture is typically evaluated through subjective self-perception rather than measurement. A parent cannot reliably distinguish between a 2° and an 8° shoulder tilt in a growing child, an office worker is rarely aware that their head posture has shifted 15° forward, and a rehabilitation patient exercising at home cannot verify whether their movement quality has deteriorated since their previous session.

### 2.2 Limitations of Existing Approaches

| Existing Approach | Key Limitations |
| --- | --- |
| **Posture braces** | Provide passive mechanical support without measuring alignment. Prolonged use can encourage device dependency while failing to strengthen underactive postural muscles, leaving users with no objective record of progress. |
| **Wearable sensors (IMU patches, smart garments)** | High hardware cost (often several hundred dollars per user) alongside charging and compliance burdens. Single-point inertial measurement units (IMUs) capture trunk inclination but cannot detect shoulder asymmetry, lateral head offset, or rib-hump prominence. |
| **Timer-based reminder apps** | Lack computer-vision or sensor input. Alerts trigger on fixed schedules regardless of the user's actual posture, causing notifications to feel irrelevant and leading to rapid user drop-off. |
| **Clinical screening and radiography** | Full-spine X-rays serve as the diagnostic gold standard, yet hospital visits, high costs, and cumulative radiation exposure restrict use to a few times per year, making weekly monitoring impractical. |
| **Outpatient scoliosis rehabilitation clinics** | Therapist-led sessions offer high-quality supervision, but they account for only a small fraction of a patient's week. Travel demands and session fees limit access, and once patients return home, they lack real-time form verification, objective logs, or early warnings when exercise technique declines. |

Across these methods, the primary gap is the lack of an accessible, repeatable measurement tool that users can apply at home on a weekly basis between clinical appointments.

### 2.3 Technological Convergence

Three recent technical and regulatory shifts make browser-based posture screening feasible:

**Client-side computer vision performance:** Lightweight landmark models—such as MediaPipe Pose Landmarker (float16, approximately 5 MB)—now execute at interactive frame rates inside mobile web browsers via WebAssembly (WASM) and WebGL, eliminating the need for native app installations, server-side GPUs, or per-inference cloud costs.

**Ubiquitous camera hardware:** Because high-resolution cameras are already built into standard smartphones, the marginal hardware cost for end users is zero.

**Stricter health data privacy expectations:** Users and regulatory frameworks increasingly scrutinize the transmission of health-related video to external servers. Processing video frames locally removes data-transfer risks while simplifying compliance.

Together, these factors allow basic postural screening—previously confined to clinical settings—to be distributed directly via a standard web URL.

## 3. Solution and Proof-of-Concept Demonstration

### 3.1 Four-Stage Closed-Loop Workflow

The application organizes each session into a four-stage feedback loop:

**Workflow Loop:** Measure (3 screening poses) → Grade (traffic-light risk tiers) → Correct (5 guided exercises) → Track (annotated photos + local history)

Each screening session concludes with a single actionable outcome: either a green-light status paired with a maintenance drill, or an amber/red alert linked directly to the corrective exercise targeting the detected deviation.

### 3.2 Implemented Proof-of-Concept Features

All features described below are implemented and operational in the current project repository.

#### 3.2.1 Three Screening Poses and Pose-Specific Metrics

| Pose | Camera Orientation | Computed Landmark Metrics | Grading Thresholds |
| --- | --- | --- | --- |
| **Back view** | Full body, back facing camera | Shoulder tilt (°), pelvic tilt (°), lateral head offset (% of shoulder width) | ≥ 2.5° (amber); ≥ 6° (red) |
| **Side view** | Profile stance, side facing camera | Forward-head angle (°), trunk lean (°), plumb-line offset (% of torso length) | Forward head: ≥ 12° / 20°; trunk lean: ≥ 10° / 15° |
| **Forward bend** | Back facing camera, torso flexed ~90° | Bend angle (°, 3D including depth axis), back asymmetry (%), posture form check | Asymmetry: ≥ 5% (amber); ≥ 10% (red) |

Joint and segment angles are calculated as vector-to-vertical angles using the inverse cosine (acos) function, which mathematically constrains outputs to the [0°, 180°] interval and avoids the quadrant wrap-around issues associated with naive atan2 implementations.

#### 3.2.2 Zero-Point Calibration

Minor camera tilt and individual anatomical baselines can bias raw angular measurements. Users can record a baseline calibration for any pose with a single tap; subsequent sessions display deviations relative to that baseline and evaluate changes against the same thresholds, improving the reliability of week-over-week comparisons.

#### 3.2.3 Five Corrective Exercises with Form-Gated Repetition Counting

| Exercise | Camera Stance | Automated Form Gate |
| --- | --- | --- |
| **Arm raise (双手托天)** | Back view | Prohibits shoulder shrugging during elevation; maintains shoulder tilt and trunk lean within limits |
| **Shoulder release (沉肩静站)** | Back view | Monitors a static hold; upward shoulder elevation triggers an immediate correction prompt |
| **Back extension (小燕飞)** | Side view, prone | Requires trunk lift to remain within a 20°–45° target range; hyperextension invalidates the repetition |
| **Cat-cow (猫式伸展)** | Side view, quadruped | Tracks full phase transitions between cervical flexion (head-down) and extension (head-up) |
| **Wall angel (靠墙天使)** | Side view, back against wall | Tracks overhead arm abduction and return; shoulder-shrugging compensation invalidates the repetition |

A repetition is counted only when posture remains within valid thresholds throughout the entire movement cycle. If a form violation persists for 3 consecutive frames, the current repetition is invalidated and an on-screen corrective prompt appears. Keypoint visibility scores are also gated so that occluded limbs cannot trigger false completions, addressing a common limitation in unsupervised home exercise.

#### 3.2.4 Pre-Exercise Stance Guides

Before starting an exercise, the interface displays a full-screen setup guide specifying the required camera orientation in plain language (e.g., "Side view · back against a wall") alongside step-by-step execution instructions. This prevents setup errors—such as filming from an oblique angle—that would otherwise distort downstream angle calculations.

#### 3.2.5 Hands-Free Image Capture and Composite Evidence

Because users cannot hold a smartphone while maintaining a full-body screening posture, the system provides two hands-free capture mechanisms: a silent 3-second visual countdown and a gesture trigger. The gesture trigger uses a lazily loaded 21-landmark hand model to recognize an "OK" sign (thumb-to-index pinch with three extended fingers) held for approximately 0.5 seconds, governed by an 8-second cooldown to prevent duplicate captures.

Each captured frame is composited locally on an HTML5 canvas—combining the camera image, skeletal overlay, calculated metrics, pose label, and timestamp—and saved both to local history and as a downloadable JPEG.

#### 3.2.6 Local Measurement History and Summary Statistics

Measurement records are persisted in browser `localStorage` (up to 20 entries, including image thumbnails). The history view displays the total record count, most recent grade, lowest recorded grade, and a chronological log of per-session metrics linked to full composite images. If browser storage quotas are exceeded, the system falls back gracefully to storing numerical metrics without thumbnails.

#### 3.2.7 Operational Robustness

| Feature | Technical Implementation |
| --- | --- |
| **Full-body framing check** | Verifies visibility and bounding-box margins for the nose and both ankles, displaying live "step back" or "ready" guidance |
| **Signal stabilization** | Applies exponential moving average (EMA) smoothing (α = 0.35) to landmark coordinates and requires 4 consecutive matching frames before updating grade states to prevent visual flicker |
| **Camera selection** | Supports toggling between front and rear cameras with persistent preference storage |
| **Bilingual interface** | Provides full Chinese and English localization (~96 translation keys per language) with runtime switching |
| **Graceful fallback modes** | Switches to simulation mode if camera access is denied or model downloads fail, and supports custom model mirror URLs for restricted network environments |
| **Camera-less demo mode** | Passing `?demo=1` in the URL routes an animated canvas figure through the live inference, overlay, and rep-gating pipeline |

## 4. Technical Architecture and Design Highlights

### 4.1 System Architecture and Data Flow

| Architectural Layer | Components & Responsibilities |
| --- | --- |
| **1. Media Acquisition (On-Device)** | HTTPS getUserMedia camera stream routed into an HTML `<video>` element as a local frame source. |
| **2. Inference Engine (WASM + WebGL)** | MediaPipe FilesetResolver (tasks-vision 0.10.14) executing PoseLandmarker (pose_landmarker_lite, float16, GPU, 33 landmarks/frame) and lazy-loaded HandLandmarker (hand_landmarker, float16, 21 landmarks for OK-gesture capture). |
| **3. Deterministic Analysis Layer** | EMA coordinate smoothing (α = 0.35); vector-to-vertical acos angle engine (2D and 3D depth variants); per-pose zero-point calibration; threshold grading with 4-frame debounce; and multi-condition form gate (visibility, tilt, shrug, ROM with 3-frame violation window). |
| **4. Presentation Layer** | HTML5 skeleton canvas overlay, live metric panel with grade indicator, plain-language stance guides / correction prompts, and capture compositor (frame + skeleton + metrics + timestamp). |
| **5. Local Storage (`localStorage`)** | Stores settings (`jp_lang`, `jp_facing`, `jp_qi`), calibration baselines (`jp_calib`), up to 20 session records with thumbnails (`jp_reports`), and optional model mirror overrides (`jp_model_url`). |
| **External Network Boundaries** | One-time static fetch of MediaPipe WASM runtime (jsDelivr CDN) and .task model weights (model host) on first load only; zero user data egress. |

**Data-Flow Summary:** Video frames from getUserMedia pass into an HTML `<video>` element and are processed directly inside the browser thread by the MediaPipe landmark models. Joint angles, calibration offsets, and form checks are computed in client-side JavaScript, and all outputs remain in local storage. The application contains no backend API, fetch or XHR telemetry calls, analytics SDKs, or user authentication modules; external network traffic is restricted exclusively to the initial download of the MediaPipe WASM runtime and model weight files from static CDNs.

### 4.2 Key Engineering Decisions

**Privacy by architectural constraint:** Rather than relying on server-side data-retention policies, the system removes external data transmission paths entirely—a property verifiable directly in the 1,567-line single-file source code. Eliminating backend servers also removes recurring GPU inference and bandwidth expenses.

**Deterministic, auditable biomechanical calculations:** Every displayed angle maps to a named pair of anatomical landmarks, an explicit acos vector-to-vertical formula, and a documented threshold. In the forward-bend assessment, calculations incorporate the depth (z) coordinate because spinal flexion occurs primarily along the anterior-posterior plane rather than laterally. This ensures all outputs are reproducible and transparent.

**Multi-condition form gating:** To distinguish therapeutic movement from unguided motion, repetition credit requires simultaneous satisfaction of keypoint visibility, tilt bounds, shoulder-shrug limits, and range-of-motion windows (such as the 20°–45° zone for back extensions). Requiring violations to persist across 3 consecutive frames filters out single-frame landmark jitter while capturing sustained postural compensation. Thresholds are also conditioned on camera view—for example, shoulder tilt is omitted in side-view exercises where left and right landmarks overlap.

**Low-friction user workflow:** Long-term adherence is often the primary bottleneck in home exercise programs. Delivering the tool as a zero-install web link with no login requirement, pre-exercise stance instructions, hands-free OK-gesture capture, and a 20-session visual log reduces setup friction. Additionally, the `?demo=1` query parameter allows full pipeline demonstrations without a live camera or human subject.

**Bounded angular arithmetic and unit testing:** Angular pipelines are prone to edge-case overflow and unit mismatches. Using a vector-angle formulation strictly bounded to [0°, 180°] and validating outputs against synthetic geometric test cases (0.0° upright, 20° lean, 45° and 90° flexion) helped identify and resolve a double degree-conversion defect and an inverted shrug-distance check during development.

## 5. Competitive Analysis

### 5.1 Feature and Capability Comparison

| Criterion | SpineCare AI | Posture Brace | Wearable Sensor | Generic App | Hospital / X-Ray | Outpatient Rehab Clinic |
| --- | --- | --- | --- | --- | --- | --- |
| **User cost** | ¥0 (existing phone) | ¥100–300 (replaced with growth) | ¥500–2,000 + charging | ¥0 | ¥300–800/visit + radiation | ¥200–600/session (2–3×/wk) |
| **Measurement** | 33 landmarks → joint angles | None (passive support) | Single-point IMU orientation | None (timer alerts) | Radiographic Cobb angle | Therapist visual & manual exam |
| **Detectable features** | Shoulder/pelvic tilt, forward head, trunk lean, bend asymmetry | None | Trunk inclination only | None | Full skeletal spinal geometry | Whole-body movement & palpation |
| **Real-time feedback** | Per-frame metrics + correction prompts | None | Vibration cues only | Scheduled reminders only | Post-visit report | Live verbal & tactile cues |
| **Form verification** | Automated form-gated rep counting | N/A | None | None | In-clinic only | Direct therapist supervision |
| **Intervention specificity** | Pose thresholds mapped to 5 exercises | Generic | Generic | Generic | In-hospital consultation | Individualized protocol |
| **Between-visit continuity** | Home session logs & trend tracking | None | Cloud app-dependent | Minimal | Follow-up visits only | Ends upon leaving clinic |
| **Longitudinal records** | Up to 20 on-device composite photos | None | Partial (cloud-stored) | Minimal | Periodic hospital imaging | Session notes; home unverified |
| **Accessibility** | Any location with smartphone | Any location | Any location (wear burden) | Any location | Hospital appointment | Clinic travel & fixed slots |
| **Data privacy** | 100% on-device processing & storage | N/A | Often vendor cloud-synced | Often vendor cloud-synced | Hospital EMR | Provider clinical records |
| **Onboarding** | Web URL (~60 seconds) | Daily physical wear | Pair, charge & calibrate | Account sign-up | Scheduling & wait times | Referral & clinical intake |
| **Regulatory category** | General wellness & screening aid | Consumer support product | Varies by device | General wellness | Regulated medical device | Licensed clinical service |
| **Practical frequency** | Unlimited | N/A | Continuous (trunk only) | N/A | 1–2× per year | 2–3×/week while enrolled |

### 5.2 Relationship to Outpatient Rehabilitation Clinics

Outpatient scoliosis rehabilitation centers represent the highest standard of hands-on care in this comparison, outperforming camera-based tools in diagnostic precision, manual correction, and individualized program design. However, clinic-based care is inherently constrained by time: a patient attending three one-hour sessions per week spends roughly 3 hours under clinical supervision and 165 hours at home without objective feedback or form monitoring.

Rather than competing with physical therapists, SpineCare AI is designed to bridge this 165-hour gap by giving patients a structured, 60-second check-in tool that tracks the same postural metrics emphasized in clinic. This complementary relationship also underpins the clinic-adjacent follow-up model outlined in Section 6.

### 5.3 Positioning and Technical Limitations

**Core positioning:** SpineCare AI addresses the monitoring gap during the intervals between annual check-ups and weekly therapy sessions, combining zero marginal hardware cost, quantitative angular tracking, and local data privacy.

**Acknowledged technical limitation:** Because standard smartphone cameras capture monocular RGB video without dedicated depth sensors, forward-bend asymmetry is estimated from 2D landmark displacement and the pose model's predicted z-coordinate rather than true 3D surface topography. Consequently, the application functions as a screening and adherence aid rather than a substitute for radiographic Cobb-angle measurement, a limitation stated explicitly on screen.

## 6. Feasibility and Deployment Strategy

### 6.1 Technical and Operational Feasibility

**Minimal hosting overhead:** Because inference runs entirely on the client device, the application can be served from static web hosting with near-zero marginal compute cost per session.

**Cross-platform browser distribution:** Deploying via a standard URL bypasses app-store approval cycles, supports both iOS and Android browsers immediately, and allows distribution via messaging apps, school notices, or workplace portals.

**Validated core pipeline:** All primary screening, calibration, form-gating, and local storage modules are already implemented in the prototype, shifting remaining effort from algorithmic exploration to product refinement.

### 6.2 Potential Deployment Tracks

**Track A — Consumer Freemium:** All 3 screening poses, risk grading, 3 corrective exercise modules, and 20-record local history (¥0); Plus tier adds full exercise library, PDF/CSV report export, multiple family profiles, extended longitudinal charts, and optional encrypted cloud backup (¥19–29/month or ¥128–198/year).

| Tier | Included Features | Proposed Pricing |
| --- | --- | --- |
| **Free** | All 3 screening poses, risk grading, 3 corrective exercise modules, and 20-record local history | ¥0 |
| **Plus** | Full exercise library, PDF/CSV report export, multiple family profiles, extended longitudinal charts, and optional encrypted cloud backup | ¥19–29/month or ¥128–198/year |

**Track B — Workplace Musculoskeletal Wellness (B2B):** Employers and insurers seeking to reduce musculoskeletal complaints can license an annual per-seat check-in module with anonymized, aggregate participation metrics. On-device video processing simplifies corporate IT and privacy reviews because no employee images enter employer servers.

**Track C — Clinic-Adjacent Patient Follow-Up:** Orthopedic and rehabilitation clinics can adopt a white-labeled version that generates exportable home-exercise adherence and posture logs, extending clinician visibility between appointments.

**Track D — School Screening Pilots (Long-Term):** In regions conducting routine school scoliosis checks, a teacher-supervised 60-second browser screening could standardize initial triage and reduce false-positive specialist referrals, subject to institutional approval and ethical review.

### 6.3 Compliance and Scope Boundaries

SpineCare AI is designed strictly as a general wellness and posture-management aid. It is not a medical device, does not output clinical diagnoses or Cobb-angle measurements, and does not replace physical examination or medical imaging.

User-facing feedback is restricted to non-diagnostic screening advisories ("within normal range," "worth observing," and "consider seeing a specialist"), accompanied by a persistent disclaimer recommending consultation with an orthopedic specialist and a full-spine radiograph for definitive clinical evaluation. All measurements and images remain inside the user's browser `localStorage` and can be deleted at any time; any future cloud synchronization will require explicit opt-in consent and end-to-end encryption.

## 7. Development Roadmap

### Phase 1: Proof-of-Concept Prototype (Completed)

- In-browser 33-landmark pose estimation (float16, GPU delegate via WASM).
- Three screening poses (back, side, and forward bend) with pose-specific thresholds.
- Bounded 2D and 3D vector-angle engine verified against known geometric baselines.
- Per-pose zero-point calibration to account for camera tilt and baseline asymmetry.
- Live full-body framing checks and stance prompts.
- Five corrective exercises with plain-language setup guides, form-gated repetition counting, and correction alerts.
- Hands-free image capture via a 3-second countdown or lazily loaded 21-landmark OK-gesture recognition.
- Local composite JPEG generation and a 20-record `localStorage` history log.
- Bilingual Chinese/English interface with camera toggling, simulation fallback, and `?demo=1` mode.

### Phase 2: Productization and Cross-Device Refinement (Months 1–3)

- Progressive Web App (PWA) packaging with a service worker for offline runtime and model caching.
- Multi-profile support to separate calibration baselines and histories for multiple family members.
- Longitudinal trend charts per metric and pose, paired with PDF/CSV export for clinical visits.
- Multi-week structured exercise progressions rather than standalone drills.
- Threshold validation against a labeled posture dataset to quantify sensitivity and specificity relative to clinician ratings.
- Adaptive frame-rate scaling for lower-end smartphones, reduced-motion accessibility options, and optional audio cues.

### Phase 3: Ecosystem Integration and Institutional Pilots (Months 6–12)

- Optional, consent-gated encrypted cloud backup.
- Read-only clinician review dashboard for patient-shared adherence and trend reports.
- Pilot deployments with 1–2 rehabilitation clinics to evaluate home-exercise adherence and measurement consistency.
- Pilot integration with an employer wellness program using anonymized aggregate participation metrics.
- Expanded exercise library featuring beginner regressions and advanced progressions.
- Optional auto-generated longitudinal trend summaries, clearly labeled as non-diagnostic.
- Formal regulatory review in target markets to ensure ongoing alignment with wellness-tool classifications.

## 8. Appendix: Verifiable Implementation Index

| Proposal Claim | Codebase Verification Location |
| --- | --- |
| **33-landmark float16 pose model with GPU delegate** | `index.html` — PoseLandmarker.createFromOptions |
| **Lazy-loaded 21-landmark hand model for gesture capture** | `index.html` — gesture toggle handler |
| **Pose-specific metrics and grading thresholds** | `index.html` — POSES configuration object |
| **Bounded vector-angle calculations (acos, 0–180°)** | `index.html` — angleUp and angleUp3 functions |
| **EMA smoothing (α = 0.35) and 4-frame grade debounce** | `index.html` — draw() render loop |
| **Multi-condition form gating with 3-frame violation window** | `index.html` — training block inside draw() |
| **20-record local measurement history** | `index.html` — addRecord() function |
| **Zero external transmission of user video or metrics** | Repository-wide (no fetch, XHR, or analytics calls; outbound traffic limited to CDN runtime/model URLs) |

**Medical Disclaimer:** SpineCare AI is a personal posture-management and screening aid; it is not a medical device and does not provide medical diagnoses. For definitive clinical evaluation, consult an orthopedic specialist and obtain a full-spine radiograph.

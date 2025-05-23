# AAVID

**Anonymized Audio-Visual Identification & Diarization**

AAVID is a three-stage pipeline that

1. **Captures** live webcam video/audio and **blurs every un-registered face in real time**  
2. **Locates active speakers on screen** via lip-sync analysis and face tracking  
3. **Builds a speaker-timeline** with neural diarization and voice biometrics – even when faces are occluded  

The result is a privacy-safe MP4 plus JSON/TXT metadata ready for downstream emotion, gaze, or speech analytics.

---

## Demo  
<!-- Replace the two links below with your own once uploaded -->
<div align="center">
  <video src="https://github.com/user-attachments/assets/13979c64-8b65-4533-a208-b09d52c92b99" controls width="60%">
    Your browser does not support the video tag.
  </video>
  <p><i>Real-time face masking & active-speaker overlay</i></p>
</div>

---

## System Architecture  
<!-- Insert graphic -->
<div align="center">
  <img src="https://github.com/user-attachments/assets/d30c6c0b-3fbd-4129-8ef4-2a9d5e786a52" width="100%" alt="AAVID system diagram">
  <p><i>End-to-end processing flow</i></p>
</div>


---

## Table of Contents
1. [Introduction](#introduction)  
2. [Key Features](#key-features)  
3. [Pipeline Overview](#pipeline-overview)  
4. [Installation](#installation)  
5. [Usage](#usage)  
6. [Directory Layout](#directory-layout)  
7. [License](#license)  
8. [Acknowledgements](#acknowledgements)

---

## Introduction
AAVID tackles the classic **“Who spoke when?”** problem under strict privacy constraints.  
It first anonymizes all un-consented faces on the fly, then fuses **audio-visual lip-sync** cues with **audio-only diarization + speaker-ID** to output accurate speaking segments—even when subjects turn away from the camera.

Typical use-cases:

* **Tele-therapy / classroom recordings** – store compliant footage while preserving analytics value  
* **Meeting summarization** – feed speaker-labeled timestamps into ASR for speaker-attributed transcripts  
* **Content moderation** – mask bystanders in live streams; highlight the person currently speaking

---

## Key Features
| Capability | Details |
|------------|---------|
| **Real-time anonymization** | RetinaFace + ArcFace blur every foreign face before disk write |
| **Multi-face tracking** | Lightweight SORT ensures stable IDs between frames |
| **Lip-sync active-speaker** | TalkNet scores per frame; robust when faces visible |
| **Neural diarization** | NVIDIA Sortformer 4-spk handles overlap (≤2 speakers best) |
| **Voice biometrics** | Titanet-S matches diarized streams to reference clips |
| **Threaded producer/consumer** | Separate capture, processing, and encoding threads to keep 30 fps |
| **Extensible** | Swap detectors, trackers, or diarizers without changing downstream formats |

---

## Pipeline Overview

| Stage | Script | Main libs/models | Output |
|-------|--------|------------------|--------|
|**1** Real-time mask | `camera_face_recognition_masking.py` | RetinaFace, ArcFace | `masked.mp4` |
|**2** AV active-speaker | `offline_face_recognition_blurring_lip_sync.py` | InsightFace, SORT, TalkNet | `speaking_segments.json`, `label_scores.txt`, `video_out.avi` |
|**3** Audio biometrics | `audio_biometrics_nemo.py` | NeMo Sortformer, Titanet-S | `timeline_labeled.txt`, `annotated_video.mp4` |

---

## Installation

```bash
git clone https://github.com/<user>/AAVID.git
cd AAVID
chmod +x setup.sh
setup.sh
```

## Usage
```bash
# 1️ Capture & anonymize a 30-s session
python camera_face_recognition_masking.py \
       --cam 0 --mic 3 \
       --outfile output/masked.mp4 \
       --duration 30

# 2️ Offline AV active-speaker analysis
python offline_face_recognition_blurring_lip_sync.py \
       --videoFolder output \
       --videoName masked \
       --face_masking                         # remove flag to skip second-pass blur
                                             # adjust --simThreshold if needed

# 3️ Audio-only diarization + speaker-ID
python audio_biometrics_nemo.py \
       --dir output/masked/pyavi \
       --video_name video_out.avi \
       --sim_threshold 0.5
```

### Directory Structure
```
AAVID/
├── camera_face_recognition_masking.py
├── offline_face_recognition_blurring_lip_sync.py
├── audio_biometrics_nemo.py
├── model/                 # TalkNet, S3FD, SORT, etc.
├── weights/               # Pre-trained weights
├── registeredFaces/       # Face gallery
├── voice_samples/         # Enrolment audio clips
├── output/                # Results
├── setup.sh               # one-shot env installer
└── README.md
```

## License
This repository is private and intended solely for Center for Unified Biometrics and Sensors Lab (University at Buffalo) use.

## Acknowledgment
This project adopts trained ASD model from [TalkNet-ASD](https://github.com/TaoRuijie/TalkNet-ASD), face detection from [S3FD](https://github.com/sfzhang15/SFD), face recognition from [Insightface](https://github.com/deepinsight/insightface), tracking code from [SORT](https://github.com/abewley/sort), and Audio ASR model from [NVIDIA NeMo](https://github.com/NVIDIA/NeMo)

## Contact
For more information, contact [Divyesh Pratap Singh](https://www.linkedin.com/in/divyesh-pratap-singh/)




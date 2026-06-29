# AI Health Sensing


Two self-contained demos showing how commodity hardware: a smartphone camera and a thermal infrared camera can extract physiological signals without any contact sensors.

---

## Notebooks

### Heart Rate from Phone Camera
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/parvthacker/ai-health-sensing/blob/main/phone_ppg/HeartRate.ipynb)

Contact photoplethysmography (PPG) using a smartphone's rear camera and flash. Press your finger over the lens, every heartbeat modulates the brightness of the image. The notebook extracts that oscillation, band-pass filters it, and reads off your heart rate via FFT and peak detection.

### Nasal Respiration from Thermal Video
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/parvthacker/ai-health-sensing/blob/main/thermal_respiration/NasalRespiration.ipynb)

Contactless breathing signal extraction from thermal infrared video using a YOLO-based nostril localiser. The mean thermal intensity of the detected nostril ROI oscillates at the breathing frequency: no sensors, no contact.

---

## How It Works

### Heart Rate (PPG)

```
phone video ──▶ per-frame mean(red channel) ──▶ 1-D signal
   signal ──▶ detrend ──▶ band-pass 0.7–4 Hz ──▶ FFT / peak-detect ──▶ heart rate (bpm)
```

Red light penetrates tissue best under a white LED flash. The camera averages millions of photons per frame, so even the tiny ~10% brightness swing from blood pulsing through capillaries is recoverable with simple signal processing.

### Nasal Respiration (Thermal)

```
thermal AVI ──▶ YOLO detection ──▶ nostril ROI crop ──▶ mean intensity ──▶ respiration signal
```

Exhaled air is warmer than inhaled air. The thermal camera picks up this temperature oscillation at the nostril, producing a waveform at the breathing frequency (~0.1–0.5 Hz, 6–30 breaths/min).

---

## Repo Structure

```
ai-health-sensing/
├── README.md
├── data/
│   └── P01_1/
│       └── thermal_rec.avi          # thermal infrared recording
├── phone_ppg/
│   ├── HeartRate.ipynb
│   └── finger_video.mp4             # real finger-on-lens recording
└── thermal_respiration/
    ├── NasalRespiration.ipynb
    └── models/
        ├── best.pt                  # primary YOLO detector
        ├── NoseLoc_200.pt
        └── 7Class_AllMS.pt
```

---

## Running in Colab

Click either badge above. The first cell in each notebook clones this repo and sets up the environment automatically — no uploads, no configuration needed.

> Select **Runtime → Change runtime type → T4 GPU** before running.

---

## Running Locally

```bash
git clone https://github.com/parvthacker/ai-health-sensing.git
cd ai-health-sensing

# Heart rate
pip install opencv-python scipy numpy matplotlib
jupyter notebook phone_ppg/HeartRate.ipynb

# Nasal respiration
pip install ultralytics opencv-python scipy numpy matplotlib tqdm
jupyter notebook thermal_respiration/NasalRespiration.ipynb
```

---

## Outputs

**HeartRate.ipynb** prints heart rate in bpm and plots:
- Raw per-frame mean intensity
- Band-passed signal with detected beats marked
- Power spectrum with peak frequency

**NasalRespiration.ipynb** writes per video:
```
P01_1/
├── nasal-respf_sig/
│   ├── thermal_rec.npy              # float32 signal array
│   └── thermal_rec_signal.csv       # time_sec, mean_intensity
├── nostril-visible_sig/
│   └── thermal_rec.npy              # binary visibility per frame
└── annotated_videos_sig/
    └── thermal_rec_annotated.avi    # video with waveform + visibility overlay
```

---

> ⚠️ **Educational demos only.** Not validated, not medical devices. Do not use for any diagnostic or clinical purpose.

---


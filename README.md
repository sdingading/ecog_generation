# Virtual ECoG Signal Synthesis for Continuous Motor Decoding

The repository provides the full computational pipeline for:

- Training the generative diffusion model (DDPM-based)
- Sampling virtual ECoG signals
- Performing continuous motor decoding and evaluating decoding performance

---

## Data Structure Description

Although the raw human ECoG recordings cannot be shared due to IRB and privacy restrictions, the structure of the dataset is described below to ensure transparency.

### 1. Channel Information

Channel metadata are stored as a Python dictionary indexed by channel number.

Each entry contains:

- `ch_name`: Electrode label (string)
- `loc`: 3D MNI-space coordinate of the electrode (array of shape (3,))
- `BA`: Brodmann area label (integer)
- `strip`: Electrode strip identifier (integer)
- `AAL3`: Anatomical region index based on the AAL3 atlas (integer)

Example structure:
```
{
0: {
  'ch_name': '1',
  'loc': [x, y, z],
  'BA': 10,
  'strip': 1,
  'AAL3': 22
},
1: {
  'ch_name': '2',
  'loc': [x, y, z],
  'BA': 10,
  'strip': 1,
  'AAL3': 4
},
...
}
```
This structure defines the spatial and anatomical configuration of the implanted electrode array.

---

### 2. Task Metadata

Task-related metadata are stored in a NumPy `.npz` file containing the following keys:

#### `events`
- Shape: (N_events, 3)
- Type: int32
- Description: Event matrix obtained from experiment.
  Columns represent:
  1. Sample index
  2. 0
  3. Event ID to identify type of events

#### `ME`
- Shape: (N_trials, 1)
- Type: int32
- Description: Timepoints corresponding to beginning of Motor Execution (ME) trials.

#### `MI`
- Shape: (N_trials, 1)
- Type: int32
- Description: Timepoints corresponding to beginning of Motor Imagery (MI) trials.

#### `sequence`
- Shape: (N_trials, 1)
- Type: uint8
- Description: Length (number of timepoints) of each trial.

#### `pos`
- Shape: (3, T)
- Type: float64
- Description: 3D hand position trajectory over time.
  Rows correspond to x, y, z coordinates.
  T denotes total number of time samples.

#### `vel`
- Shape: (3, T)
- Type: float64
- Description: 3D hand velocity trajectory over time.
  Computed as the temporal derivative of position.

---

### 3. ECoG Signal Data (.fif format)

Neural recordings are stored in MNE `.fif` format.

Two main formats are used:

#### (a) Continuous Raw Data
- Multichannel raw ECoG signals

#### (b) Epoched Data
- Shape: (N_epochs, N_channels, N_timepoints)
- Time-locked to task events
- Preprocessed prior to epoching (filtering, referencing, artifact rejection)

The `.fif` files follow standard MNE-Python conventions and can be loaded using:

```python
import mne
epochs = mne.read_epochs("file_epo.fif", preload=True)
```
---
### 4. Overall Data Organization
```
data/
 ├── raw_data/
 │    ├── *_channel_info.pkl
 │    ├── *_session_meta.npz
 │    └── *_raw.fif
 │
 └── epoched_generate/
      ├── *_session_meta.npz
      └── *_epo.fif
```
---
## System Requirements

The software has been tested under the following configuration:

- OS: Linux (Ubuntu)
- Python: 3.9
- PyTorch: 2.2.1
- CUDA: 12.x
- MNE: 1.6.1
- RAM: ≥16 GB recommended for training
- GPU: NVIDIA GPU recommended for model training (CPU execution possible for limited steps)

All required Python dependencies are listed in `requirements.txt`.

## Installation Guide

We recommend using a virtual environment.

```bash
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
---

## How to Run the Code
 Using Jupyter Notebooks

Run notebooks in the notebooks/ directory in sequential order:

- Preprocessing and data loading

- DDPM model training

- Virtual signal generation

- Continuous motor decoding

---
## License

This code is released under the MIT License.

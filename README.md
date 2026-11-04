# Audio Noise Reduction

This project explores digital signal processing methods for reducing noise in speech recordings. The main deliverable is the Jupyter notebook [`audio_noise_reduction_compatible_mixture.ipynb`](./audio_noise_reduction_compatible_mixture.ipynb), which builds controlled noisy mixtures, compares denoising methods, and tests a reusable denoiser on recordings without a clean reference.

## Project contents

- `audio/clean_speech.wav` — clean speech used as the known reference in controlled experiments.
- `audio/noise_recording.wav` — separate noise recording used to create mixtures and estimate a noise spectrum.
- `noisy_audio/` — ten WAV recordings used for the batch test.
- `hci_audio_record.wav` — a separate real-recording demonstration, without a corresponding clean reference.
- `audio_noise_reduction_compatible_mixture.ipynb` — experiments, plots, listening comparisons, and reusable function.

## Setup and run

The notebook uses Python, NumPy, SciPy, Matplotlib, and IPython. Install the packages and Jupyter if needed:

```bash
python -m pip install numpy scipy matplotlib ipython jupyter
```

Open a terminal in the `Term Project` directory so the notebook's relative paths (such as `./audio/clean_speech.wav`) resolve correctly, then start Jupyter:

```bash
jupyter notebook audio_noise_reduction_compatible_mixture.ipynb
```

Run the notebook from top to bottom. The analysis cells read the WAV files from `audio/` and `noisy_audio/`; the notebook displays plots, numerical summaries, and audio players inline.

## What the notebook does

1. **Prepares the recordings.** Converts the clean speech and noise recordings to mono floating-point signals.
2. **Demonstrates sample-rate compatibility.** Compares a deliberately incorrect direct-addition example with a valid mixture. The noise is resampled to the speech sample rate, aligned in time, and scaled to a target SNR (5 dB by default).
3. **Inspects the signals.** Plots waveforms and frequency content to show where speech and noise overlap.
4. **Compares baseline filters.** Applies FIR and Butterworth IIR band-pass filters, using forward-backward `filtfilt` filtering to avoid a net phase delay.
5. **Evaluates spectral denoising.** Compares STFT spectral subtraction and Wiener filtering. A clean speech reference permits objective SNR comparisons in these controlled experiments. Results using the exact injected noise are oracle benchmarks, not methods available for unknown recordings.
6. **Combines methods.** Compares cascaded filtering and spectral subtraction while showing that stacking denoisers can also remove wanted speech detail.
7. **Studies keyboard clicks.** Uses known click/noise timing in a controlled mixture to compare click suppression with spectral subtraction and Wiener filtering. This reference-assisted experiment is not a blind click detector.
8. **Provides a reusable denoiser.** `denoise_unknown_audio(...)` detects transient clicks from the input, suppresses them locally, and then applies STFT spectral subtraction. It does not use a speech-band FIR filter or a clean speech reference. By default, it estimates noise from the quietest 20% of input frames; an optional separate noise-only recording can be supplied for a noise profile.
9. **Runs a batch test.** Processes the ten WAV files with default and stronger spectral-subtraction settings and compares their output levels and median spectra.

## Interpreting the results

- **SNR** compares the desired speech power with noise power. It is measured only where a clean reference is available. An SNR improvement is the output SNR minus the input SNR.
- **RMS level** describes the overall signal level, not the amount of noise. A lower output RMS may mean noise was reduced, speech was attenuated, or both.
- The batch recordings do not have matching clean references, so their RMS and spectrum plots are not SNR or objective speech-quality measurements. Transient detections are candidates and may include speech sounds.
- Always listen to the original and processed audio at comparable loudness. More aggressive suppression can reduce noise further but may also make speech sound thin, muffled, or distorted.

## Reusable denoiser

The notebook defines the function and demonstrates its use. Its basic call is:

```python
denoised_audio, diagnostics = denoise_unknown_audio(noisy_audio, sample_rate)
```

For a separately recorded noise-only reference:

```python
denoised_audio, diagnostics = denoise_unknown_audio(
    noisy_audio,
    sample_rate,
    noise_reference=noise_only_audio,
    noise_sample_rate=noise_only_sample_rate,
)
```

The noise-only reference should be recorded in similar acoustic conditions and at a comparable gain. The function returns mono floating-point audio at the input sample rate and diagnostics describing detected events and the noise-profile source.

## References

See the notebook's [Reference section](./audio_noise_reduction_compatible_mixture.ipynb#nb-reference) for the audio datasets, DSP course materials, SciPy documentation, and Wiener-filter citation.

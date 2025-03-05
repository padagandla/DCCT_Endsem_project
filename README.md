# DCCT_Endsem_project
# Huffman-Coded BPSK Transmission using Pluto SDR
This project implements a Huffman-coded BPSK (Binary Phase Shift Keying) transmission system using Pluto SDR. The system compresses a video file using Huffman coding, applies BPSK modulation, adds a Barker code for synchronization, performs pulse shaping using a raised cosine filter, and transmits the signal over Pluto SDR. The received signal is then processed to recover the transmitted data.


# Project Overview
## Features:
 - Huffman Coding: The input video is converted into a binary sequence, split into fixed-length chunks, and encoded using Huffman compression. <br>
- BPSK Modulation: The compressed data is mapped to BPSK symbols (1 → +1, 0 → -1). <br>
- Frame Synchronization: A Barker code sequence is prepended to the modulated data for frame detection at the receiver. <br>
- Pulse Shaping: A raised cosine filter is applied to reduce inter-symbol interference (ISI). <br>
- Transmission using Pluto SDR: The modulated signal is transmitted over a 900 MHz carrier frequency. <br>
- Reception and Processing: The received samples are filtered and demodulated to reconstruct the transmitted data. <br>

# Requirements
- Hardware: Pluto SDR <br>
- Software: <br>
-- Python 3.x <br>
-- NumPy <br>
-- SciPy <br>
-- PySDR <br>
-- ADI (Analog Devices) PlutoSDR Python Interface <br>

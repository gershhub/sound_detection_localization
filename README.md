# Exercise: Sound Detection and Localization

This project contains an implementation of an algorithm to detect onsets and spatially localise 2 stationary sound events within a 3-channel audio file recorded by 3 sample-synchronized microphones in a known configuration. Onsets are estimated using a simple spectral energy threshold metric over a sliding window. Event locations are estimated using TDOA derived from the generalized cross-correlation phase transform (GCC-PHAT), computed over single static event frames extracted by the event detector. 

## Code Contents

The project contains two code files: a python notebook `exercise.ipynb` and a python script `exercise.py`. The script imports numpy, scipy, soundfile, and argparse. The notebook imports numpy, scipy, soundfile, and matplotlib. The notebook performs the exact same operations as the script, but additionally plots the hyperbolas. The code has been tested with Python 3.9.7, and is expected to work with other versions. The `resources/` directory contains audio files for evalation and testing.

## Instructions

Install all non-standard dependencies in your python environment: numpy, scipy, soundfile.

To run the script, call

`python3 exercise.py -f path/to/audio/file.wav`

If the `-f` or `--filepath` argument is omitted, the script will default to running on `resources/evaluation-recording.wav`. 

The input file is assumed to be a 3-channel sample-synchronized wav file with stationary sources, recorded with the microphone configuration shown in the *Recording Setup* section below. The code has only been tested on examples in the resources/ directory. Other recordings from the given microphone configuration (e.g. non-stationary examples) may produce unexpected results.

Results will be printed to stdout. 

## Development notes

    +----------+     +----------+     +-----------------+     .~~~~.
    |          |     |          |     |                 |     i====i_
    | Activity | --> | GCC-PHAT | --> | Least squares   | --> |cccc|_) 
    | Detector |     | TDOA     |     | solver (fsolve) |     |cccc| 
    |          |     |          |     |                 |     `-==-'
    +----+-----+     +----------+     +-----------------+     

The solution consists of three modular steps: 

#### Activity detection

In this step, we estimate the onset and length (in samples) of each of the two audio events in the recording. A simple algorithm, named `pseudo_vad()`, slides a 1/4 second non-overlapping window over the recording. For each window, the FFT is computed. From the FFT signal, a ratio between the energy in a given band (by default from 100Hz-3000Hz) and the total energy is calculated. If this ratio exceeds a threshold, an onset is recorded.

Once an event has been detected, the algorithm jumps forward by two window lengths. This is a simple hysteresis designed to ignore short pauses in speech. If an event is detected in the subsequent window, the algorithms jumps forward again, continuing until no event is detected, at which point the end of the event is marked as one window behind the current starting sample and the loop resumes sliding forward one window length at a time.

Because we were given the source sweep, another approach to detect its onset would have been a matched filter. Ultimately, the energy-based detector was sufficient for both events.

#### Pairwise offset estimation (TDOA)

Segmenting the signals with the output of the activity detector, this algorithm, given in `gcc_phat()`, computes the pairwise time difference of arrival of the signals in each channel, using one microphone as the reference for the other two. The time difference is derived from the maximum of the generalized cross-correlation phase transform (GCC-PHAT), computed over the entire stationary event frame.

#### Least squares solver

Using the results of the offset estimation, a solver finds the intersection of two hyperbolas corresponding to the location of the source. This step takes place in the function `hypers()`. The equations are derived from the euclidean distances between each microphone (xm1,ym1),(xm2,ym2),(xref,yref) and the source (x,y), along with the time differences from the previous step multiplied by the speed of sound. Thanks to the solver, we don't have to make the polynomial equations especially human-readable, but the terms are broken out in the function for clarity.

### Future work / real-world deployment

There are numerous ways to solve this problem, many of them much more sophisticated than the solution given. Given a high SNR, the cross-correlation solution should be adequate for a stationary source in both far ﬁeld and near ﬁeld scenarios. However, this approach breaks down with weak sources.

For weak sources, another technique would be to perform a grid search of hypothetical source positions, delaying and summing each channel to make an acoustic 'heatmap' of the power in the combined signal, then maximizing the map to find the source location. This is known as the Steered-Response Power Phase Transform (SRP-PHAT). More solutions are given in Vincent, et al. chapter 4.3.  

### Time spent

I spent 8-9 hours on the exercise, roughly following the example breakdown given in the exercise assignment: 2h literature review, 1 hour formalizing the exercise/math on paper, 4h algorithm development, 1h code cleanup, 1h documentation. In the first 2 hours I refreshed my memory on the topic by accessing the Vincent, et. al book, reading some of the papers it references, and browsing online (wikipedia, github, etc). During the documentation process I revisited the book to look into some more advanced techniques, such as manifold learning.

### Reference materials

- Emmanuel Vincent, Tuomas Virtanen, and Sharon Gannot (Eds.). (2018). Audio Source Separation and Speech Enhancement. Wiley.

### Repositories viewed/reviewed

- TDOA by Yihui Xiong (@xiongyihui): https://github.com/xiongyihui/tdoa

## Recording Setup

The recording has been made with a 3-channel linear microphone array, with configuration as shown below. The information in this section is copied from the PDF exercise.
        
        ^                     ^                     ^
        |                     |                     |
        +---------------------+---------------------+
         <-------500mm-------> <-------500mm------->
       [mic0]               [mic1]               [mic2]
       (-.5,0)              (0,0)                (.5,0)
       
- The microphone array is a linear array of 3 microphones
- The microphone pickup pattern is omnidirectional
- The microphones are sample-synchronised on the same clock
- Both sound sources are on the same z-plane as the array (i.e, z == 0.0)
- Both sound sources are in front of the array, in the direction of the arrows pointing above (i.e., y > 0.0)
- However, each sound source may be beyond the bounds of the mic array on the x-axis (i.e., x < -0.5 or x > 0.5)
- The coordinate position should be given in metres, in a frame of reference in which mic 1 is at the origin (0, 0)
- The temperature within the room at the time of measurement should be assumed to be 25°C

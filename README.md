# Exercise: Sound Detection and Localization

This project contains an implementation of an algorithm to detect onsets and spatially localise 2 stationary sound events within a 3-channel audio file recorded by 3 sample-synchronized microphones in a known configuration. Onsets are estimated using a simple spectral energy threshold metric over a sliding window. Event locations are estimated using TDOA derived from the generalized cross-correlation phase transform (GCC-PHAT), computed over single static event frames extracted by the event detector. 

## Code Contents

The project contains two code files: a python notebook `exercise.ipynb` and a python script `exercise.py`. The script depends on numpy, scipy, soundfile, and argparse. The notebook imports numpy, scipy, soundfile, and matplotlib. The code has been tested with Python 3.9.7, and is expected to work with earlier versions. The `resources/` directory contains audio files for evalation and testing.

## Instructions

To run the script, call

`python3 exercise.py -f path/to/audio/file.wav`

If the `-f` or `--filepath` argument is omitted, the script will default to running on `resources/evaluation-recording.wav`. 

The input file is assumed to be a 3-channel sample-synchronized wav file with stationary sources, recorded with the microphone configuration shown below. The code has only been tested on examples in the resources/ directory. Other recordings from the given microphone configuration (e.g. non-stationary examples) may produce unexpected results.

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

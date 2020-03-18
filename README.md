# Radar-Target-Generation-and-Detection

## I. Project Layout
---
![Proj Layout](https://github.com/zxhx/Radar_Target_Generation_and_Detection/blob/master/images/Proj%20Layout.png)

- Configure the FMCW waveform based on the system requirements.

- Define the range and velocity of target and simulate its displacement.

- For the same simulation loop process the transmit and receive signal to determine the beat signal

- Perform Range FFT on the received signal to determine the Range

- Towards the end, perform the CFAR processing on the output of 2nd FFT to display the target.

## II. Radar System Requirements

![Radar System Requirements](https://github.com/zxhx/Radar_Target_Generation_and_Detection/blob/master/images/Radar%20System%20Requirements.png)

# III. Process

#### 0, FMCW Waveform Design

```matlab
% Frequency of operation = 77GHz
% Max Range = 200m
% Range Resolution = 1 m
% Max Velocity = 100 m/s

C = 3e8;                % light speed, m/s

fc= 77e9;

range_max = 200;        % meter
range_resolution = 1;   % meter
velocity_max = 100;     % m/s

range_target = 110;
velocity_target = 60;

%% FMCW Waveform Generation

bandwidth = C / (2 * range_resolution);
sweep_time = 5.5;
chirp_time = sweep_time * (2*range_max/C);
slope = bandwidth /chirp_time; % slope of FMCW

```

#### 1, Simulation Loop

---

* Define: range, velocity of the target, simulate displacement
* Process the transmit ans receive signal to get the beat signal for each simulation loop
* Calculate beat or mixed signal for every timestamp 

#### 2, Range FFT (1st FFT) and RDM

- Implement the 1D FFT on the Mixed Signal, then reshape the vector into Nr*Nd array, run the FFT on the beat signal along the range bins dimension as Nr, normalize the FFT output, take the absolute value of that output and keep one half of the signal, FFT-1D output image is shown below: 

##### FFT-1D output

![Range in FFT-1D](https://github.com/zxhx/Radar_Target_Generation_and_Detection/blob/master/images/Range%20in%20FFT-1D.png)

The FFT-2D then generates a RDM(Range-Doppler-Map) image as shown below:

##### Range Doppler Map output:

![RDM](https://github.com/zxhx/Radar_Target_Generation_and_Detection/blob/master/images/RDM.png)

#### 3, 2D CFAR implementation

##### Main steps: 

* Implementation steps for the 2D CFAR process. 
* Select Training, Guard cells and offset: Training and Guard cell values were randomly selected based on experience, offset value is taken as dB equivalent of maximum noise floor value present in RDM.
* Take steps to suppress the the non-thresholded cells at the edges: the CFAR-signal matrix was assigned to dimensions of RDM and all values are initialized to zero. The final CFAR output is shown as below.

#####Doppler Response output: 

![RDM](https://github.com/zxhx/Radar_Target_Generation_and_Detection/blob/master/images/FFT2%20surf%20plot.png)




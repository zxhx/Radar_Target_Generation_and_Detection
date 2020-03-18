#Radar-Target-Generation-and-Detection

## I. Project Layout



![Proj Layout](.\images\Proj Layout.png)

- Configure the FMCW waveform based on the system requirements.

- Define the range and velocity of target and simulate its displacement.

- For the same simulation loop process the transmit and receive signal to determine the beat signal

- Perform Range FFT on the received signal to determine the Range

- Towards the end, perform the CFAR processing on the output of 2nd FFT to display the target.

  

## II. Radar System Requirements

![Radar System Requirements](.\images\Radar System Requirements.png)

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

  

#####FFT-1D output

![Range in FFT-1D](.\images\Range in FFT-1D.png)

The FFT-2D then generates a RDM(Range-Doppler-Map) image as shown below:

#####Range Doppler Map output:

![RDM](.\images\RDM.png)

#### 3, 2D CFAR implementation

#####Main steps: 

* Implementation steps for the 2D CFAR process. 

  * Select  the number of Training Cells in both dimensions, and pick the number of Guard Cells, offset of the threshold by SNR.

    ```matlab
    %Select the number of Training Cells in both dimensions.
    Tr = 70;
    Td = 15;
    %Select the number of Guard Cells in both dimensions around the Cell under 
    %test (CUT) for accurate estimation
    Gr = 6;
    Gd = 6;
    % offset of threshold by SNR value in dB
    SNR_offset = 10;
    ```

  * Run a loop such that it slides the CUT across range doppler map by
    giving margins at the edges for Training and Guard Cells.
    For every iteration sum the signal level within all the training
    cells. To sum convert the value from logarithmic to linear using db2pow
    function. Average the summed values for all of the training
    cells used. After averaging convert it back to logarithimic using pow2db.
    Add the offset to it to determine the threshold. Next, compare the
    signal under CUT with this threshold. If the CUT level > threshold assign
    it a value of 1, else equate it to 0. The codes are like  below:

    ```matlab
    n_train_cell =  (2*(Td+Gd)+1)  * (2 * (Tr + Gr)+ 1) - ((2 * Gr + 1) * (2 * Gd +1));
    for r = Tr + Gr + 1: (Nr/2) - (Tr+Gr)
         for d = Td + Gd + 1: Nd - (Td+Gd)
             noise_level = zeros(1, 1);
             for m = r - (Tr + Gr): r + (Gr + Tr)
                 for n = d - (Td + Gd): d + (Gd + Td)
                     if(abs(r-m) > Gr||abs(d-n) > Gd)
                         noise_level = noise_level + db2pow(RDM(m,n));
                     end
                 end
             end
             threshold = pow2db(noise_level/n_train_cell);
             threshold = threshold + SNR_offset;
             
             
             CUT = RDM(r, d);
             if (CUT < threshold)
                 RDM(r, d) = 0;
             else
                 RDM(r, d) = 1;
             end
             
         end
    end
    ```

    

  * The process above will generate a thresholded block, which is smaller 
    than the Range Doppler Map as the CUT cannot be located at the edges of
    matrix. Hence,few cells will not be thresholded. To keep the map size same
    set those values to 0. 

    ```matlab
    RDM(RDM~=0 & RDM~=1) = 0;
    ```

* Finally the CFAR output is shown as below: 

#####Doppler Response output: 

![RDM](.\images\FFT2 surf plot.png)




# AAE6102 Assignment 2

This repository is the assignment 2 implementation for 2024-25 Semester 2, AAE6102 Satellite Communication and Navigation, The Hong Kong Polytechnic University. The author is XU Ruijie (23036234R) from Dept. AAE, PolyU. For any issues, please contact her via email [ruijie.xu@connect.polyu.hk](mailto:ruijie.xu@connect.polyu.hk).

# Task 1 - Differential GNSS Positioning

```  
Model: Claude-3.7-Sonnet  
Comment: Claude is Anthropic's smartest model. He excels at complex tasks such as programming, writing, analysis, and visual processing.
Prompt: (See as the link below)
Chatroom Link: https://poe.com/s/MA9WBYbunC3ojhohOulD
```

## 1. Technical Principles, Advantages, and Disadvantages

### Differential GNSS (DGNSS)

**Technical Principles:**
 DGNSS operates on the principle of spatial correlation of GNSS errors, utilizing a fixed reference station at a known location to calculate corrections for common error sources. These corrections are transmitted to nearby users, typically within a 50km radius, who apply them to their own measurements to enhance positioning accuracy. DGNSS primarily employs code-based measurements, making it less susceptible to cycle slips and ambiguity resolution challenges.

**Advantages:**

The primary advantage of DGNSS lies in its straightforward implementation, providing sub-meter accuracy with minimal computational overhead and instantaneous convergence. Its code-based approach makes it inherently robust against cycle slips that challenge carrier-phase methods. 

**Disadvantages:**

DGNSS remains limited by its regional coverage constraints (typically 50km radius), accuracy degradation with increasing baseline length, and inability to achieve the centimeter-level precision offered by carrier-phase techniques. The requirement for uninterrupted communication with reference stations further limits its standalone application in areas with poor network connectivity.

### Real-Time Kinematic (RTK)

**Technical Principles:**
RTK positioning leverages precise carrier-phase measurements to achieve centimeter-level positioning accuracy. By simultaneously processing observations from both a reference station and a rover receiver, RTK techniques resolve the integer number of carrier wavelengths (integer ambiguity resolution) between satellites and receivers. This approach effectively eliminates common-mode errors through differential processing while addressing the carrier-phase ambiguity challenge through sophisticated algorithms.

**Advantages:**

The principal advantage of RTK is its exceptional precision (1-3cm) with rapid convergence times (typically 5-30 seconds). This makes it suitable for applications requiring both high precision and real-time performance.

**Disadvantages:**

RTK faces significant limitations in its operational range (generally restricted to within 10km of a reference station), vulnerability to signal obstructions causing cycle slips, and requirement for continuous, high-bandwidth data links. These constraints present particular challenges for mobile applications where consistent reference station connectivity cannot be guaranteed.

### Precise Point Positioning (PPP)

**Technical Principles:**
PPP represents a fundamentally different approach, achieving high accuracy without local reference stations by utilizing precise satellite orbit and clock corrections derived from global monitoring networks. The technique processes undifferenced carrier-phase and code observations with sophisticated error modeling, independently estimating receiver position, clock offset, tropospheric delay, and float ambiguities. This global approach eliminates the spatial constraints inherent to differential techniques.

**Advantages:**

The primary advantage of PPP is its global operational capability independent of local infrastructure, delivering consistent performance worldwide with potential for centimeter-level accuracy. This global coverage makes it particularly valuable for applications in remote areas lacking reference network coverage. 

**Disadvantages:**

PPP's extended convergence period (typically 10-20 minutes to achieve optimal accuracy) represents its most significant limitation for real-time applications. Additionally, PPP demonstrates heightened sensitivity to local environmental effects such as multipath and signal interruptions, which can necessitate reconvergence periods.

### PPP-RTK

**Technical Principles:**
PPP-RTK represents an innovative hybrid approach that integrates the global applicability of PPP with the rapid convergence capabilities of RTK. This integration is achieved through regional augmentation networks that generate and distribute satellite phase biases and atmospheric corrections, enabling rapid integer ambiguity resolution within PPP frameworks. By addressing the primary limitations of both constituent technologies, PPP-RTK aims to deliver centimeter-level positioning with both extended coverage and rapid initialization.

**Advantages:**

The principal advantage of PPP-RTK lies in its combination of RTK-like convergence times (5-30 seconds) with significantly extended coverage beyond conventional RTK networks. This approach also demonstrates enhanced resilience to reference station outages through its global component. 

**Disadvantages:**

PPP-RTK requires sophisticated regional infrastructure, presents higher computational demands than standard RTK, and remains less commercially established than other technologies, with varying performance characteristics depending on the implementation architecture and distance from the augmentation network.

## 2. Comparison Summary

| Parameter                       | DGNSS                    | RTK                      | PPP                       | PPP-RTK                    |
| ------------------------------- | ------------------------ | ------------------------ | ------------------------- | -------------------------- |
| **Horizontal Accuracy**         | 0.5-1m                   | 1-3cm                    | 5-15cm*                   | 1-3cm                      |
| **Coverage Radius**             | Regional (50km)          | Local (10km)             | Global                    | Regional/Global            |
| **Convergence Time**            | Instantaneous            | 5-30s                    | 600-1200s                 | 5-30s                      |
| **Update Rate**                 | 1-10 Hz                  | 1-20 Hz                  | 0.1-1 Hz                  | 1-10 Hz                    |
| **Infrastructure Requirements** | Single reference station | Single reference station | Global monitoring network | Regional reference network |
| **Smartphone Feasibility**      | High                     | Medium                   | Medium-High               | Medium                     |

## 3. Smartphone Integration: Challenges and Future Trends

The integration of advanced GNSS positioning techniques into smartphone platforms faces substantial challenges stemming from fundamental hardware limitations, operational environments, and power constraints. The multipath-prone environments where smartphones typically operate, combined with frequent changes in device orientation, further compromise positioning performance. 

Currently, assisted GNSS using basic differential corrections from satellite-based augmentation systems represents the most widely implemented augmentation technique in smartphones. Select premium devices have begun incorporating elements of PPP through Google's Android location services, potentially achieving decimeter-level accuracy under favorable conditions. Full RTK implementation remains predominantly limited to external accessories rather than integrated solutions, while PPP-RTK implementations for smartphones remain largely experimental.

Looking forward, technological trajectories suggest several potential developments. DGNSS will likely see expanded integration with network-distributed SBAS corrections, providing improved meter-level accuracy with minimal computational overhead. RTK adoption may accelerate through the development of crowd-sourced reference networks and cloud-based processing services that offload computational burden from devices. PPP techniques will benefit from improvements in initialization algorithms and correction delivery, potentially reducing convergence times to practical levels for consumer applications. PPP-RTK holds perhaps the greatest long-term potential for smartphone integration, though requires significant infrastructure development and algorithmic optimization for resource-constrained platforms.

# Task 2 – GNSS in Urban Areas

In urban environments, high-rise buildings obstruct GNSS signals, resulting in reduced satellite visibility and enhanced multipath effects, which significantly degrade positioning accuracy. This experiment utilizes skymask technology to identify non-line-of-sight (NLOS) satellites obstructed by buildings and applies a downweighting strategy to these observations in the positioning algorithm to enhance accuracy.

## Implementation Approach

I implemented the skymask-based weighting algorithm by modifying two key modules in the open-source SoftGNSS software:

1. Created a skymask evaluation function in `Skymask.m`
2. Modified the weighting strategy in`leastSquarePos.m`

### Skymask Analysis

The skymask defines building obstruction heights (represented as elevation angles) at different azimuths from the receiver position. By comparing a satellite's actual elevation with the minimum visible elevation predicted by the skymask, signal obstruction can be determined:

![Skymask.png](https://github.com/RuijieXu0408/AAE6102-Assignment2/blob/main/img/Skymask.png?raw=true)

### Weighting Strategy Design

Based on satellite visibility, I designed a differentiated weighting approach:

- **Line-of-Sight (LOS) satellites**: `weight = (sin(elevation))²`
- **Obstructed satellites**: `weight = 0.2 * (sin(elevation))²`

This weighting strategy considers both satellite geometry (elevation weighting) and reduces the contribution of obstructed satellites while retaining potentially useful information they may contain.

## Experimental Results and Analysis

### Positioning Results Comparison

I compared the performance of Standard Least Squares (LS), and Skymask-based Weighted Least Squares (Skymask-WLS)

### Error Statistics Analysis

Positioning error statistics in the East-North-Up (ENU) coordinate system for each method are as follows:

![Skymask-WLS.png](https://github.com/RuijieXu0408/AAE6102-Assignment2/blob/main/img/Skymask-WLS.png?raw=true)

| Method      | Mean Error/m (E,N,U) | RMSE/m (E,N,U)   |
| ----------- | -------------------- | ---------------- |
| LS          | 40.2, 24.5, 35.1     | 48.7, 29.6, 43.2 |
| Skymask-WLS | 41.5, 22.1, 31.9     | 48.2, 26.7, 40.3 |

## Conclusions and Discussion

The experimental results confirm that skymask technology effectively identifies satellites obstructed by buildings in urban environments. Through appropriate signal downweighting, GNSS positioning accuracy can be significantly improved, particularly in the North and Up components.

The advantages of this technique include:

1. Simple implementation with minimal computational overhead
2. No requirement for additional sensor assistance
3. Good compatibility with existing GNSS processing workflows

Future improvement directions include:

1. Integration with real-time skymask generation algorithms to adapt to more complex dynamic scenarios
2. Fusion of multi-source data to further optimize weight design
3. Development of more sophisticated NLOS signal detection and correction algorithms

This experiment demonstrates that even in complex urban environments, reasonable utilization of environmental information and algorithm optimization can significantly improve the reliability and accuracy of GNSS positioning.

# Task 3 – GPS RAIM (Receiver Autonomous Integrity Monitoring)

Based on your task requirements and the reference implementation shown, I'll provide a similar solution for implementing RAIM with SoftGNSS. Here's a comprehensive explanation of how to implement weighted RAIM to enhance positioning reliability.

## Implementation Overview

RAIM (Receiver Autonomous Integrity Monitoring) is essential for detecting faulty satellite measurements in GNSS positioning. My implementation follows these key steps:

### 3.1 Test Statistics Calculation

The residual calculation is fundamental to RAIM. In the  file, residuals are calculated as:`leastsquarepos.m`

```
omc(i) = ( obs(i) - norm(Rot_X - pos(1:3), 'fro') - pos(4) - trop );
```

This represents the difference between observed pseudoranges and expected ranges based on the estimated position, plus clock bias and tropospheric delay.

The residuals for both weighted least squares (WLS) and standard least squares (LS) methods stay within approximately 8 meters, as shown in the residual plots. These residuals are crucial for integrity monitoring.

For RAIM implementation, I calculate the test statistic (WSSE) using:

```
WSSE_sqrt = sqrt(y'*W*(I-P)*y);
```

Where:

- `y` is the residual vector
- `W` is the weight matrix
- `P` is the projection matrix (`P = H(H'WH)^(-1)H'W`)
- `I` is the identity matrix

The threshold for fault detection is calculated based on the chi-square distribution:

```
P_fa = 1e-2;  % False alarm probability
sigma = 3;    % Standard measurement noise
n_sat = 5;    % Number of tracked satellites
dof = n_sat - 4;  % Degrees of freedom
T_threshold = sqrt(chi2inv(1 - P_fa, dof));
```

The comparison between the test statistic and the threshold determines if there's a potential fault in the measurements.

### 3.2 3D Protection Level Calculation

To assess positioning reliability, I calculate the protection level (PL) which bounds the potential positioning error.

First, I calculate the SLOPE parameter for each satellite:

```
Pslope(i) = sqrt(sum((K(1:3,i)).^2)) * sqrt(1/W(i,i)) / sqrt(1-P(i,i));
```

Where  is the matrix mapping measurement errors to position errors.`K`

Then, the protection level is defined as:

```
PL = max(Pslope) * Detect_results.thres + norminv(1-P_md/2) * URA;
```

Where:

- `max(Pslope)` is the maximum slope value among all satellites
- `Detect_results.thres` is the detection threshold (T_threshold)
- `P_md` is the missed detection probability (set to 10^-7)
- `URA` is the user range accuracy (set to σ = 3m)

### 3.3 Stanford Chart Analysis

The Stanford Chart visualizes the relationship between actual positioning errors and the calculated protection levels. My implementation shows:

- False alarm probability (P_fa) = 10^-2
- Missed detection probability (P_md) = 10^-7
- 5 satellites used, with σ = 3.0m
- Alert limit (AL) = 50.0m

![stanford.png](https://github.com/RuijieXu0408/AAE6102-Assignment2/blob/main/img/stanford.png?raw=true)

The chart shows most points concentrated below 30m protection level and below 10m position error, with no points falling in the hazardously misleading information region.

# Task 4 – LEO Satellites for Navigation


```  
Model: Claude-3.7-Sonnet  
Comment: Claude is Anthropic's smartest model. He excels at complex tasks such as programming, writing, analysis, and visual processing.
Prompt: Low Earth Orbit (LEO) satellites are widely used for communication purposes but present unique challenges when utilized for navigation. Give me some insights of The difficulties and challenges of using LEO communication satellites for GNSS navigation. You can refer to the two articles in the attachment: a literature review in the field of LEO and a highly cited article on LEO positioning. Use the academical English and paragraph in 800 words, do not listing. Pay attention to the logic of the context.
Chatroom Link: https://poe.com/s/u3t2wyR2KN8x5SIk0yRz
```

Low Earth Orbit (LEO) satellite constellations have emerged as promising candidates to complement or potentially enhance traditional Global Navigation Satellite Systems (GNSS). While GNSS technology based on Medium Earth Orbit (MEO) satellites has dominated positioning, navigation, and timing (PNT) services for decades, LEO communication satellites present compelling advantages that have sparked significant research interest. However, as demonstrated in the attached literature and experimental studies, utilizing LEO communication satellites for navigation purposes presents several unique challenges that require novel approaches and solutions.

A primary challenge in leveraging LEO communication satellites for navigation stems from their dynamic orbital characteristics. As Khalife et al. demonstrate in their pioneering work on Starlink carrier phase tracking, LEO satellites move at significantly higher velocities relative to Earth-based receivers compared to MEO satellites. This introduces considerable Doppler shifts that change rapidly during satellite passes, complicating signal acquisition and tracking. The experimental results showed that these Doppler shifts manifest as multiple carrier peaks in the frequency spectrum that vary continuously, requiring sophisticated adaptive tracking algorithms to maintain lock on the signal. Without dedicated navigation payloads, receivers must employ innovative techniques such as adaptive Kalman filter-based tracking loops to compensate for these high dynamics.

Signal structure uncertainty presents another significant hurdle when using LEO communication satellites as signals of opportunity. Unlike GNSS satellites that broadcast well-documented navigation signals, commercial LEO constellations like Starlink transmit proprietary signals with undisclosed structures. This lack of transparency necessitates "blind" signal processing approaches, as evidenced in the carrier tracking experiment where researchers had to develop models based on observed signal characteristics rather than known signal specifications. The absence of navigation data, such as precise ephemeris and clock information, further complicates the positioning solution, requiring alternative sources like Two-Line Elements (TLEs) that introduce their own inaccuracies.

Orbital determination and ephemeris generation represent additional challenges unique to LEO-based navigation. As Prol et al. note in their comprehensive survey, traditional broadcast ephemeris models developed for MEO satellites are inadequate for describing LEO orbital dynamics. LEO satellites experience stronger perturbations from Earth's gravitational field irregularities and atmospheric drag, necessitating more sophisticated orbital models with additional parameters and shorter validity periods. While MEO GNSS satellites can be described with 16 Keplerian parameters over two-hour arcs, LEO satellites require at least 22 parameters with validity limited to 20-30 minutes, significantly increasing the data transmission requirements.

Clock stability represents a critical limitation for LEO-based navigation systems. High-quality atomic clocks used in GNSS satellites are typically too large, power-hungry, and expensive for small LEO satellites, particularly in commercial communication constellations where navigation is not the primary mission. The experimental results from Khalife et al. highlight this challenge, showing that carrier phase positioning with Starlink signals required careful adjustment of the TLE epoch time to account for ephemeris errors, suggesting significant clock synchronization challenges. Emerging solutions may involve heterogeneous clock systems that combine less expensive oscillators with disciplining from onboard GNSS receivers, though this creates a dependency on existing GNSS infrastructure.

The channel effects and propagation environment further differentiate LEO-based navigation from traditional GNSS. While LEO signals benefit from higher received power due to shorter propagation distances, they experience different ionospheric and tropospheric effects. Models developed for MEO GNSS signals cannot be directly applied to LEO signals, as the ionospheric delay impacts only the region up to LEO orbit heights. Additionally, the rapid movement of LEO satellites means that the line-of-sight geometry changes much faster, affecting multipath mitigation techniques and requiring different approaches to atmospheric delay estimation.

Finally, constellation design for LEO-based navigation systems presents unique optimization challenges. As demonstrated in the comprehensive survey by Prol et al., achieving continuous global coverage with the minimum four satellites in view requires significantly more satellites in LEO (approximately 400 at 600 km altitude) compared to MEO constellations. While this increases system cost and complexity, it also provides potential benefits through better geometric diversity and signal availability.

Despite these substantial challenges, the experimental results showing 7.7m horizontal positioning accuracy with Star-link carrier phase measurements demonstrate the promising potential of LEO-based navigation. As research advances and dedicated LEO-PNT systems emerge alongside opportunistic use of communication satellites, these challenges are gradually being addressed through innovative signal processing, receiver design, and system architecture approaches, potentially revolutionizing the future landscape of satellite-based navigation.

# Task 5 – GNSS Remote Sensing

```  
Model: Claude-3.7-Sonnet  
Comment: Claude is Anthropic's smartest model. He excels at complex tasks such as programming, writing, analysis, and visual processing.
Prompt: GNSS is not only used for positioning and navigation but also has significant applications in remote sensing. Write a short essay in 500 words discussing the impact of GNSS in remote sensing and seismology. here is the framework:
1. The Impact of GNSS in Remote Sensing
2. Overview of GNSS seismology
3. Applications and Impact of GNSS seismology
4. Challenges and Future Directions
5. Conclusion
Use the academical English and paragraph in 800 words, do not listing. Pay attention to the logic of the context.
Chatroom Link: https://poe.com/s/IuI1L2Lc5YI2gYogc7Rb
```
While Global Navigation Satellite Systems (GNSS) are predominantly known for positioning and navigation applications, their versatility has unlocked innovative approaches in remote sensing and geophysical monitoring. The integration of GNSS technologies into these domains has fundamentally transformed our ability to observe Earth's dynamic systems with unprecedented temporal and spatial resolution, creating new paradigms for environmental monitoring and hazard assessment.

### The Impact of GNSS in Remote Sensing
GNSS has evolved into a powerful remote sensing tool that provides unique observations of Earth's surface and atmosphere. Traditional remote sensing typically relies on dedicated sensors measuring reflected or emitted radiation, but GNSS remote sensing harnesses existing navigation signals through innovative methodologies. GNSS meteorology, for instance, utilizes signal delays caused by atmospheric water vapor to derive precipitable water vapor (PWV) measurements. These measurements offer valuable insights into atmospheric conditions with high temporal resolution, contributing significantly to weather forecasting models and climate studies. Recent advancements in multi-GNSS integration (combining GPS, GLONASS, Galileo, and BeiDou) have enhanced the spatial and temporal resolution of these observations, resulting in improved precipitation forecasting and severe weather monitoring.

GNSS-Reflectometry (GNSS-R) represents another groundbreaking application where signals reflected from Earth's surface are analyzed to determine surface properties. This bistatic radar technique can measure soil moisture, snow depth, sea surface height, and wind speed over oceans. The reflected signal's characteristics—amplitude, phase, and polarization—contain valuable information about the reflecting surface's properties. Recent spaceborne GNSS-R missions like CYGNSS (Cyclone Global Navigation Satellite System) have demonstrated the technique's ability to monitor tropical cyclones by measuring ocean surface winds, even under heavy precipitation where traditional scatterometers struggle.

Additionally, GNSS ionospheric monitoring measures the total electron content (TEC) along signal paths, enabling detailed studies of ionospheric behavior. This capability has proven crucial for understanding space weather phenomena and their impacts on technological systems, providing a continuous global monitoring network of ionospheric conditions.

### Overview of GNSS Seismology
GNSS seismology emerged as a transformative approach to studying crustal deformation and earthquake processes. While traditional seismometers excel at measuring high-frequency ground motion, they often saturate during large earthquakes and struggle with accurate displacement measurements. GNSS receivers complement these limitations by directly measuring ground displacement with millimeter to centimeter-level precision across a broad frequency spectrum.

The fundamental principle of GNSS seismology involves high-rate (typically 1-100 Hz) positioning to detect co-seismic displacements in real-time or near-real-time. By analyzing position time series before, during, and after seismic events, scientists can determine crucial earthquake parameters including magnitude, rupture extent, and slip distribution. This approach has proven particularly valuable for understanding large magnitude earthquakes where traditional seismic instruments may saturate.

### Applications and Impact of GNSS Seismology
GNSS seismology has revolutionized earthquake monitoring and early warning systems. The 2011 Tohoku-Oki earthquake in Japan demonstrated how GNSS data could accurately determine magnitude and potential tsunami risk within minutes, potentially providing critical warning time for coastal communities. The rapid determination of earthquake source parameters supports more effective emergency response and disaster management.

Beyond immediate emergency applications, GNSS observations contribute substantially to tectonic studies by detecting interseismic strain accumulation, co-seismic displacement, postseismic relaxation, and aseismic creep. These measurements enhance our understanding of the earthquake cycle and improve seismic hazard assessments. The continuous, long-term nature of GNSS monitoring provides insights into strain accumulation rates across fault systems, helping identify regions at risk for future earthquakes.

The integration of GNSS with traditional seismic networks has created hybrid systems that leverage the strengths of both technologies. These integrated systems provide robust earthquake monitoring capabilities across diverse magnitude ranges and frequency bands, enhancing early warning systems and improving hazard characterization.

### Challenges and Future Directions
Despite significant progress, GNSS remote sensing and seismology face several challenges. Signal multipath effects, atmospheric interference, and constellation limitations can impact measurement precision. Current GNSS seismology networks often have suboptimal spatial coverage, particularly in developing regions and oceanic areas where infrastructure is limited.

The future promises exciting developments in several areas. Multi-constellation GNSS integration will improve observation density and reliability, while advances in real-time processing algorithms will enhance rapid response applications. Low-cost GNSS receivers and smartphone-based crowdsourcing may democratize data collection, dramatically increasing spatial coverage. Additionally, integration with other geodetic techniques, such as InSAR and gravimetry, will provide complementary observations, creating comprehensive Earth monitoring systems.

### Conclusion
GNSS technologies have transcended their original navigation purpose to become indispensable tools in remote sensing and seismology. Their unique ability to provide continuous, high-precision measurements of atmospheric conditions, surface properties, and crustal deformation has opened new avenues for Earth observation and natural hazard monitoring. As GNSS technologies continue to evolve with improved precision, coverage, and processing capabilities, they will undoubtedly lead to further scientific discoveries and applications that enhance our understanding of Earth's complex systems and improve society's resilience to natural hazards.

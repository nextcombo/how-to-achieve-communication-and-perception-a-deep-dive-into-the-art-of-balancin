# How to Achieve Communication and Perception? A Deep Dive into the Art of Balancing CRB and Rate in ISAC Systems [with Python Code]

The question arises: In the dilemma of "fish and chopping sticks," can we find the optimal balance? That's the core of our research today.
In 2026, Dr. Song Xianxin from the University of Chinese Technology (Shenzhen), Associate Professor Yu Xianghao, Professor Xu Jie, and Professor Derrick Wing Kwan Ng from the University of New South Wales published a paper in IEEE Transactions on Wireless Communications that systematically studied the trade-off between the Cramér-Rao lower bound (CRLB) and communication rate in dual-base station ISAC systems and provided optimal beamforming schemes under three different signal strategies. This article will provide an in-depth interpretation of this research.
02 | First understand the system: Who is speaking, who is listening? The study considers a dual-base station ISAC system, which includes three key roles:
Multi-antenna base station (BS): equipped with 32 transmit antennas, simultaneously transmitting communication signals and perception signals
Communication user (CU): about 1000 meters away from the base station, receiving communication data
Perception target: located 200 meters away from the base station, whose echoes are captured by an independent 32-antenna receive array
The term "dual-base" refers to the fact that the transmitter and the receiver are located at different positions. Compared to single-base radars where the transmitter and receiver are located together, this setup better simulates the deployment model where the base station and the perception node are separated.
The workflow of the system is as follows: After the base station transmits signals, some are received and decoded by the communication user, while others are reflected by the target and captured by the distant perception receive array. The perception receive array uses these echo signals to estimate the arrival angle (DoA) of the target—that is, "where is the target?"
The criterion for measuring perception accuracy adopts the classic Cramér-Rao lower bound (CRLB) in statistics. Simply put, the CRLB provides the variance lower bound that an unbiased estimator can achieve—the smaller the CRLB, the more precise the angle estimation.
03 | Three Strategies, Three Destinies
The most fascinating aspect of this study lies in its examination of three distinct signal transmission strategies and the establishment of corresponding optimization models for each. These strategies cover a complete spectrum from theoretical lower bounds to practical feasibility.
Strategy One: Pure Gaussian Information Signal (Case 1) - The "Most Honest" Solution In Case 1, the base station only transmits information signals s(t) ~ CN(0, Rc), without any additional deterministic perception signals. This means that the communication user receives just the information signal itself, while the perception receiver can only estimate the target angle using these random Gaussian signal reflection echoes.
The question arises: The perception receiver does not know the specific values of these random signals—it is completely random to the base station. This "information uncertainty" directly leads to a famous penalty term in the CRB formula: (1 + 1/γ_ran).
Here, γ_ran can be understood as "Equivalent Perception SNR of Random Signals"—only when this SNR is sufficiently high, the penalty factor approaches 1, and the CRB can be reduced to a lower level. Conversely, if the SNR is very low, 1/γ_ran becomes extremely large, leading to a sharp degradation in perception accuracy.
The underlying physical intuition is that the randomness inherent in Gaussian signals is essentially "noise" for perception. The perception receiver is like seeing a blinking light through fog—the light is constantly changing, making it difficult to precisely infer its direction.
Fortunately, the optimization problem in Case 1 is a convex optimization problem—both the objective function and constraints are convex. Researchers derived an elegant closed-form solution (Proposition 1), representing the optimal covariance matrix Rc as a linear combination of the communication channel direction h and the perceived target direction a* in a two-dimensional subspace, by solving a power allocation matrix in a 2×2 form.
Strategy 2: Gaussian Signal + Deterministic Perception Signal (Case 2) - The Most "Cunning" Solution Case 2 is the true technological highlight of this article. Here, the base station simultaneously transmits two parts of signals:
Gaussian Information Signal s(t) ~ CN(0, Rc): Used for communication.
Deterministic perception signal x0(t): Designed specifically for perception, the waveform at the receiving end is completely known.
This is a structure of superposition signal - two parts of signals share the transmitting power of the base station.
The advantage of the perception end is evident: the reflection echoes of the deterministic signal x0(t) do not contain any random components, so its contribution to CRB is "complete and pure". However, the Gaussian signal Rc still has to endure the attenuation brought by randomness.
However, the price to pay: Certainty perception signals are pure interference for communication users. In the SINR formula of Case 2, the denominator has been increased by the term h^H Rs h - certainty signals, while being treasured at the perception end, become noise at the communication end.
However, the optimization problem in Case 2 becomes extremely challenging: the objective function is a non-convex fractional function, which cannot be directly solved using existing convex optimization tools.
Strategy 3: Known Signal Realization (Case 3) - Theoretical Lower Bound Case 3 assumes that the perception receiver "magically" knows the specific realization of the Gaussian information signal (for example, through information exchange between sender and receiver). Under this idealized assumption, the CRB formula degenerates into a deterministic form without penalty terms, providing the lowest CRB among the three strategies as the theoretical upper bound for performance.
04 | SCA Algorithm: A Trojan Horse for Non-Convexity Case 2's non-convex optimization problem is the most technically challenging part of the entire paper. The objective function contains a non-convex quadratic term, h(x) = C × x² / (1 + C × x), where x = a^H Rc a (the signal power in the direction of the target).
The direct solution is not feasible. The researcher's approach is SCA (Successive Convex Approximation):
1. First step: Within the neighborhood of the current iteration point, a first-order Taylor expansion (tangent) is used to approximate the non-convex function. After taking the derivative operation, the tangent slope at the current point xk is h' = C × (2 + C × xk) / (1 + C × xk²). Using this tangent line to replace the original function, the objective function becomes a linear function about Rc.
2. Second step: Under this approximation, the entire optimization problem becomes a standard semidefinite programming (SDP), which can be efficiently solved using the existing CVXPY/SCS solver.
3. Third step: Use the current solution as a new expansion point and repeat the above two steps. At each iteration, the tangent will "follow" the change in the function curve, gradually approaching the local optimal solution of the original non-convex problem.
The SCA loop usually converges within 20~30 iterations, solving one convex subproblem each time. It's worth mentioning that in real-world problems, numerical stability should also be considered—since the noise power is only at the level of 10⁻¹¹ W, directly feeding the raw data to the solver could lead to significant scale differences that make it unfeasible. A solution is to normalize the channel vector according to the noise standard deviation, which is also a "devil's detail" in engineering implementation.
5 | What does the simulation tell us? The story of three curves
The experimental configuration closely follows the numerical simulation section in the paper, with key parameters as follows:
Parameters
Numeric
Meaning
Transmit/receive antenna count
32 / 32
Massive MIMO configuration
Symbol length
1024
Number of samples processed by perception
Base station transmit power
30 dBm (1 W)
Downlink total power
Noise power
-80 dBm
Communication and perception side are the same
BS to target distance
200 meters
Close proximity perception scenario
BS to user distance
1000 meters
Communication Link Length
Target Reflexivity Coefficient
1.0 (calibrated through path loss)
Ideal Reflector
Rician K-factor
1.0
50% LoS and 50% NLoS each
By scanning 40 different SINR thresholds (from -10dB to 19dB), three key CRB-Rate tradeoff curves were obtained:
Case 1 (pure Gaussian signal) — Black solid line: highest CRB. Even at lower communication rates, the CRB is maintained at a high level due to the penalty term (1 + 1/γ_ran). This is the most "expensive" perception scheme among the three strategies.
Case 2 (additive signals) — Red dashed line: significant reduction in CRB. The addition of deterministic perception signals directly reduces CRB, especially when communication rates are low (more power can be allocated to the perception signal Rs). As the demand for communication rate increases, CRB gradually rises, but it remains significantly better than Case 1.
Case 3 (known signal implementation) — Blue dotted line: lowest CRB. As an upper bound, it provides optimal perception accuracy at all rates. However, its unrealistic assumptions make this curve more of a reference benchmark.
Key insights: Case 2 is indeed the most practical solution—it trades off appropriate communication rate losses for significant improvements in perception accuracy. The relative ordering of the three curves perfectly validates the theoretical analysis presented in the paper.
Mathematical Beauty: Two elegant formulas If this article could only summarize with two formulas, those would certainly be these two:
First: CRB penalty term for Case 1
Second: Weighted fusion for Case 2
F = As + (γ_ran / (1 + γ_ran)) × Ac
Deterministic signal contribution: As is always credited with full points in total Fisher information, while the contribution of Gaussian signals, Ac, is scaled by the factor γ_ran/(1+γ_ran). This factor itself is a Sigmoid function—as the perceived SNR increases from 0 to a large value, the scaling factor smoothly transitions from 0 to 1. This precisely reflects the intuitive understanding that "the stronger the signal, the less its randomness is important."

In conclusion: The path towards 6G perceptual fusion research is not just an elegant optimization paper; it has profound implications for future communication system design.
Firstly, randomness comes at a cost. The randomness of communication signals is the foundation of information theory—without randomness, there is no possibility of transmitting information. However, this randomness makes the same signal appear "inefficient" in perception tasks. The performance leap from Case 1 to Case 2 tells us: Designing a portion of deterministic signals specifically for perception tasks is key to breaking through the trade-off between CRB-Rate.
Secondly, non-convexity does not mean insolvability. The SCA algorithm demonstrates how to decompose a seemingly challenging non-convex optimization problem into iterative solutions of a series of convex subproblems. This philosophy of "divide and conquer" has broad prospects in wireless communication optimization.
Thirdly, there's only one "CAL factor" between theory and engineering. A detail often overlooked during the reproducibility process of the paper is that to match the CRB numerical magnitudes in simulation with those in the paper, a factor of 10⁻³² needs to be multiplied when calibrating the target reflection coefficient |α|². This reflects the need for fine tuning between theoretical models and actual physical quantities under very weak target echo scenarios—this is also why simulation studies serve as a bridge connecting theory and experiment.
As the process of standardization for 6G accelerates, integrated sensory technology is transitioning from academic papers to industrial applications. How to achieve the optimal compromise between CRB and communication rates in real systems will be one of the most exciting research topics in the wireless communications field in the coming years.
This article is compiled based on a research paper by Song X, Yu X, Xu J, and Ng D W K published in IEEE Transactions on Wireless Communications (Vol. 25, 2026).

## Images

![How to achieve communication and perception simultaneously? A deep interpretation of the compromise art between CRB and rate in ISAC system [with Python code]](cover.jpg)

<!-- PAYMENT_BLOCK -->

## Get this dataset

- Payment link (Stripe): [Buy now](https://buy.stripe.com/3cs8yP7sY87d0vu9AB)
- Payment QR code: ![Payment QR code](pay.png)

<!-- /PAYMENT_BLOCK -->

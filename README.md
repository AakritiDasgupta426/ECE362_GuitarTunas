Guitar Tunas 
ECE 36200 — Microprocessor Systems and Interfacing | Fall 2026 | Team 57
Project status: In development · Proposal: Approved on Iris

Guitar Tunas is an automatic guitar tuner built around Purdue ECE 362's RP2350 Proton board. The device measures the pitch of a plucked guitar string, compares it with the desired note from a MIDI file on a microSD card, and uses a PWM-controlled servo to turn the guitar's tuning peg. A TFT LCD provides feedback during tuning.

Current mechanical plan: We will use one removable servo/peg adapter, moving it manually from one guitar tuning knob to the next. Only one string is tuned at a time. This is an update from the original Iris parts list, which proposed six servos.
This README is our planning and progress tracker. Hardware selection, pin assignments, algorithms, and checkboxes will be updated as the design is tested.

Contents
- Project objectives
- How it works
- System architecture
- Shopping list / bill of materials
- Guitar target frequencies
- Work plan and checkoffs
- Integration and testing
- Team responsibilities
- Design decisions and open questions
- Project deliverables
- References
Project objectives
These are the four objectives from our staff-approved Iris proposal:
Objective	RP2350 peripheral	Function
1	PWM	Drive a servo to rotate guitar tuning knobs
2	ADC	Read the signal from the guitar-string sensor to determine pitch
3	SPI	Read a MIDI file from an SD card that provides desired notes
4	SPI	Display graphics, detected note, and tuning information on a TFT LCD


Course clarification to confirm: The introductory course slides describe a requirement for four unique peripherals, but our Iris proposal was approved with SPI listed twice. We will ask the TA/lab coordinator whether the approved objectives are sufficient as written. We should not assume a fifth feature is required or change the approved objectives without discussing it.

The RP2350 will be the only computer controlling the device. A laptop may be used to prepare/copy MIDI files to the SD card; it is not required to control tuning at runtime.
How it works
1. Create a MIDI file containing the desired tuning notes and copy it onto the SD card.
2. Select the string/target note to tune (user interaction method still to be finalized).
3. Place the single servo adapter over the corresponding tuning peg.
4. Pluck the string.
5. The sensor produces an analog signal; the RP2350 samples it with the ADC.
6. Firmware estimates the string's fundamental frequency and compares it with the target.
7. If the pitch is too low (flat) or too high (sharp), the RP2350 commands a small servo adjustment through PWM.
8. Stop the motor, pluck/measure again, and repeat until within the chosen tolerance.
9. Display IN TUNE on the TFT LCD, remove the adapter, and move to the next string.
Important: The servo must not spin continuously without feedback. We will use short adjustment steps with limits and a reliable stop/neutral command to reduce the chance of overtightening or breaking a string.
System architecture:
                     microSD card (MIDI file)
                              |
                             SPI
                              |
Guitar string -> Piezo/audio sensor -> ADC -> RP2350 Proton
                                            |       |
                                           PWM     SPI
                                            |       |
                                    One movable   TFT LCD
                                        servo      display
                                            |
                                    Tuning peg adapter
                                            |
                                   Guitar tuning knob

External regulated supply -> Servo power
RP2350 and servo supply -> Common ground
Signal processing: Guitar signals contain harmonics, so detecting the highest-amplitude frequency alone may produce the wrong note. We will evaluate a suitable fundamental-frequency estimation method (e.g., autocorrelation or a validated alternative) and handle low-signal/noisy readings.

SPI: The SD reader and TFT may share SPI clock/data lines if supported by the hardware, using separate chip-select pins. Exact wiring is TBD.
Shopping list / bill of materials
Parts to order or fabricate
Item	Qty	Model / notes	Purpose	Status
Continuous-rotation micro servo	1	Adafruit FS90R, DigiKey 1528-1501-ND (original Iris proposal listed 6)	Physically rotates the peg	⬜ Check availability / order
Piezoelectric sensing element	2	Same Sky CEB-2704-L100 (1 active + 1 spare)	Produces analog vibration signal	⬜ Order
Regulated external servo supply	1	Voltage/current to match the final servo datasheet	Power servo without loading RP2350 output	⬜ Select / order
Piezo input-conditioning/protection parts	As needed	Bias network, resistors, capacitors, clamps / protection as designed	Keep ADC input within the RP2350's safe range	⬜ Design / source
Wiring and connectors	As needed	Breadboard wires, servo connector, shared ground connection	Interconnect hardware	⬜ Check lab kit
Servo-to-tuning-peg adapter	1	3D-printed or fabricated coupler	Grips tuning knob	⬜ Design / fabricate
Handheld servo housing / grip	1	3D-printed or fabricated	Prevents servo body from rotating	⬜ Design / fabricate
microSD card	1	Small FAT-formatted compatible card	Holds MIDI file	⬜ Check kit / obtain


Proposed purchasing links:  
- CEB-2704-L100 piezo — DigiKey search
- 1528-1501-ND servo — DigiKey search
Before ordering: Verify the selected servo's torque is sufficient to turn the actual guitar's tuning pegs. A small continuous-rotation servo may need a different gear ratio or a higher-torque replacement. The one-servo plan replaces, rather than adds to, the original quantity of six.
Parts expected from ECE 362 lab kit / team supplies
Item	Qty	Notes
RP2350 Proton development board	1	Main controller; use course hardware
TFT LCD	1	SPI display (verify kit module)
microSD reader	1	Verify kit module and voltage requirements
Breadboard	1	Prototype wiring
Debug Probe, cables, computer	As needed	Firmware development / debugging only
Guitar	1	Test instrument


Do not purchase duplicates until kit inventory has been checked. Prices, stock, and shipping are not tracked in this README.
Shopping checkoffs
- [ ] Check ECE kit inventory for display, SD interface, wires, and microSD card.
- [ ] Verify exact servo specification and test expected torque/rotation.
- [ ] Order one servo (not six).
- [ ] Order piezo element(s).
- [ ] Select safe servo power supply.
- [ ] Design and obtain ADC conditioning/protection parts.
- [ ] Measure tuning peg and servo shaft; make adapter.
- [ ] Confirm all ordered components have arrived and work.
Guitar target frequencies
Standard guitar tuning (A4 = 440 Hz):
Guitar string	Note	MIDI note number	Frequency
6 — lowest/thickest	E2	40	82.41 Hz
5	A2	45	110.00 Hz
4	D3	50	146.83 Hz
3	G3	55	196.00 Hz
2	B3	59	246.94 Hz
1 — highest/thinnest	E4	64	329.63 Hz


MIDI note numbers can be converted using:

frequency_hz = 440.0 * 2^((midi_note - 69) / 12)

A tuning tolerance expressed in cents is useful across strings:

cents_error = 1200 * log2(measured_frequency / target_frequency)
- Negative: flat (frequency too low).
- Positive: sharp (frequency too high).
- Near zero: in tune.
The exact acceptance tolerance (for example, ±5 cents) must be chosen and validated by the team.
Work plan and checkoffs
Check a box only after it has been tested. Unchecked means not yet verified, not necessarily that no work has started.
0. Planning and setup
- [x] Form team and submit Iris proposal.
- [x] Receive approval as Team 57 — Guitar Tunas.
- [x] Create GitHub repository.
- [x] Agree to use one movable servo, tuning one string at a time.
- [ ] Confirm final part models and circuit design.
- [ ] Confirm accepted peripheral objectives with TA.
- [ ] Decide how user selects string/target note.
- [ ] Record final RP2350 pin assignments and wiring diagram.
1. ADC — sensor and pitch detection
- [ ] Build piezo mounting and safe analog input circuit.
- [ ] Verify ADC input voltage range and protection.
- [ ] Read ADC samples using RP2350 firmware.
- [ ] Establish stable sample rate and sample buffer.
- [ ] Detect when the string is plucked / signal is strong enough.
- [ ] Estimate frequency from samples.
- [ ] Verify each of the six standard-string frequencies with a known reference tuner.
- [ ] Handle noisy audio, harmonics, and missing signal.
- [ ] Output stable measured frequency to the rest of the program.
2. PWM — servo and peg adapter
- [ ] Wire servo with external power and shared ground.
- [ ] Produce PWM signal from RP2350.
- [ ] Find reliable clockwise, counterclockwise, and stop commands.
- [ ] Test safe, short rotation bursts.
- [ ] Confirm servo can turn the guitar's tuning peg.
- [ ] Fabricate and test removable tuning-peg adapter.
- [ ] Verify tightening/loosening direction on the actual guitar.
- [ ] Add motor timeout, step limit, and emergency stop method.
- [ ] Demonstrate servo control independently of the sensor.
3. SPI — SD card and MIDI
- [ ] Wire and initialize SD card interface.
- [ ] Read a test file from the card.
- [ ] Prepare a valid MIDI file with tuning notes.
- [ ] Parse required MIDI header/track/note events (document supported format).
- [ ] Convert MIDI note numbers to frequencies.
- [ ] Map selected guitar string to target note.
- [ ] Handle missing card, invalid file, and unsupported MIDI content.
- [ ] Pass desired frequency to the tuning controller.
4. SPI — TFT LCD
- [ ] Wire and initialize TFT LCD.
- [ ] Display text and graphics.
- [ ] Show selected string/target note.
- [ ] Show measured frequency (Hz).
- [ ] Show FLAT / SHARP / IN TUNE status.
- [ ] Show a useful tuning meter or visual indicator.
- [ ] Show errors (no signal, missing card, motor stop, etc.).
- [ ] Test display updates alongside ADC sampling and SPI SD access.
5. Closed-loop tuning and integration
- [ ] Combine measured frequency and target-note modules.
- [ ] Implement frequency/error comparison.
- [ ] Set a tolerance for successful tuning.
- [ ] Decide small, safe servo correction duration based on error.
- [ ] Stop servo between adjustments and remeasure.
- [ ] Ignore unreliable frequency readings.
- [ ] Prevent uncontrolled rotation when readings are absent or wrong.
- [ ] Demonstrate complete one-string automatic tuning.
- [ ] Move same servo to another tuning peg and repeat.
- [ ] Integrate SD-driven targets and live TFT feedback.
- [ ] Test all six strings one at a time.
6. Documentation and demonstration
- [ ] Add circuit schematic / wiring diagram.
- [ ] Document the final bill of materials and pin map.
- [ ] Document how to build, flash, and run the firmware.
- [ ] Add photos of hardware and adapter.
- [ ] Record successful tuning demo.
- [ ] Prepare project explanation and testing results.
- [ ] Complete course checkoff / demonstration.
Integration and testing
Test	Success condition	Status
Sensor electrical test	ADC voltage remains safe under test conditions	⬜
Frequency accuracy	Detected string pitch matches a reliable reference tuner within team tolerance	⬜
Servo direction	Motor can tighten, loosen, and stop reliably	⬜
Mechanical attachment	Adapter fits peg and can be moved without damage	⬜
SD/MIDI read	Firmware obtains intended target note(s) from file	⬜
Display	Correct note, frequency, and status appear	⬜
Automatic correction	Out-of-tune string moves toward target, not away	⬜
Stop condition	Servo stops when target tolerance is reached	⬜
Safety handling	Servo stops on invalid measurement, timeout, or user abort	⬜
Full demo	Same servo tunes more than one string sequentially	⬜


Initial milestone: Make the RP2350 measure the pitch of a string correctly; independently test whether the servo can physically turn a peg. These two tests reduce the biggest project risks.
Team responsibilities
Workstreams below are proposed, not final member assignments.
Workstream	Main tasks	Owner
Servo / mechanical	PWM, motor testing, coupling, safety limits	TBD
Audio / frequency	Sensor conditioning, ADC sampling, pitch detection	TBD
SD / MIDI	SPI file access, MIDI parsing, target-note selection	TBD
Display / integration	TFT interface, graphics, end-to-end testing	TBD


Everyone should help with code reviews, integration, debugging, and final demonstrations.
Design decisions and open questions
Confirmed
- Main controller: RP2350 Proton.
- Firmware-based sensing and motor control.
- Desired tuning comes from MIDI file on SD card.
- TFT displays detected pitch/tuning feedback.
- One servo is moved manually between knobs.
- Only one string is tuned at a time.
To validate
- Piezo vs microphone: The Iris description uses “microphone,” while the related-work description specifies a piezo vibration sensor. The listed CEB-2704-L100 is a piezo element; final sensor implementation must be tested.
- Servo torque: Confirm the FS90R can turn the particular guitar's tuning pegs. Replace with a stronger servo if needed.
- Frequency algorithm: Choose a method that handles the guitar's harmonics and noise.
- MIDI file format: Decide exact supported subset (e.g., single-track MIDI with specific note events) to keep parsing manageable.
- String selection: Decide whether selection uses a physical button or existing input hardware.
- Four unique peripherals: Clarify the approved SPI/SPI objectives with the TA.
- Safety: Set a maximum number/duration of servo adjustments, a no-signal stop rule, and a way to abort immediately.
Project deliverables
- Working RP2350-based hardware prototype
- Functional ADC pitch sensing, PWM servo control, SPI SD/MIDI reading, and SPI TFT display
- One removable servo-to-peg adapter
- Safe closed-loop tuning demonstration
- Source code and wiring documentation
- Testing results and final course demo/checkoff
Course timeline: The course introduction identifies Nov 30–Dec 4, 2026 as the project checkoff/demo period. A PCB is optional extra credit, separate from the main prototype; the uploaded PCB FAQ gives a Nov 6, 2026 at 5 PM design submission deadline. If we pursue PCB extra credit, check the latest instructor instructions before submission.
References
- ECE 362 Guitar Tunas GitHub repository
- Team 57 approved project proposal (Iris, Fall 2026)
- ECE 362 course project introduction (Fall 2026)
- Elektor — DIY Guitar Tuner
- SparkFun — Piezo Vibration Sensor Hookup Guide
- Component datasheets should be added once the final sensor, servo, and power supply are confirmed.

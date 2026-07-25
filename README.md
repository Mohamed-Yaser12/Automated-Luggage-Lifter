# Automated Electro-Pneumatic Luggage Lifter


## 📌 Project Overview
The **Luggage Lifter** is a pneumatic-powered automated system designed to transfer luggage from a lower receiving platform to an upper storage level in a continuous sequence. Built with a stable frame structure and driven by three pneumatic cylinders, the system is designed to reduce manual handling effort in transport operations such as airports or storage facilities.

<img width="1280" height="720" alt="1" src="https://github.com/user-attachments/assets/905c7ab8-af36-46bc-b958-a2146ed53236" />
<img width="1280" height="960" alt="4" src="https://github.com/user-attachments/assets/ad07b445-d710-4884-be86-66fbe8349681" />
<img width="1280" height="720" alt="2" src="https://github.com/user-attachments/assets/2502ba0f-83c5-45d5-8ad8-b6895922577d" />
<img width="1280" height="960" alt="3" src="https://github.com/user-attachments/assets/b62c93dd-9278-430e-baf2-5cfbf326013f" />


---

## ⚙️ System Architecture & Working Principle

### **Mechanism Description**
1. **Frame Structure:** Ground-mounted frame supporting all mechanical and pneumatic components.
2. **Receiving Platform (Level 2):** Initial placement zone for luggage.
3. **Cylinder 1 (Horizontal Transfer):** Pushes luggage from the receiving platform onto the lifting platform.
4. **Cylinder 2 (Vertical Lift):** Elevates the platform from Level 2 to Level 3.
5. **Cylinder 3 (Horizontal Push):** Pushes luggage from the elevated platform onto a slider, where it returns via gravity/pass-through to complete the cycle.

### **Pneumatic Actuators Summary**
* **Cylinder A / Cylinder 1:** MAL20×125 (Horizontal Transfer)
* **Cylinder B / Cylinder 2:** MAL20×100 (Vertical Lifting)
* **Cylinder C / Cylinder 3:** MAL20×125 (Horizontal Exit Push)

---

## 📐 SolidWorks Design Pictures

### **3D Assembly Schematic**

| View 1 | View 2 |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/5ca76464-0ef8-4fb8-b6d2-1fbf1d3b42a9" width="100%" alt="SolidWorks View 1" /> | <img src="https://github.com/user-attachments/assets/e88f5884-7660-438b-998a-006050e22186" width="100%" alt="SolidWorks View 2" /> |
| **View 3** | **View 4** |
| <img src="https://github.com/user-attachments/assets/42adb996-b688-4c66-b03b-4e0b238b3cad" width="100%" alt="SolidWorks View 3" /> | <img src="https://github.com/user-attachments/assets/34b471c6-be29-4a26-af88-0c6843c9f7ab" width="100%" alt="SolidWorks View 4" /> |


---

## ⚡ Electro-Pneumatic Circuit & Control Logic

### **Sequence Execution**
$$A+ \rightarrow (A- \text{ and } B+) \rightarrow \text{Delay} \rightarrow C+ \rightarrow (C- \text{ and } B-)$$

#### **Sequence Step-by-Step Explanation:**
* **Step 1 ($A+$):** When the `START` push button is engaged and proximity sensor `S1` detects luggage on Level 2, **Cylinder A** extends to transfer the item onto the vertical lifting platform.
* **Step 2 ($A-$ & $B+$):** Reaching reed switch `R1` triggers **Cylinder A** to retract while simultaneously activating **Cylinder B** to lift the platform vertically from Level 2 to Level 3.
* **Step 3 ($\text{Delay}$):** Once **Cylinder B** reaches the upper level (hitting reed switch `R2`), a timer introduces a short delay to stabilize the platform before moving to the next phase.
* **Step 4 ($C+$):** **Cylinder C** extends horizontally to push the luggage off the elevated platform and onto the slider mechanism.
* **Step 5 ($C-$ & $B-$):** Reaching reed switch `R3` signals **Cylinder C** to retract and **Cylinder B** to descend back to Level 2, completing the cycle and resetting the system for the next load.

---

### **Pneumatic Step Diagram**
<p align="center">
  <img src="https://github.com/user-attachments/assets/a0ba3663-3886-4b67-bdee-bf8242cda34b" width="80%" alt="Pneumatic Step Diagram" />
</p>

#### **Step Diagram Overview:**
* **Position vs. Time Graph:** The step-displacement diagram visually displays the state (Extended vs. Retracted) of Cylinders A, B, and C across each phase of operation.
* **Interlock Control:** It illustrates how feedback elements (proximity sensor `S1` and reed switches `R1`, `R2`, `R3`) trigger sequential transitions, preventing mechanical clashes during movement.

---

### **FluidSim Electro-Pneumatic Circuit Diagram**
<p align="center">
  <img src="https://github.com/user-attachments/assets/3dbb739a-d578-471a-8d9f-bf716790d95c" width="80%" alt="FluidSim Circuit Diagram" />
</p>

#### **Circuit Architecture Details:**
* **Pneumatic Power Section:** Features three double-acting pneumatic cylinders controlled by $5/2$-way double solenoid directional control valves (`SOL-1`, `SOL-2`, `SOL-3`) connected to a regulated pressure supply.
* **Electrical Control Section:** Uses classic relay logic powered by a $24\text{V DC}$ supply. Relay contacts (`K0`, `C1`, `C2`, `C3`) act as latching circuits to store memory states during step transitions.
* **Safety Integration:** Includes a master control relay (`K0`) wired to latching `START` / `STOP` buttons for emergency halt capability.

---

## 💻 Control Implementation Code

Below is the structured PLC/Control Logic pseudocode representing the Relay Ladder Logic implemented for the electro-pneumatic sequence:

```pascal
// ========================================================
// Electro-Pneumatic Sequence Control Logic
// Sequence: A+ -> (A- AND B+) -> Delay -> C+ -> (C- AND B-)
// Actuators: SOL_1 (Cylinder A), SOL_2 (Cylinder B), SOL_3 (Cylinder C)
// Sensors: S1 (Proximity), R1 (Reed Switch 1), R2 (Reed Switch 2), R3 (Reed Switch 3)
// ========================================================

PROGRAM Luggage_Lifter_Control
VAR
    Start_PB      : BOOL; (* Start Push Button *)
    Stop_PB       : BOOL; (* Stop Push Button *)
    S1_Proximity  : BOOL; (* Object present on receiving platform *)
    R1_Reed       : BOOL; (* Cylinder A Extended Limit *)
    R2_Reed       : BOOL; (* Cylinder B Extended Limit *)
    R3_Reed       : BOOL; (* Cylinder C Extended Limit *)

    K0_MasterRelay: BOOL;
    C1_Relay      : BOOL;
    C2_Relay      : BOOL;
    C3_Relay      : BOOL;

    SOL_1_Extend  : BOOL;
    SOL_1_Retract : BOOL;
    SOL_2_Extend  : BOOL;
    SOL_2_Retract : BOOL;
    SOL_3_Extend  : BOOL;
    SOL_3_Retract : BOOL;

    Step_Timer    : TON;  (* On-Delay Timer *)
END_VAR

// Master Control Relay (Latch / Unlatch)
K0_MasterRelay := (Start_PB OR K0_MasterRelay) AND NOT Stop_PB;

IF K0_MasterRelay THEN

    // Step 1: Cylinder A Extension (Push to lifter)
    IF S1_Proximity AND NOT C1_Relay THEN
        SOL_1_Extend := TRUE;
        SOL_1_Retract := FALSE;
    END_IF;

    // Step 2: Cylinder A Retracts AND Cylinder B Extends (Vertical Lift)
    IF R1_Reed THEN
        C1_Relay := TRUE;
        SOL_1_Extend := FALSE;
        SOL_1_Retract := TRUE;
        
        SOL_2_Extend := TRUE;
        SOL_2_Retract := FALSE;
    END_IF;

    // Step 3: Delay at top level before Cylinder C triggers
    IF R2_Reed THEN
        C2_Relay := TRUE;
        Step_Timer(IN := TRUE, PT := T#2S); // 2 Second Delay
    END_IF;

    // Step 4: Cylinder C Extension (Push to slider)
    IF Step_Timer.Q THEN
        SOL_3_Extend := TRUE;
        SOL_3_Retract := FALSE;
    END_IF;

    // Step 5: Cylinder C Retracts AND Cylinder B Retracts (System Reset)
    IF R3_Reed THEN
        C3_Relay := TRUE;
        SOL_3_Extend := FALSE;
        SOL_3_Retract := TRUE;
        
        SOL_2_Extend := FALSE;
        SOL_2_Retract := TRUE;
        
        // Reset Timer
        Step_Timer(IN := FALSE);
    END_IF;

END_IF;

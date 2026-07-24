# Automated Electro-Pneumatic Luggage Lifter


## 📌 Project Overview
The **Luggage Lifter** is a pneumatic-powered automated system designed to transfer luggage from a lower receiving platform to an upper storage level in a continuous sequence[cite: 1]. Built with a stable frame structure and driven by three pneumatic cylinders, the system is designed to reduce manual handling effort in transport operations such as airports or storage facilities[cite: 1].

---

## ⚙️ System Architecture & Working Principle

### **Mechanism Description**
1. **Frame Structure:** Ground-mounted frame supporting all mechanical and pneumatic components[cite: 1].
2. **Receiving Platform (Level 2):** Initial placement zone for luggage[cite: 1].
3. **Cylinder 1 (Horizontal Transfer):** Pushes luggage from the receiving platform onto the lifting platform[cite: 1].
4. **Cylinder 2 (Vertical Lift):** Elevates the platform from Level 2 to Level 3[cite: 1].
5. **Cylinder 3 (Horizontal Push):** Pushes luggage from the elevated platform onto a slider, where it returns via gravity/pass-through to complete the cycle[cite: 1].

### **Pneumatic Actuators Summary**
* **Cylinder A / Cylinder 1:** MAL20×125 (Horizontal Transfer)[cite: 1]
* **Cylinder B / Cylinder 2:** MAL20×100 (Vertical Lifting)[cite: 1]
* **Cylinder C / Cylinder 3:** MAL20×125 (Horizontal Exit Push)[cite: 1]

---

## 📐 SolidWorks Design Pictures

### **3D Assembly Schematic**
<img width="740" height="649" alt="Screenshot 2025-10-23 003105" src="https://github.com/user-attachments/assets/5ca76464-0ef8-4fb8-b6d2-1fbf1d3b42a9" />
<img width="713" height="619" alt="Screenshot 2025-10-23 003036" src="https://github.com/user-attachments/assets/e88f5884-7660-438b-998a-006050e22186" />
<img width="877" height="637" alt="Screenshot 2025-10-23 003020" src="https://github.com/user-attachments/assets/42adb996-b688-4c66-b03b-4e0b238b3cad" />
<img width="925" height="600" alt="Screenshot 2025-10-23 002943" src="https://github.com/user-attachments/assets/34b471c6-be29-4a26-af88-0c6843c9f7ab" />

### **SolidWorks Design Model Views**

| Isometric View | Front View | Top View |
| :---: | :---: | :---: |
| <!-- Place image at: assets/images/solidworks_iso.png --> ![SolidWorks Isometric](assets/images/solidworks_iso.png) | <!-- Place image at: assets/images/solidworks_front.png --> ![SolidWorks Front](assets/images/solidworks_front.png) | <!-- Place image at: assets/images/solidworks_top.png --> ![SolidWorks Top](assets/images/solidworks_top.png) |

---

## ⚡ Electro-Pneumatic Circuit & Control Logic

### **Sequence Execution**
$$A+ \rightarrow (A- \text{ and } B+) \rightarrow \text{Delay} \rightarrow C+ \rightarrow (C- \text{ and } B-)$$

### **Pneumatic Step Diagram**
<img width="764" height="595" alt="aa" src="https://github.com/user-attachments/assets/a0ba3663-3886-4b67-bdee-bf8242cda34b" />


### **FluidSim Electro-Pneumatic Circuit Diagram**
<!-- Place image at: assets/images/fluidsim_circuit.png -->
<img width="855" height="566" alt="ccs" src="https://github.com/user-attachments/assets/3dbb739a-d578-471a-8d9f-bf716790d95c" />

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

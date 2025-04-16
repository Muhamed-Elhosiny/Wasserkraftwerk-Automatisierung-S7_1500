# Wasserkraftwerk-Automatisierung-S7_1500

# Wasserkraftwerk Projekt (TIA Portal)

## Projektübersicht

![image](https://github.com/user-attachments/assets/d1d627d8-9ff7-4903-8603-8c70b36f63f2)

Dieses Projekt simuliert ein einfaches **Wasserkraftwerk**, das mit einer SPS-Steuerung betrieben wird.\
Das Ziel ist es, ein Steuerungssystem zu entwickeln, das:

- **Analoge und digitale Signale verarbeitet**
- **Die Generatorleistung reguliert**
- **Eine HMI-Schnittstelle bietet** zur Interaktion mit dem System
- **Sicherheitsmaßnahmen** wie Alarm- und Not-Aus-Mechanismen integriert

---

## Funktionsweise

- Der **Fluss** ändert regelmäßig seine Geschwindigkeit.
- Ein **Baffle (Drosselklappe)** steuert den **Wasserfluss** mit einem **Servomotor (4-20 mA Signal)**.
- Das Wasser treibt einen **Generator** an, der Strom produziert.
- Eine **Ölpumpe (VFD-gesteuert)** kühlt die **Rotorlager**.
- Bei ausreichend hoher Leistung wird ein **Interlock (Schutzschalter) aktiviert**, um Strom in das Netz einzuspeisen.
- Im **Stillstand** bleibt das **Baffle offen** und die **Bremse aktiviert**, um das System zu sichern.

---

## Eingangs- und Ausgangssignale (IO)

### **Baffle (Drosselklappe)**

- **Signale:**
  - `Baffle_En` (**Digitalausgang** – Aktivierung)
  - `Baffle_Out` (**Analogausgang** – Position)
- **Skalierung:** 0–27648 → **HMI-Anzeige:** 0–100%
- **Polarisierung:** Umgekehrt (0 = 100% offen, 27648 = geschlossen)

### **Rotorgeschwindigkeit**

- **Signal:** `RPM_In` (**Analogeingang**)
- **Skalierung:** 0–27648 → **HMI-Anzeige:** 0–200 U/min
- **Polarisierung:** Gerade (Höherer Wert = Höhere Drehzahl)

### **Ölpumpe (VFD)**

- **Signale:**
  - `OilPumpVFDen_Out` (**Digitalausgang** – Aktivierung)
  - `OilPumpVFDsp_Out` (**Analogausgang** – Geschwindigkeit)
- **Skalierung:** 0–27648 → **HMI-Anzeige:** 0–100%
- **Polarisierung:** Gerade (Höherer Wert = Höhere Pumpenleistung)

### **Ölfluss**

- **Signal:** `OilFlow_In` (**Analogeingang**)
- **Skalierung:** 0–27648 → **HMI-Anzeige:** 0–30 GPM
- **Polarisierung:** Gerade (Höherer Wert = Höherer Durchfluss)

### **Öltemperatur**

- **Signal:** `OilTemp_In` (**Analogeingang**)
- **Skalierung:** 0–27648 → **HMI-Anzeige:** 0–500°C
- **Polarisierung:** Gerade (Höherer Wert = Höhere Temperatur)

### **Leistungsüberwachung & Schutzfunktionen**

| **Komponente**           | **Signal**             | **Typ** | **Funktion**                              |
| ------------------------ | ---------------------- | ------- | ----------------------------------------- |
| **Leistungsüberwachung** | `AC_Power_In`          | AI      | Leistungsmessung (0–700 kW)               |
| **Netzschütz**           | `StationInterlock_Out` | DQ      | Interlock aktivieren                      |
| **Alarmhupe**            | `AlarmHorn_Out`        | DQ      | Alarm auslösen                            |
| **Rotorbremse**          | `Brake_Out`            | DQ      | Bremse aktivieren                         |
| **Not-Aus**              | `EStop_In`             | DI      | Not-Aus (umgekehrt: geschlossen = normal) |

---

## ⚠ Alarmbedingungen

| **Alarm**             | **Grenzwert**  | **Aktionen**                                           |
| --------------------- | -------------- | ------------------------------------------------------ |
| **Überstrom**         | >550 kW        | Interlock trennen, Shutdown einleiten                  |
| **Hohe Öltemperatur** | >400°C         | Interlock trennen, Shutdown einleiten                  |
| **Rotor-Überspeed**   | >180 U/min     | Interlock trennen, Shutdown einleiten                  |
| **Öl-Niedrigfluss**   | <5 GPM für 10s | Interlock trennen, VFD deaktivieren, Bremse aktivieren |

---

## Betriebsmodi & Ablaufsteuerung

### 1️_**Warmup**

- Rotor auf **20–50 U/min**
- Ölpumpe auf **2–10 GPM**
- Nach **10s bei 150°C** → **Weiter zu Stabilisierung**

### 2️_**Stabilisierung**

- Generator fährt hoch
- Zielwert **kW +/- 5kW für 10s** erreicht → **Weiter zu Erzeugung**

### 3️_**Erzeugung**

- System läuft im stabilen Modus
- **Interlock geschlossen** → Stromerzeugung aktiv

### 4️_**Cooldown (Abkühlen)**

- Nach **Stop-Befehl**: Rotor auf **20–50 U/min**
- Hoher Ölfluss, bis **150°C unterschritten**

### 5️_**Leerlauf (Idle)**

- Interlock deaktiviert, **Baffle offen**, Bremse aktiv

### 6️_ **Fehlermodus (Fault)**

- Bei einem Alarm wird die **Sequenz gestoppt**, und der Fehler wird bearbeitet.

---

## HMI (Bedienoberfläche)

| **HMI-Bildschirm**     | **Funktion**                                       |
| ---------------------- | -------------------------------------------------- |
| **Systemstatus**       | Anzeige der Betriebsmodi, Alarme & Hauptwerte      |
| **Prozessübersicht**   | Grafische Darstellung mit Echtzeitwerten           |
| **Alarmverwaltung**    | Alarmhistorie, Reset- & Stummschaltoptionen        |
| **IO / HOA-Steuerung** | Manuelle Bedienung und Setpoints für analoge Werte |


![image](https://github.com/user-attachments/assets/307e96fa-619e-4054-8e07-770b6a624cc5)

![image](https://github.com/user-attachments/assets/dcd15870-4600-4b0e-9e62-fd74444e803a)

![image](https://github.com/user-attachments/assets/815a32e4-dc5a-48b1-ba39-3d7c1f3f7a0b)

![image](https://github.com/user-attachments/assets/bf012702-e49d-446b-b8e3-9c58ce07a507)

![image](https://github.com/user-attachments/assets/d01d9ff4-a9ca-4a7b-8987-e0d9e2a9fe18) 


# Eingänge /Ausgänge  (E/A)

## Notiz
Die erste Implementierung erfolgt in **Ladder Diagram (LAD)**, da diese grafische Programmiersprache weit verbreitet ist und sich besonders für **logische Steuerungen und Ein-/Ausgangsoperationen** eignet.  
In diesem Abschnitt werden die **Hauptfunktionen des Wasserkraftwerks** mithilfe von **Schütz- und Relaislogik** umgesetzt.

## Main [OB1]
Falls jemand die **Ladder-Taste** drückt oder die Zahl außerhalb des Bereichs von **1 bis 4** liegt, dann **setze eine Eins**.


![image](https://github.com/user-attachments/assets/6cfaddd5-f1c4-477e-80fa-84be9e56df82)

## Notiz  
Nun wird der **Programmbaustein (IO_LAD_DB_1)** jedes Mal aufgerufen, wenn die **Programmiersprache gleich 1** ist.  
In unserem **Main-Programm** rufen wir verschiedene **Datenbausteine (DBs)** auf, wie **IO, ALARMS und HOAs**, für jede Sprache.

![image](https://github.com/user-attachments/assets/ccbc6bff-221f-4645-9267-b2744a3b3e34)

## Notiz  
Nun wird der **Programmbaustein (IO_FBD_DB)** jedes Mal aufgerufen, wenn die **Programmiersprache gleich 2** ist.

![image](https://github.com/user-attachments/assets/dfe31d1b-6bc9-47ba-8486-0b3ba9089846)

## Notiz  
Nun wird der **Programmbaustein (IO_SCL_DB)** jedes Mal aufgerufen, wenn die **Programmiersprache gleich 3** ist.

![image](https://github.com/user-attachments/assets/86f4b62e-b04d-41bb-8e1f-1ca9f16ba819)

## Notiz  
Nun wird der **Programmbaustein (IO_STL_DB)** jedes Mal aufgerufen, wenn die **Programmiersprache gleich 4** ist.

![image](https://github.com/user-attachments/assets/4c0824b4-825e-43c0-ad45-e226828caa24)


## IO_LAD [FB2]  (Eingangs- und Ausgangssignale)

Hier werden **digitale und analoge Signale** entsprechend der Steuerungslogik verarbeitet und weitergeleitet. 

## Netzwerk 1: Digitale Ausgänge  

![image](https://github.com/user-attachments/assets/893d624a-9239-4f4a-b7e9-b357307a4cfc)

![image](https://github.com/user-attachments/assets/c025f0b2-07ad-40ce-b700-5c01109949cf)

## Netzwerk 2: Analoge Ausgänge  
In diesem Netzwerk werden die **analogen Ausgänge** für die Steuerung der Prozesse gesetzt.  
Hierzu gehören:  

- **Oilpump_VFD** → Steuert die Frequenz des **Ölpumpen-VFDs** zur Anpassung des Ölflusses.  
- **Baffle_Out** → Regelt die **Drosselklappenposition** zur Steuerung des Wasserflusses.  

Die Werte werden über **Analogsignale (0-27648)** an die Aktoren ausgegeben.

![image](https://github.com/user-attachments/assets/e2f00f63-03a9-41e0-887c-9922d5954c1b)

## Netzwerk 3: Analoge Eingänge  
Dieses Netzwerk verarbeitet die **analogen Eingangssignale**, um Messwerte aus der Anlage zu erfassen.  

- **Temp_Sensor** → Liest die **Öltemperatur** im Bereich **0 bis 500°C** ein.  
- Die Temperatur kann als **Real-Wert** gespeichert werden, jedoch wird sie hier zur **Vereinfachung als Integer** behandelt.  

Die Skalierung erfolgt auf **0-27648** für eine 16-Bit-Auflösung.

![image](https://github.com/user-attachments/assets/5079f831-d847-4cf9-af41-11fa0dc18ea6)

## Vollständige Programmdokumentation (PDF)  

Um die GitHub-Seite übersichtlich zu halten, enthält diese PDF-Datei die vollständige Programmdokumentation:  

**Komplette Steuerungssequenz in vier SPS-Sprachen:**  
   - Ladder Diagram (LAD)  
   - Statement List (STL)  
   - Function Block Diagram (FBD)  
   - Structured Control Language (SCL)  

**PID-Regelung**  
   - Implementierung und Funktionsweise der PID-Steuerung im System  

**WINCC HMI-Visualisierung**  
   - Beschreibung der HMI-Oberfläche mit Visualisierung der Prozessdaten  

Diese PDF ermöglicht einen **kompletten Überblick über das Steuerungssystem**, ohne die GitHub-Seite zu überladen.  

## Vollständige Programmdokumentation (PDF) :

[Wasserkraftwerk-Automatisierung-S7_1500.pdf](https://github.com/user-attachments/files/19148915/Wasserkraftwerk-Automatisierung-S7_1500.pdf)








 

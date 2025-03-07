# Wasserkraftwerk-Automatisierung-S7_1500

# 📂 Wasserkraftwerk Projekt (TIA Portal)

## 🔹 Projektübersicht

![image](https://github.com/user-attachments/assets/d1d627d8-9ff7-4903-8603-8c70b36f63f2)

Dieses Projekt simuliert ein einfaches **Wasserkraftwerk**, das mit einer SPS-Steuerung betrieben wird.\
Das Ziel ist es, ein Steuerungssystem zu entwickeln, das:

- **Analoge und digitale Signale verarbeitet**
- **Die Generatorleistung reguliert**
- **Eine HMI-Schnittstelle bietet** zur Interaktion mit dem System
- **Sicherheitsmaßnahmen** wie Alarm- und Not-Aus-Mechanismen integriert

---

## 🔧 Funktionsweise

- Der **Fluss** ändert regelmäßig seine Geschwindigkeit.
- Ein **Baffle (Drosselklappe)** steuert den **Wasserfluss** mit einem **Servomotor (4-20 mA Signal)**.
- Das Wasser treibt einen **Generator** an, der Strom produziert.
- Eine **Ölpumpe (VFD-gesteuert)** kühlt die **Rotorlager**.
- Bei ausreichend hoher Leistung wird ein **Interlock (Schutzschalter) aktiviert**, um Strom in das Netz einzuspeisen.
- Im **Stillstand** bleibt das **Baffle offen** und die **Bremse aktiviert**, um das System zu sichern.

---

## 📌 Eingangs- und Ausgangssignale (IO)

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

## 🔄 Betriebsmodi & Ablaufsteuerung

### 1️⃣ **Warmup**

- Rotor auf **20–50 U/min**
- Ölpumpe auf **2–10 GPM**
- Nach **10s bei 150°C** → **Weiter zu Stabilisierung**

### 2️⃣ **Stabilisierung**

- Generator fährt hoch
- Zielwert **kW +/- 5kW für 10s** erreicht → **Weiter zu Erzeugung**

### 3️⃣ **Erzeugung**

- System läuft im stabilen Modus
- **Interlock geschlossen** → Stromerzeugung aktiv

### 4️⃣ **Cooldown (Abkühlen)**

- Nach **Stop-Befehl**: Rotor auf **20–50 U/min**
- Hoher Ölfluss, bis **150°C unterschritten**

### 5️⃣ **Leerlauf (Idle)**

- Interlock deaktiviert, **Baffle offen**, Bremse aktiv

### 6️⃣ **Fehlermodus (Fault)**

- Bei einem Alarm wird die **Sequenz gestoppt**, und der Fehler wird bearbeitet.

---

## 🎛 HMI (Bedienoberfläche)

| **HMI-Bildschirm**     | **Funktion**                                       |
| ---------------------- | -------------------------------------------------- |
| **Systemstatus**       | Anzeige der Betriebsmodi, Alarme & Hauptwerte      |
| **Prozessübersicht**   | Grafische Darstellung mit Echtzeitwerten           |
| **Alarmverwaltung**    | Alarmhistorie, Reset- & Stummschaltoptionen        |
| **IO / HOA-Steuerung** | Manuelle Bedienung und Setpoints für analoge Werte |


![image](https://github.com/user-attachments/assets/307e96fa-619e-4054-8e07-770b6a624cc5)

![image](https://github.com/user-attachments/assets/dcd15870-4600-4b0e-9e62-fd74444e803a)

![image](https://github.com/user-attachments/assets/815a32e4-dc5a-48b1-ba39-3d7c1f3f7a0b)



 

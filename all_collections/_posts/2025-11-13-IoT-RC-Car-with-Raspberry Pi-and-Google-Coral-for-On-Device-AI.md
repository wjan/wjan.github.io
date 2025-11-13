# Motivation

After the first proof of concept with the ESP32Cam Wi-Fi RC car, I realized the biggest bottlenecks were camera quality and computing power.  
The ESP32Cam is great for basic remote control and simple streaming, but its limited sensor, poor image quality, and weak processing capabilities made any real-time object detection nearly impossible.

This naturally led me to the next step — giving the robot a proper **“brain”** and **“eyes.”**  
I wanted a system that could handle local object detection at decent frame rates, communicate with the existing ESP32-based control system, and run autonomously without relying on my laptop.

Enter the **Raspberry Pi Zero 2 W** paired with a **Google Coral USB Accelerator**.  

This compact but powerful combo provided the computational muscle I needed for real-time inference — all packed into a surprisingly stable, improvised enclosure on top of the same RC car base.

---

## Hardware Setup

- **Raspberry Pi Zero 2 W** — main controller and inference engine  
- **Raspberry Pi Camera Module** — providing higher quality, low-latency video feed  
- **Google Coral USB Accelerator** — performing on-device inference using TensorFlow Lite models optimized for the Edge TPU  
- **ESP32Cam** — still responsible for motor control, Wi-Fi communication, and receiving commands over HTTP  
- **Power Supply** — single 16340 Li-ion cell (700 mAh) powering both Pi + Coral and ESP32Cam  

Despite sounding like an underpowered setup, the single small battery could drive the entire system — motors, compute modules, and Wi-Fi — for around **30 minutes** of runtime.  
A more rigorous battery test still needs to be done, but the initial results were promising.

📸 *Photos of the setup would go here.*

---

## Software & Control Architecture

The architecture remained similar in spirit to the first prototype but was much more capable:

1. The **ESP32Cam** runs a minimized and optimized version of the **Pilot Hobbies** firmware, now with smoother motion and slower speeds to reduce camera jitter.  
2. The **Raspberry Pi** captures frames from the Pi Camera and runs inference locally on the **Coral TPU** using the **SSD MobileNet V2 EdgeTPU** model.  
3. The Pi sends control commands via HTTP over Wi-Fi back to the ESP32Cam to steer the RC car.  
4. All logic — including target detection, tracking, and decision-making — runs on the Pi.

With the Coral’s acceleration, inference throughput jumped to an impressive **~25 FPS**, making the system finally capable of near real-time tracking.

---

## Person-Following Algorithm

This second-generation algorithm achieved surprisingly good results — the car could **follow the largest detected person** in its field of view with smooth and stable behavior.

### Logic Overview

- If **no person** is detected, the car spins in place (direction based on last known target position) until a person appears.  
- When a person **enters the frame**, the car adjusts its heading to keep the person centered horizontally.  
- Once the person is roughly centered, the car **drives forward** to approach.  
- When the person’s bounding box covers **>70%** of the frame, the car **stops**.  
- If the box grows **>90%**, the car **backs up** to maintain distance.  
- The system has **simple short-term memory**: if detection is lost briefly (for up to 5 frames), it assumes the target is still nearby and doesn’t immediately switch to “spinning” mode.  
- The algorithm currently doesn’t handle obstacle avoidance — that’s left for future iterations.

### State Management

```python
state = {
  "mode": "spinning",
  "last_direction": "left",
  "person_lost_count": 0
}
```

This minimalist state machine dramatically improved reliability.
I initially resisted adding state to keep things simple, but it turned out essential for maintaining context between frames and avoiding erratic movements.

### GUI vs CLI Modes

I built two modes for running the software:

- GUI Mode — includes a live preview and bounding box overlay; perfect for debugging and fine-tuning parameters, but more demanding on the network and battery.
- CLI Mode — lightweight, headless mode that runs automatically on boot; ideal for extended tests and eventual “out-of-the-box” behavior.

Car speed, camera parameters, and model options are all configurable via command-line arguments, allowing quick tuning for performance versus stability.

One note: as the battery voltage drops, the motors naturally slow down, which actually helps stabilization and inference quality.
Increasing the speed parameter compensates for this, keeping motion consistent during longer runs.

## Challenges Encountered
- Camera Jitter	- Improved via lower motor speed, mechanical stabilization (taping modules together), and ESP32 firmware tweaks.
- Wi-Fi Stability	- Generally solid, but long-range operation from the router can cause noticeable lag.
- Google Coral Setup - Slightly painful due to deprecated official support; reused experience from the first article to get it running.
- Low Battery Behavior - As voltage drops, the Pi can reboot or corrupt the SD card — must monitor closely or use safe-shutdown scripts.
- Parameter Tuning - Each hardware setup behaves differently; careful calibration was essential for balanced movement.
- State Management - Eventually added simple state handling to stabilize behavior and prevent over-reacting to transient detection losses.
- Battery Swapping - External chargers are a must for smooth testing — you can charge one cell while running on another.

## Future Improvements Ideas
- Enhanced State Machine — incorporate prediction and smarter decision-making.
- Memory & Backtracking — remember last seen locations or paths.
- Upgraded Hardware — larger batteries, more stable chassis, improved camera angles (POV-style).
- Sensors — ultrasonic or LiDAR modules for obstacle avoidance (though staying vision-only would be an interesting challenge).
- Full Autonomy — move toward onboard-only control without external Wi-Fi or human intervention.

## Summary

This second version feels like a genuine success — not just a technical one but also creatively fulfilling.
The RC car now reacts to its environment in real time, follows people intelligently, and runs a full AI inference pipeline onboard, all powered by a single small battery.

From a shaky start with a laggy, low-quality ESP32Cam experiment, this project has evolved into a miniature autonomous robot — proof that with a bit of tinkering, patience, and persistence, DIY robotics can yield surprisingly powerful results.
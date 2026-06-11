# Jesse You

Staff Software Engineer with 8+ years specializing in native C++ cross-platform development (Android/iOS), real-time streaming systems, and edge AI pipeline integration. Reduced live-streaming latency from 4–5s to 2–3s, improved platform crash-free rate from 96% to 99%, and scaled the system to 140K+ daily active users. Deep expertise in audio/video synchronization, GPU renderingpipelines, SDK architecture, and build system engineering.

## Staff Software Engineer - Lang Live Inc. Apr 2022 - Present

### Kotlin/Java/Objective-C/C++/Bazel/MediaPipe/Live2D/Box2D/OpenGL ES

**Real-Time Streaming Platform**

- Architected LangStreamer v3 on a C++ cross-platform codebase (Android & iOS) using Google MediaPipe Framework, enabling multithreaded
processing for real-time computer vision pipelines and simplifying edge-AI model deployment.
- Reduced end-to-end streaming latency from 4–5s to 2–3s via RTMP protocol optimization and adaptive bitrate control, improving viewer experience
across varying network conditions.
- Implemented audio/video synchronization monitoring with A/V diff threshold warnings, MIC_ROUTE_CHANGED event detection, and SEI bitrate
embedding for real-time diagnostics.
- Scaled platform to 140K+ daily active users while maintaining 99% crash-free rate (up from 96%).

**SDK Integration & Platform Engineering**

- Integrated Agora and SenseMe SDKs into a cohesive real-time pipeline; diagnosed and eliminated GL context pollution (player-side black screens) from libraries sharing a single GL context (client-side Spine, Tencent PAG) by isolating child GL contexts and enforcing GL state save/restore.
- Diagnosed and fixed a production-blocking iOS streaming timestamp bug on high-end iPhones (duplicate timestamps violating MediaPipe's strictly-increasing microsecond timestamp contract), restoring stable stream output.
- Developed UVC Camera support and resolved cross-platform audio capture edge cases including burst-data warnings and audio route changes.

**Infrastructure & Build System**

- Migrated third-party libraries (ffmpeg-kit, ExoPlayer) from Gradle to Bazel, reducing Android build time by 50% and iOS build time by 30%; achieved
reproducible builds for iOS frameworks and Android AARs.
- Led the Android 15 16KB page-size migration across the entire native stack (SenseME, ffmpeg-kit, openh264, libuvccamera), including rebuilding ffmpeg-kit from an archived build config after upstream dropped 16KB support.
- Resolved Xcode 26 compatibility issues ensuring uninterrupted iOS SDK delivery.
- Designed LangEngineSDK with dual-target support (iOS/Android), Swift module stability, and proper modulemap structure for seamless integration.
**Interactive Physics System — "Bonk Gift** "
- Engineered a complete Box2D-based interactive gift system with gesture recognition (SenseMe SDK), supporting three interaction modes: Normal,
Merge, and Gesture.
- Designed gift-manifest protobuf schema and built a comprehensive asset loading system supporting WebP animations, PNG sprites, and WAV/MP
audio from ZIP archives with memory-efficient caching.

**Vtuber Streaming System (Live2D)**

- Built end-to-end Vtuber streaming system integrated into the PlayOne product: designed sprite anchoring for mesh-attached decorations,
implemented color-palette shaders for real-time hair/outfit color changes, and developed a submodel outfit system (hair, eyes, mouth, accessories)
with 25fps on-device rendering.
- Created dynamic sprite rendering with WebP support and avatar-part color customization using palette-texture mapping and custom GLSL shaders,
handling premultiplied alpha and grayscale-to-color transformations.
**Media Editor SDK**
- Developed comprehensive image/video editing SDK for Android/iOS with text/image sticker systems supporting rotation, scaling, translation, and
crop-aware rendering using model matrix transformations.

## Senior Software Engineer - Quanta Computer Sep 2017 - Mar 2022

### Swift/Kotlin/Python/C++/Firebase/Azure IoT/RxKotlin/RxSwift

**Smartwatch Product Engineering**

- Delivered 3 carrier-distributed kids smartwatch products: Verizon GizmoWatch 2, Verizon GizmoWatch Disney Edition, and T-Mobile SyncUp Kids
Watch — shipped to retail across US carriers.
- Integrated the client-side Azure IoT SDK and Device Provisioning Service (DPS) registration flow on T-Mobile SyncUp, supporting reliable cloud connectivity for 1M+ shipped devices.
- Built the client-side architecture of T-Mobile SyncUp Kids Watch: MVVM + Repository pattern, database layer, API integration, source control standards, and scrum processes.
- Refactored database and Retrofit layers on GizmoWatch Disney Edition, improving code maintainability and reducing technical debt.
- Defined team coding standards and backend API specifications, facilitating consistent delivery across a cross-functional global team.

## Software Engineer Intern - Delta Electronic Aug 2016 - Jun 2017

## Education

## M.S. Computer Science, National Central University 2015 - 2017

## B.S. Electronic engineering , National Kaohsiung University of Applied Sciences 2011 - 2015

```
Phone
+886-975-086-326
```
```
E-mail
microvoyager@hotmail.com
```
```
Github
ymh514
```


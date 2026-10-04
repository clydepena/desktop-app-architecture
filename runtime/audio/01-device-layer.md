# The device layer

The lowest part of an audio engine, and strictly speaking part of the **platform layer**: one interface over the
operating system's audio APIs (WASAPI on Windows, CoreAudio on macOS and iOS, ALSA or PulseAudio/PipeWire on
Linux, or a portable library such as OpenAL).

## What it offers

Opening an output (or input) device with:

- a **sample rate**, e.g. 48,000 samples per second
- a **channel layout**: mono, stereo, 5.1, 7.1
- a **sample format**, usually 32-bit floating point inside the engine
- a **buffer size**, e.g. 256 or 512 samples per callback

```
interface AudioDevice {
    DeviceInfo[] listOutputs();
    void open(DeviceInfo device, int sampleRate, ChannelLayout layout, int bufferFrames, AudioCallback callback);
    void close();
    Event<DeviceChange> changed();   // plugged, unplugged, default device switched
}

interface AudioCallback {
    void fill(float[] output, int frames);   // called on the device's own thread
}
```

## The pull model

Rendering **pushes**: the main loop decides when to draw a frame. Audio **pulls**: the device decides when it
needs more sound, and calls the callback on its own high-priority thread:

```
device.open(..., (output, frames) -> {
    engine.mix(output, frames);   // must finish in time, every time
});
```

The time available is the buffer's length. At 48 kHz with 512-sample buffers, each callback has about
**10.7 ms**. If it is late even once, the device has nothing to play: a click or a gap.

## Device changes

Audio devices come and go while the app runs: headphones plugged in, a Bluetooth headset disconnecting, the user
switching the default output in the OS. The device layer reports these as events, and the engine closes and
reopens the device (on the new default, or the one the user chose), without the application noticing. This is the
audio version of "device lost" in rendering.

Other details the layer hides:

- **Sample rate mismatches:** the hardware may run at 44.1 kHz while the engine mixes at 48 kHz; either the OS or
  the layer converts.
- **Exclusive vs shared mode:** exclusive access gives lower latency but takes the device from other apps; shared
  mode goes through the OS mixer.
- **Input devices** (microphones) work the same way, with a callback delivering captured samples.

## Patterns

- **Bridge / Adapter:** the engine is written against `AudioDevice`; one backend per OS API.
- **Callback on a foreign thread:** the engine doesn't own the audio thread's timing; it must be ready whenever
  the device calls ([Threads and timing](05-threads-and-timing.md)).

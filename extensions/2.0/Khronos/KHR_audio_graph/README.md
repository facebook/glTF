

## KHR Audio Graph Design




## **Contributors**



* Chintan Shah, Meta
* Alexey Medvedev, Meta

Copyright 2024 The Khronos Group Inc.
See [Appendix](#appendix-full-khronos-copyright-statement) for full Khronos Copyright Statement.

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

This extension provides a standardized way to represent an audio graph consisting of multiple interconnected audio node objects to create the final audio output. It’s designed for managing audio routing, mixing, and processing which can be mapped to glTF assets.


### Motivation

The core idea involves an audio graph consisting of multiple interconnected audio node objects to create the final audio output. This design seeks to incorporate the audio capabilities found in modern web, game, and XR engines as well as processing, mixing, and filtering functions available in audio production softwares. Although this system is designed with diverse use cases in mind, it might not cover every specialized feature found in state-of-the-art audio tools. In such instances, users are encouraged to develop custom extensions. Nevertheless, the proposed system will support many complex audio applications by default and has been designed to facilitate future expansion with more sophisticated features.


## Graph based audio processing

**Audio nodes** are the building blocks of an audio graph for rendering audio to the audio hardware. Graph based audio routing allows arbitrary connections between different audio node objects. An audio graph can be represented by **audio sources**, the **audio destination/sink**, and intermediate processing nodes. Each node can have **inputs** and/or **outputs**. A source node has no inputs and a single output. A destination or sink node has one input and no outputs. In the simplest case, a single source can be routed directly to the output.

One or more intermediate processing nodes such as filters can be placed between the source and destination nodes. Most processing nodes will have one input and one output. Each type of audio node differs in the details of how it processes or synthesizes audio. But, in general, an audio node will process its inputs (if it has any), and generate audio for its outputs (if it has any). An output may connect to one or more audio node inputs, thus fan-out is supported. An input (except source and sink) may be connected to one or more audio node outputs, thus fan-in is supported. Each input and output has one or more channels. The exact number of channels depends on the details of the specific audio node.



## Features

The core specification supports:

- A JSON-based audio graph with nodes, connections, and optional sinks (emitters) for spatialized audio.
- Audio content from memory (bufferView/uri) or procedural oscillator data.
- Encoding metadata (bitsPerSample, duration, samples, sampleRate, channels).
- Playback controls (play, stop, pause, looping, speed) with timing in milliseconds (ms).
- Spatialized audio via Emitter + glTF node transforms (HRTF/equal-power, distance attenuation, sound cones).
- Basic processing: gain, delay, filtering, distortion (waveshaping), and IR-based reverb.
- Channel routing: splitting, merging, up/down-mixing, and summing mixes.
- Animation and dynamic updates (see KHR_animation_pointer; refined separately).

### Definitions

- **Audio Node**: a function that generates or processes audio.
- **Source Node**: 0 inputs / 1 output; content via `data.oneOf` — audioData reference or oscillator data.
- **Processor Node**: typically 1 input / 1 output; includes gain, delay, filters, waveshaper, reverb. Splitter is 1→N; Channel Merger is N→1; mixers as declared.
- **Emitter Node**: 1 input / 0 outputs. Acts as a sink; spatialization derives from Emitter + glTF node transform.
- **Listener**: scene-level (not a graph node), attached to camera; refined separately.
- **Audio Graph**: directed acyclic graph (DAG) of nodes with explicit connections by node indices and optional port indices.
  * **Inputs** are nodes that define the interface for a graph’s incoming data.
  * **Outputs** are nodes that define the interface for a graph’s outgoing data.

## Graph based audio processing

Audio nodes are the building blocks of an audio graph for rendering audio to the audio hardware. Graph based audio routing allows arbitrary connections between different audio node objects. An audio graph can be represented by audio sources, the audio destination/sink, and intermediate processing nodes. Each node can have inputs and/or outputs. A source node has no inputs and a single output. A destination or sink node has one input and no outputs. In the simplest case, a single source can be routed directly to the output.

One or more intermediate processing nodes such as filters can be placed between the source and destination nodes. Most processing nodes will have one input and one output. Each type of audio node differs in the details of how it processes or synthesizes audio. But, in general, an audio node will process its inputs (if it has any), and generate audio for its outputs (if it has any). An output may connect to one or more audio node inputs, thus fan-out is supported. An input (except source and sink) may be connected to one or more audio node outputs, thus fan-in is supported. Each input and output has one or more channels. The exact number of channels depends on the details of the specific audio node.

## Extension Declaration

Add `KHR_audio_graph` to `extensionsUsed` and declare the extension payload under `extensions`.

```json
{
  "extensionsUsed": ["KHR_audio_graph"],
  "extensions": {
    "KHR_audio_graph": {
      "schemaVersion": "0.1",
      "audioData": [ /* see Encoding + AudioData */ ],
      "graphs": [ /* see Graph Schema */ ]
    }
  }
}
```


### Conventions

- Units: all time values are expressed in milliseconds (ms). Implementations mapping to Web Audio convert ms→s for API calls (e.g., `DelayNode.delayTime`).
- Connections: edges reference node array indices, with optional `input`/`output` port indices. Omitted indices refer to the default/main port.
- Sinks: a valid graph must have at least one sink — either an `emitter` node or a node index listed in `outputs[]`.

### Web Audio Mapping (Informative)

- General units: Unless specified otherwise, times are expressed in milliseconds (ms). When mapping to Web Audio API, timing values are converted to seconds (s), e.g., `delayTime(ms)` → `DelayNode.delayTime(s)`.
- Source Node (4.1): Maps to `AudioBufferSourceNode`. `playbackSpeed` corresponds to `playbackRate`. `loopStart`/`loopEnd` map to seconds. `when` is seconds and is passed to `start(when, offset, duration)` after ms→s conversion for `offset`/`duration`.
- Oscillator Data (4.3): Maps to `OscillatorNode`. If `type = square` and `pulseWidth` is provided, a static PWM can be implemented via `PeriodicWave`. Dynamic PWM modulation is out of scope.
- Gain (6.1): Maps to `GainNode`. Optional smoothing may be expressed with `interpolation: 'linear'|'custom'` and `duration (ms)`. Linear smoothing uses `linearRampToValueAtTime`; ‘custom’ can be approximated with `setTargetAtTime`.
- Delay (6.2): Maps to `DelayNode` with `delayTime` in seconds (ms→s conversion).
- Filters (6.8.x): Map to `BiquadFilterNode` with corresponding `type`. `frequency` (Hz ≥ 0), `qualityFactor` maps to `Q` (≥ 0), `gain` (dB). Optional `bypass` may be supported via build‑time routing or runtime dry/wet crossfade.
- Reverb (6.9): IR‑based reverb maps to `ConvolverNode`. The overall wet/dry is implemented by summing dry and wet paths with respective gains. Algorithmic reverbs are out of scope here and may be future optional nodes.
- Panning: For simple stereo pan use `StereoPannerNode`; for 3D spatialization use `PannerNode`. In this extension, spatialization is typically realized by an `emitter` + glTF node transform.
- Emitter (5.1): Implemented as a gain stage with optional spatialization via `PannerNode`. A single scene listener is assumed; listener details remain part of the specification.
- Pitch Shifter (6.3): Deferred from initial scope; commonly realized via custom DSP/Worklet.
- Bypass (Implementation Note): For processors, an optional `bypass: boolean` may be honored by either (1) build‑time rewiring (omit node), or (2) runtime dry/wet crossfade. Exact behavior may be implementation‑defined.
- Channel Interpretation (Implementation Note): Where supported by Web Audio (`AudioNode.channelInterpretation`), an optional `channelInterpretation: 'speakers'|'discrete'` may refine mixing behavior.


### Audio Nodes

Node kinds and parameters are defined by JSON Schemas (see `schema/`). Graphs use a discriminator shape `{ "kind": "<enum>", "params": { ... }, "label?": string }` for each node entry.

### KHR_animation_pointer

Animation integration remains out of scope for this pass. Refer to `KHR_animation_pointer` for animating node parameters via JSON Pointers. Pointers to graph nodes/params will be refined in a later phase.

## Listener

TODO:

Listener node should be attached to the camera


### Graphs

Graphs are defined under `graphs[]` using the following JSON structure.

Example — global output sink via `outputs[]`

```json
{
  "name": "simple",
  "nodes": [
    {
      "label": "osc source",
      "kind": "source",
      "params": {
        "data": { "oscillator": { "type": 1, "frequency": 440, "pulseWidth": 0.25 } },
        "when": 0,
        "duration": 500
      }
    },
    { "label": "gain", "kind": "gain", "params": { "gain": 0.5 } }
  ],
  "connections": [
    { "from": { "node": 0 }, "to": { "node": 1 } }
  ],
  "outputs": [1]
}
```

Example — emitter sink (no `outputs[]`)

```json
{
  "name": "with-emitter",
  "nodes": [
    { "label": "buffer source", "kind": "source", "params": { "data": { "audioData": 0 }, "when": 0, "loop": false } },
    { "label": "emitter", "kind": "emitter", "params": { "emitterType": "global", "gain": 1 } }
  ],
  "connections": [ { "from": { "node": 0 }, "to": { "node": 1 } } ]
}
```

### Connections

Connections are declared with `from` and `to` endpoints referencing node indices and optional port indices. See Graph Schema for the normative shape.


### Graph Schema (JSON)

This section describes the JSON shape used by KHR_audio_graph for authoring and interchange. It complements the descriptive material above.

Graph object

```json
{
  "name": "optional name",
  "nodes": [ { "kind": "...", "params": { /* node parameters */ }, "label": "optional debug" } ],
  "connections": [ { "from": { "node": 0, "output": 0 }, "to": { "node": 1, "input": 0 } } ],
  "outputs": [ 3, 7 ]
}
```

- nodes: array of node entries. Each entry uses a discriminator:
  - kind: one of
    - source, gain, delay
    - lowpass, highpass, bandpass, lowshelf, highshelf, peaking, notch, allpass
    - reverb, waveshaper
    - splitter, channelmerger, channelmixer, audiomixer
    - emitter
  - params: object validated by the corresponding node schema.
  - label: optional human‑readable string for debugging. Identifiers are not used for connections; array indices are used instead.
- connections: each describes an edge from an upstream node/output port to a downstream node/input port. The `output`/`input` indices are optional and default to the main port when omitted.
- outputs: optional array of node indices which feed the global/default destination when no emitters are present (supports non‑spatial/global audio). If emitters are present, they are also valid sinks.

Source node data (ms-based timing)

```json
{
  "kind": "source",
  "params": {
    "data": { "audioData": 0 },
    "when": 0, "loop": false, "offset": 0, "duration": 250, "playbackSpeed": 1.0
  }
}

{
  "kind": "source",
  "params": {
    "data": { "oscillator": { "type": 1, "frequency": 440, "pulseWidth": 0.25 } },
    "when": 0, "duration": 500
  }
}
```

- Source is a single 0‑input/1‑output node kind. Its `params.data` is oneOf:
  - `{ "audioData": <glTFid> }` – refers to an entry in the extension’s `audioData[]` registry
  - `{ "oscillator": <oscillator parameters> }` – inlined oscillator data
- Web Audio Mapping: maps to `AudioBufferSourceNode` (or `OscillatorNode` via `PeriodicWave` when `oscillator` is used). Convert timing fields from ms→s when calling `start(when, offset, duration)` and setting `loopStart/loopEnd`.

Spatialization via Emitter

- Spatialization and panning are realized by the Emitter sink combined with the transform of the glTF node the emitter is attached to. No separate panner node is defined.
- Implementers typically map `spatializationModel` to `panningModel` and attenuation fields to `distanceModel`, `refDistance`, `maxDistance`, `rolloffFactor`, and `cone*`.

Listener

- The Listener is not a graph node. It remains a scene‑level concept attached to the camera (per node extension) and is outside graph connectivity.

### Animation and Pointers (TODO)

- KHR_animation_pointer applicability across Listener, emitters, and node parameters is deferred in this pass. A later revision will specify the accessible properties and JSON Pointer paths. Implementations may experiment, but no normative guidance is provided here yet.

### Emitter Binding to glTF Nodes

- Graph emitters are placement‑agnostic. Binding to scene transforms is done at the glTF node level via a node extension:
  - Scalar form: `extensions.KHR_audio_graph = { "emitter": <emitterId> }`
  - Array form: `extensions.KHR_audio_graph = { "emitters": [ <emitterId>, ... ] }`
- Each glTF node referencing an emitter id creates a distinct emitter instance driven by that node’s transform (translation/rotation/scale). Multiple nodes may reference the same emitter id.
- Runtime optimization: upstream graph routing is shared; only the final spatial stage (panner + optional post‑panner gain) is instantiated per instance.

Graph rules (normative)

- Graphs are directed acyclic graphs (no cycles).
- An emitter node has exactly one incoming connection and no outgoing connections.
- A graph must contain at least one sink (an emitter or a node listed in `outputs`).
- Multiple sources may reference the same audio data.
- Fan‑out/fan‑in is allowed subject to node input/output arities (e.g., splitter 1→N, channelmerger N→1, mixers N→1).

The following example shows a minimal graph using the container shape; see `tools/spec-validate/examples/graph.example.json` for a full sample.

### BYPASS

Each processing node can have a `bypass` property. If `bypass` is set to `true`, the node will be excluded from execution.


## 4. Audio source


### 4.1 Source node (0 input / 1 output)

Audio sources reference audio data and define playback properties for it. An audio source node has no inputs and exactly one output, which has the same number of channels as indicated in encoding properties.


|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the audio source in the scene.|Yes|
|**data**|'object'|Audio or oscillator. See 4.2 AudioData. See 4.3 Oscillator data. Only selective source node properties apply with oscillator data.|Yes|
|**priority**|'integer'|Determines the priority of this audio source among all the ones that coexist in the scene (0 = most important, 256 = least important, default = 128). Need to  persist and propagate priority with downstream processing.|No|
|**gain**|'number'|Gain value applied to the audio.|No|
|**state**|'string'|Playback state (paused, playing, stopped).|No|
|**autoPlay**|'boolean'|Play on load.|No|
|**loop**|'boolean'|Playback in a loop (false = no-loop, true = loop). If set to true, then once playback reaches the time specified by “loop end position” (or the end of the asset, whichever is first), the source node will continue playback again from a position specified by the “loop start position” property.|No|
|**loopStart**|'number'|The starting position in ms when looping.|No|
|**loopEnd**|'number'|The ending position (ms) of the loop.|No|
|**playbackSpeed**|'number'|Rate of playback. A value of 1.0 would playback the audio at the standard rate. A value of 2.0 would play back the asset at double the speed.|No|
|**duration**|'number'|Length of the underlying audio data in ms.|No|
|**offset**|'number'|If 0 is passed in for this value, then playback will start from the beginning of the buffer. Offset sould not be  negative. If offset is greater than loopEnd, playbackRate is positive or zero, and loop is true, playback will begin at loopEnd. If offset is greater than loopStart, playbackSpeed is negative, and loop is true, playback will begin at loopStart. offset is silently clamped to [0, duration], when startTime is reached, where duration is the value of the duration attribute of the AudioData or Oscillator set to the buffer attribute of this node.|No|
|**when**|'number'|The when parameter describes at what time (in seconds) the sound should start playing.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 4.2 Audio data

Audio data objects define where audio data is located and what format the data is in. The data is either accessed via a bufferView or uri.

|   |Type|Description|Required|
|---|---|---|---|
|**bufferView**|'integer'|The index of the buffer view that contains the audio data. The buffer represents an audio asset residing in memory, created from decoding an audio file, or from raw data.|Yes|
|**encodingProperties**|'KHR_audio_graph.encoding.schema.json'|Encoding properties. See 4.4 Encoding properties.|Yes|
|**mimeType**|'string'|The audio's MIME type. Required if buffer view is defined.|Yes|
|**uri**|'string'|The uri of the audio file. Relative paths are relative to the .gltf file.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 4.3 Oscillator data

This represents an audio source generating a periodic waveform. It can be set to a few commonly used waveforms. Oscillators are common foundational building blocks in audio synthesis.

|   |Type|Description|Required|
|---|---|---|---|
|**type**|'string'|Specifies the waveform type (saw, square, triangle, sine, custom).|Yes|
|**frequency**|'number'|The Oscillator frequency, from 0-20kHz. The default value is 440 Hz.|Yes|
|**pulseWidth**|'number'|The amount of pulse width modulation applied when the “square” waveform is selected. A 0.5 value will produce a pure square wave, and increasing or decreasing the value will add harmonics which change the timbre of the sound.|Yes|
|**encodingProperties**|'KHR_audio_graph.encoding.schema.json'|Encoding properties. See 4.4 Encoding properties.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 4.4 Encoding properties

|   |Type|Description|Required|
|---|---|---|---|
|**bitsPerSample**|'integer'|Number of bits per audio sample.|Yes|
|**duration**|'number'|Length of the underlying audio data in ms.|Yes|
|**samples**|'integer'|Number of samples.|Yes|
|**sampleRate**|'number'Audio sampling rate (Hz).|Yes|
|**channels**|'integer'|Number of audio channels.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


## 5. Audio sink/destination


### 5.1 Emitter node (1 input / 0 output)

Audio emitter of type “Global” is non-spatialized, whereas a “Spatial” emitter is used for spatial audio. A scene can contain one or more global emitters. Spatial emitter is connected to a scene node. A scene node can have one or more spatial emitters. A spatial emitter by default inherits the pose (position and orientation) properties of the scene node, hence we do not support updating these properties within the spatial emitter node.

Using a spatial emitter, an audio stream can be spatialized or positioned in space relative to a listener node. A scene has a single listener node. Both spatial emitters and listeners have a position in 3D space. Spatial emitters have an orientation representing in which direction the sound is projecting. Additionally, they have a sound cone representing how directional the sound is. For example, the sound could be omnidirectional, in which case it would be heard anywhere regardless of its orientation, or it can be more directional and heard only if it is facing the listener.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the audio source in the scene.|Yes|
|**emitterType**|'string'|Emitter type (global, spatial).|Yes|
|**gain**|'number'|Gain applied to the signal by the emitter. It's a linear value in the range [0, 1]. |No|
|**spatialProperties**|'KHR_audio_graph.spatial.schema.json'|See 5.2 Spatial properties.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 5.2 Spatial properties

|   |Type|Description|Required|
|---|---|---|---|
|**spatializationModel**|'string'|Determines which spatialization model will be used to position the audio in 3D space (equal power, HRTF, Custom).equalpower: Represents the equal-power panning algorithm, generally regarded as simple and efficient. equalpower is the default value. HRTF: Renders a stereo output of higher quality than equalpower — it uses a convolution with measured impulse responses from human subjects. Custom: User-defined panning algorithm.|Yes|
|**attenuation**|'object'|See 5.3 Attenuation properties.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 5.3 Attenuation properties

|   |Type|Description|Required|
|---|---|---|---|
|**distanceModel**|'string'|Specifies the distance model for the audio emitter  linear, inverse, exponential, custom.|Yes|
|**refDistance**|'number'|A reference distance for reducing volume as the emitter moves further from the listener.|Yes|
|**maxDistance**|'number'|The maximum distance between the emitter and listener, beyond which the audio cannot be heard.|Yes|
|**rolloffFactor**|'number'|Describes how quickly the volume is reduced as the emitter moves away from the listener.|Yes|
|**shape**|`string`|Shape in which emitter emits audio (cone, omnidirectional, custom).|No|
|**coneInnerAngle**|`object`|The angular diameter of a cone inside of which there will be no angular volume reduction.|No|
|**coneOuterAngle**|`object`|A parameter for directional audio sources that is an angle, in degrees, outside of which the volume will be reduced to a constant value of coneOuterGain.|No|
|**coneOuterGain**|`object`|A parameter for directional audio sources that is the gain outside of the cone outer angle. It's a linear value in the range [0, 1].|No|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 5.5 Audio listener node (0 input / 0 output)

Describes the position and other physical characteristics of a listener from which the audio output of spatial emitter nodes is heard when spatial audio processing is used. A listener node is typically attached to the main camera and by default inherits camera pose (position and orientation) properties. Hence, we do not support updating these properties within the listener node.


## 6. Audio processors


### 6.1 Gain node (1 input / 1 output)

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**gain**|'number'|The gain to apply. In's a linear value in the range [0..1]. Should be nultiplied to previous node (emitter, source) gain. Once set, the actual gain applied will transition from it’s current setting to the new one set over 'duration' milliseconds. |Yes|
|**interpolation**|'string'|The curve to apply when changing gains (linear, custom).|No|
|**duration**|'number'|When changing gain, this parameter controls how long to spend interpolating in ms from the previously set gain value to the one that is specified.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.2 Delay node (1 input / 1 output)

The node that causes a delay between the arrival of an input data and its propagation to the output. A delay node always has exactly one input and one output, both with the same amount of channels.


|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**delayTime**|'number'|Representing the amount of delay to apply, specified in ms.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.3 Wave shaper node (1 input / 1 output)

Use the node to apply waveshaping distortion to an audio signal.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the node in the scene.|Yes|
|**amount**|'number'|Distortion intensity in [0..1]. Implementations may map this to a shaping curve (e.g., tanh).|No|
|**oversample**|'string'|Optional oversampling factor for distortion quality (none, 2x, 4x).|No|
|**curve**|'array'|Optional explicit shaping curve as a normalized array in [-1..1].|No|
|**bypass**|`boolean`|Disables this processor while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete".|No|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.4 Channel splitter node (1 input / N outputs)

Node for accessing the individual channels of an audio stream in the routing graph. It has a single input, and a number of outputs which equals the number of channels in the input audio stream. For example, if a stereo input is connected to this node then the number of active outputs will be two (one from the left channel and one from the right).

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use discrete to  maps channels to outputs.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.5 Channel merger node (N inputs / 1 output)

Node for combining channels from multiple audio streams into a single audio stream. It has a variable number of inputs, and a single output whose audio stream has a number of channels equal to the number of inputs. To merge multiple inputs into one stream, each input should be a single channel audio stream.


|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use discrete to  maps channels to outputs.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.6 Channel mixer node (1 input / 1 output)

Up-mixing refers to the process of taking a stream with a smaller number of channels and converting it to a stream with a larger number of channels. Down-mixing refers to the process of taking a stream with a larger number of channels and converting it to a stream with a smaller number of channels. Channel mixer node should ideally use these [mixing rules](https://webaudio.github.io/web-audio-api/#mixing-rules).


|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**outputChannels**|'number'|Number of channels in output audio.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use discrete to  maps channels to outputs.|Yes|
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.7 Audio mixer node (N inputs / 1 output)

Use the Audio mixer node to combine the output from multiple audio sources. Number of channels should be the same across all inputs.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.8 Filter nodes (1 input / 1 output)

Low-order filters are the building blocks of basic tone controls (bass, mid, treble), graphic equalizers, and more advanced filters. Multiple filters can be combined to form more complex filters. The filter parameters such as frequency can be changed over time for filter sweeps, etc. Each filter node always has exactly one input and one output.

TODO:
Add https://www.w3.org/TR/webaudio/#filters-characteristics explanations for the filter math

### 6.8.1 Lowpass filter node (1 input / 1 output)

A lowpass filter allows frequencies below the cutoff frequency to pass through and attenuates frequencies above the cutoff. It implements a standard second-order resonant lowpass filter with 12dB/octave rolloff.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the lowpass filter node in the scene.|Yes|
|**frequency**|'number'|The cutoff frequency in Hz.|Yes|
|**qualityFactor**|`number`|Controls how peaked the response will be at the cutoff frequency. A large value makes the response more peaked.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.8.2 Highpass filter node (1 input / 1 output)

A highpass filter allows frequencies above the cutoff frequency to pass through and attenuates frequencies below the cutoff. It implements a standard second-order resonant highpass filter with 12dB/octave rolloff.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the highpass filter node in the scene.|Yes|
|**frequency**|'number'|The cutoff frequency in Hz.|Yes|
|**qualityFactor**|`number`|Controls how peaked the response will be at the cutoff frequency. A large value makes the response more peaked.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.8.3 Bandpass filter node (1 input / 1 output)

A bandpass filter allows a range of frequencies to pass through and attenuates the frequencies below and above this frequency range. It implements a second-order bandpass filter.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the bandpass filter node in the scene.|Yes|
|**frequency**|'number'|The cutoff frequency in Hz.|Yes|
|**qualityFactor**|`number`|Controls the width of the band. The width becomes narrower as the Q value increases.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.8.4 Lowshelf filter node (1 input / 1 output)

The lowshelf filter allows all frequencies through, but adds a boost (or attenuation) to the lower frequencies. It implements a second-order lowshelf filter.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the lowshelf filter node in the scene.|Yes|
|**frequency**|'number'|The upper limit of the frequences where the boost (or attenuation) is applied in Hz.|Yes|
|**gain**|`number`|"The boost, in dB, to be applied. If the value is negative, the frequencies are attenuated.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|

### 6.8.5 Highshelf filter node (1 input / 1 output)

The highshelf filter is the opposite of the lowshelf filter and allows all frequencies through, but adds a boost to the higher frequencies. It implements a second-order highshelf filter.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the highshelf filter node in the scene.|Yes|
|**frequency**|'number'|The lower limit of the frequences where the boost (or attenuation) is applied in Hz.|Yes|
|**gain**|`number`|"The boost, in dB, to be applied. If the value is negative, the frequencies are attenuated.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|


### 6.8.6 Peaking filter node (1 input / 1 output)

The peaking filter allows all frequencies through, but adds a boost (or attenuation) to a range of frequencies.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the peaking filter node in the scene.|Yes|
|**frequency**|'number'|The center frequency of where the boost is applied.|Yes|
|**qualityFactor**|`number`|Controls the width of the band of frequencies that are boosted. A large value implies a narrow width.|No|
|**gain**|`number`|"The boost, in dB, to be applied. If the value is negative, the frequencies are attenuated.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|

### 6.8.6 Notch filter node (1 input / 1 output)

The notch filter (also known as a band-stop or band-rejection filter) is the opposite of a bandpass filter. It allows all frequencies through, except for a set of frequencies.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the notch filter node in the scene.|Yes|
|**frequency**|'number'|The center frequency of where the boost is applied.|Yes|
|**qualityFactor**|`number`|Controls the width of the band of frequencies that are boosted. A large value implies a narrow width.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|

### 6.8.6 Allpass filter node (1 input / 1 output)

An allpass filter allows all frequencies through, but changes the phase relationship between the various frequencies. It implements a second-order allpass filter.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the notch filter node in the scene.|Yes|
|**frequency**|'number'|The frequency where the center of the phase transition occurs. Viewed another way, this is the frequency with maximal group delay.|Yes|
|**qualityFactor**|`number`|Controls how sharp the phase transition is at the center frequency. A larger value implies a sharper transition and a larger group delay.|No|
|**bypass**|`object`|Disables this filter while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|

### 6.9 Reverb node (1 input / 1 output)

Reverberation is the persistence of sound in an enclosure after a sound source has been stopped. This is a result of the multiple reflections of sound waves throughout the room arriving at the ear so closely spaced that they are indistinguishable from one another and are heard as a gradual decay of sound.

|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**mix**|'number'|Blend between the source signal ('dry') and the reverb effect. A value of 0 will not add any reverb. A value of 50 will mix the signal 50% dry and 50% reverberation. A value of 100 will result in only reverberation, without any of the source signal.|Yes|
|**earlyReflectionsGain**|'number'|Loudness control for the reverb decay as it returns to silence.|Yes|
|**diffusionGain**|'number'|Loudness control for the reverb decay as it returns to silence.|Yes|
|**roomSize**|'number'|Approximates the size of the room you want to simulate in meters from wall to wall.|Yes|
|**reflectivity**|'number'|Defines how much of the audio is reflected at each bounce on a wall. Value range: 0 to 100. Low values will simulate softer sounding materials like carpet or curtains. High values will simulate harder materials like wood, glass or metal. A value of 100 will result in self-oscillation and is not recommended.|Yes|
|**reflectivityHigh**|'number'|Separate value for the reflectivity of high frequencies.|Yes|
|**reflectivityLow**|'number'|Separate value for the reflectivity of low frequencies.|Yes|
|**earlyReflections**|'number'|The number of early reflections of reverberation. The value range is 0 to 32.|Yes|
|**minDistance**|'number'|The distance from the centerpoint that the reverb will have full effect at.|Yes|
|**maxDistance**|'number'|The distance from the centerpoint that the reverb will not have any effect.|Yes|
|**reflectionDelay**|'number'|Initial reflection delay time in ms.|Yes|
|**reverbDelay**|'number'|Late reverberation delay time relative to initial reflection.|Yes|
|**diffusionGain**|'number'||Yes|
|**bypass**|`boolean`|Disables this processor while still allowing unprocessed audio signals to pass.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
|**extensions**|`object`|JSON object with extension-specific objects.|No|
|**extras**|[`any`](#reference-any)|Application-specific data.|No|



## 7. Audio graph rules



* Multiple audio source nodes can reference the same audio data.
* An output of a source node can serve as input to multiple processor and emitter nodes.
* An output of a processor node can serve as input to multiple processor and emitter nodes.
* One audio emitter node can have only one input.
* A scene can have multiple emitters.
* A node can have multiple spatial emitters.
* A scene can have only one audio listener.

## Appendix: Full Khronos Copyright Statement

Copyright 2013-2017 The Khronos Group Inc.

Some parts of this Specification are purely informative and do not define requirements
necessary for compliance and so are outside the Scope of this Specification. These
parts of the Specification are marked as being non-normative, or identified as
**Implementation Notes**.

Where this Specification includes normative references to external documents, only the
specifically identified sections and functionality of those external documents are in
Scope. Requirements defined by external documents not created by Khronos may contain
contributions from non-members of Khronos not covered by the Khronos Intellectual
Property Rights Policy.

This specification is protected by copyright laws and contains material proprietary
to Khronos. Except as described by these terms, it or any components
may not be reproduced, republished, distributed, transmitted, displayed, broadcast
or otherwise exploited in any manner without the express prior written permission
of Khronos.

This specification has been created under the Khronos Intellectual Property Rights
Policy, which is Attachment A of the Khronos Group Membership Agreement available at
www.khronos.org/files/member_agreement.pdf. Khronos grants a conditional
copyright license to use and reproduce the unmodified specification for any purpose,
without fee or royalty, EXCEPT no licenses to any patent, trademark or other
intellectual property rights are granted under these terms. Parties desiring to
implement the specification and make use of Khronos trademarks in relation to that
implementation, and receive reciprocal patent license protection under the Khronos
IP Policy must become Adopters and confirm the implementation as conformant under
the process defined by Khronos for this specification;
see https://www.khronos.org/adopters.

Khronos makes no, and expressly disclaims any, representations or warranties,
express or implied, regarding this specification, including, without limitation:
merchantability, fitness for a particular purpose, non-infringement of any
intellectual property, correctness, accuracy, completeness, timeliness, and
reliability. Under no circumstances will Khronos, or any of its Promoters,
Contributors or Members, or their respective partners, officers, directors,
employees, agents or representatives be liable for any damages, whether direct,
indirect, special or consequential damages for lost revenues, lost profits, or
otherwise, arising from or in connection with these materials.

Vulkan is a registered trademark and Khronos, OpenXR, SPIR, SPIR-V, SYCL, WebGL,
WebCL, OpenVX, OpenVG, EGL, COLLADA, glTF, NNEF, OpenKODE, OpenKCAM, StreamInput,
OpenWF, OpenSL ES, OpenMAX, OpenMAX AL, OpenMAX IL, OpenMAX DL, OpenML and DevU are
trademarks of The Khronos Group Inc. ASTC is a trademark of ARM Holdings PLC,
OpenCL is a trademark of Apple Inc. and OpenGL and OpenML are registered trademarks
and the OpenGL ES and OpenGL SC logos are trademarks of Silicon Graphics
International used under license by Khronos. All other product names, trademarks,
and/or company names are used solely for identification and belong to their
respective owners.

---

## Implementation Notes (Non‑normative)

### Conventions: Units & Web Audio Mapping
- Unless specified otherwise, times are expressed in milliseconds (ms) in this specification. When mapping to the Web Audio API, timing values are converted to seconds (s). Examples:
  - `delayTime(ms)` → `DelayNode.delayTime(s)`
  - Source `offset(ms)` / `duration(ms)` → `start(whenSeconds, offsetSeconds, durationSeconds)`
  - `loopStart(ms)` / `loopEnd(ms)` → `AudioBufferSourceNode.loopStart/loopEnd` (seconds)

### Informal Mapping to Web Audio API
These notes describe a practical mapping for implementers. They do not change the normative syntax.

- Source (4.1)
  - Maps to `AudioBufferSourceNode`. `playbackSpeed` → `playbackRate`. Loop points in seconds. `when` uses seconds.
- Oscillator (4.3)
  - Maps to `OscillatorNode`. If `type='square'` and `pulseWidth` is provided, a static PWM can be realized via `PeriodicWave`. PWM modulation is out of scope.
- Gain (6.1)
  - Maps to `GainNode`. Optional smoothing may be expressed with `interpolation: 'linear'|'custom'` and `duration (ms)`. Linear → linear ramp; custom → approximated via `setTargetAtTime`.
- Delay (6.2)
  - Maps to `DelayNode`. Convert `delayTime` from ms to seconds.
- Filters (6.8.x)
  - Maps to `BiquadFilterNode` with corresponding `type`. `qualityFactor` → `Q`. `gain` is in dB where applicable.
- Reverb (6.9)
  - Realized as IR‑based `ConvolverNode`. Wet/dry ratio implemented by mixing dry and convolved paths with gains. Algorithmic room parameters are out of scope for the IR mapping.
- Panning
  - Spatialization is realized by the Emitter sink combined with the glTF node’s transform (position/orientation). Implementations map `spatializationModel` to `panningModel` and attenuation fields to `distanceModel`, `refDistance`, `maxDistance`, `rolloffFactor`, and `cone*` on the underlying engine. No separate panner node is defined in this specification.
- Channel Split/Merge/Mix (6.4–6.7)
  - Split: `ChannelSplitterNode`. Merge: `ChannelMergerNode`. Mixer: request up/down‑mix via `channelCountMode='explicit'` and `channelCount`, following Web Audio mixing rules. Audio mixer (N→1) can be realized by summing connections (e.g., a `GainNode` sum).

### Bypass Behavior (Optional)
- For processors, an optional `bypass: boolean` may be honored by either:
  1) Build‑time routing: rewire around the node, or
  2) Runtime wrapper: dry/wet crossfade around the processor.
- Both approaches are viable; exact behavior is implementation‑defined and may vary by engine.

### Channel Interpretation (Optional)
- Where supported by Web Audio (`AudioNode.channelInterpretation`), implementations MAY expose `channelInterpretation: 'speakers'|'discrete'` on nodes to refine mixing behavior. Not all runtimes support this property.
- Graph outputs
  - Graphs MAY omit emitters and provide an `outputs[]` array listing node indices whose outputs feed the default/global destination. This supports global/non-spatial audio use‑cases without defining a separate panner node.

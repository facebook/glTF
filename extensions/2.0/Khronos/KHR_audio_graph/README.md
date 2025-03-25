

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

The core specification should support these primary features:



* Graph based audio routing for simple or complex mixing and processing architectures.
* Processing of audio data stored in memory buffer or accessed via file paths.
* Capturing audio metadata such as encoding properties.
* Audio playback functionalities including playing, stopping, pausing, looping, and controlling playback speed.
* Spatialized audio supporting a wide range of 3D applications and immersive environments with 6DoF source/listener capabilities, panning models (equal power, HRTF), distance attenuation, and sound cones.
* Basic audio signal processing to control gain, delay, and pitch.
* Flexible handling of channels in an audio stream, allowing splitting, merging, up-mixing, or down-mixing.
* Audio mixing, reverb, and filtering with a set of low-order audio filters.
* Animation control and dynamic update of node properties.

### Definitions

The following is a set of definitions to provide context for the ausio graph representation.

* A **Audio Node** is a function that generates or operates upon audio data.

* **Node Input and Output Ports** The interface for a node’s incoming data is declared through **input ports**, which may be audio data, oscillator data or audio channels data . The interface for a node’s outgoing data is declared through **output ports**.

* There is a specific set of supported **Data Types**. Every port must have a data type.

* An **Audio channel data** contains the actual sound bitstream for a single channel

* An **Audio data** contains all channels for actual **Audio channel data**

* A **Node Graph** is a directed acyclic graph (DAG) of nodes, which may be used to define arbitrarily complex generation or processing networks.  Node Graphs describe a network of nodes flowing from source to listener, or define a complex or layered node in terms of simpler nodes. The former is called a **compound nodegraph** and the latter a **functional nodegraph**.

* Node Graph:
  * **Inputs** are nodes that define the interface for a graph’s incoming data.
  * **Outputs** are nodes that define the interface for a graph’s outgoing data.

## Graph based audio processing

Audio nodes are the building blocks of an audio graph for rendering audio to the audio hardware. Graph based audio routing allows arbitrary connections between different audio node objects. An audio graph can be represented by audio sources, the audio destination/sink, and intermediate processing nodes. Each node can have inputs and/or outputs. A source node has no inputs and a single output. A destination or sink node has one input and no outputs. In the simplest case, a single source can be routed directly to the output.

One or more intermediate processing nodes such as filters can be placed between the source and destination nodes. Most processing nodes will have one input and one output. Each type of audio node differs in the details of how it processes or synthesizes audio. But, in general, an audio node will process its inputs (if it has any), and generate audio for its outputs (if it has any). An output may connect to one or more audio node inputs, thus fan-out is supported. An input (except source and sink) may be connected to one or more audio node outputs, thus fan-in is supported. Each input and output has one or more channels. The exact number of channels depends on the details of the specific audio node.

## Extension Declaration

Usage of the procedural structure is indicated by adding the `KHR_texture_procedurals` extension identifier to the `extensionsUsed` array.

```json
{
    "extensionsUsed": [
        "KHR_audio_graph"
    ]
}
```

Usage of a given extension is defined in the `extensions` object as follows:
```json
{
    "extensions": {
        "KHR_audio_graph": {
            "audio_nodes": [],
            "procedurals" : []
        }
    }
}
```


### Data Types

The supported data types are:

* single `float`
* single `integer`
* single `boolean`
* single `channel`: Data stream for a single audio channel
* single `stream`: Data stream for all channel


### Audio Nodes

TODO:

We have to declare all node upfront, they will be referenced by the graph


FIXME:: input/output type of the node is actually determined by the node (scheme) itself. Either we need to express input/output type od remove it


Each Audio graph object is composed of:

  * An `audionodetype` category which must be one of the follwoing: audiodata, oscillator, source, mixer, TODO
  * A `type` which is the output type of the audio node. This is a supported data type, or `multioutput` if there is more than one output node for the graph.
  * A `value` actual JSON object that represents the data for this audio node - See (## 4. Audio source)

```JSON
{
  "audio_nodes" [
    {
      "audionodetype": "audiodata",
      "type": "stream",
      "uri" : "urltotheasset"
    },
    {
      "audionodetype": "source",
      "type": "stream",
      "value" : [
        "id": 0,
        "data" : 0,
        "autoPLaye" : "true"
      ]
    },
    {
      "audionodetype": "emitter",
      "type": "stream",
      "value" : [
        "id": 1,
        "emitterType" : "global",
      ]
    },
    ],
}
```

### KHR_Animation_pointer integration


TODO

Shoudl it be a part of nodes declaration of the actual graph?

```JSON
                    "extensions": {
                        "KHR_animation_pointer": {
                            "pointer": "/nodes/0/mixer1"
                        }
                    }
```

## Listener

TODO:

Listener node should be attached to the camera


### Procedurals Graphs

One ore more procedurals graphs can be defined in the `proceduras` array.

A graph __cannot__ be nested (contain another graph). Any such configurations must be “flattened” to single level graphs.

Each procedural graph object is composed of:

  * An optional string `name` identifier
  * A `nodetype` category which must be `nodegraph`
  * A `type` which is the output type of the graph. This is a supported data type, or `multioutput` if there is more than one output node for the graph.
  * Three array children:
    * `inputs` : lists input "interface" nodes for passing data into the graph.
    * `outputs` : lists output "interface" nodes for passing data out of the graph. See [Node Graph Connections](#node-graph-connections) for connection information.
    * `nodes` : processing nodes.

The structure of audio nodes is described in  [Audio Nodes](#procedural-nodes) section.

Note that input and output node types are `input` and `output` respectively.

#### Graph Structure

```JSON
{
  "name": "<optional name>",
  "nodetype": "nodegraph",
  "type": "<data-type>",
  "inputs": [],
  "outputs": [],
  "nodes": []
}
```

### Graph Nodes

* An atomic function is represented as a single node with the following properties:

    * An optional string `name` identifier

    * `nodetype` : a string identifier for the node type. This is an 'audionode' type or a custom node type.

    * `type` : the output type of the node. This is a supported data type or `multioutput` if there is more than one output for the node.

    * A list of input ports under an `inputs` array.
    If an `input` is specified it's input value overrides that of the node definition default.

    * A list of output ports under an `outputs` array. Every `output` defined by the node's definition __must__ be specified.

    * Each input port:
        * Must have a node type: `input`
        * May have an optional string `name` identifier
        * Must have a `type` which is a supported data type.
        * Either:
          * A `value` which is a constant value for the node. or
          * A connection specifier. See [Node Graph Connections](#node-graph-connections) for allowed connections.

    * All `output` ports specified by the node's definition must be specified for each node instance. Each output port:
        * Must have a node type: `output`
        * Must have a `type` which is a supported data type.

#### Node Structure

```JSON
{
  "name": "<node name>",
  "type": "<data type>",
  "audionode": <id of audionode>  or
  "inputs": [],
  "outputs": []
}
```

where an each input port in the `inputs` array  has the following structure:

```JSON
{
  "name": "<input name>",
  "nodetype": "input",
  "type": "<data type>",
  "node": <processing node index> or
  "input": <input node index> or
  "output": <output node index>
}
```
and each output port in the `outputs` array has the following structure:

```JSON
{
  "name": "<input name>",
  "nodetype": "output",
  "type": "<data type>",
}
```

### Node Graph Connections

Connections inside a graph can be made:

* To a `node input` from a an `nodegraph input` by specifying a `input` value which is an index into the graph's `inputs` array.
* To a `node input` from a node `output` by specifying a `node` value which is an index into the graph's `nodes` array.
* To a nodegraph `output` from a node `output` by specifying a `node` value which is an index into the graph's `nodes` array.

If the upstream node has multiple outputs, then an `output` value which is an index into the the upstream nodes `outputs` array  __must__ additionally be specified.


The following example shows the basic audio data, audio source and global emitter nodes


<details>
<summary>Example</summary>

```JSON
{
    "audio_nodes" [
        {
            "audionodetype": "audiodata",
            "type": "stream",
            "value" : [
              "uri" : "urltotheasset"
            ]
        },
        {
            "audionodetype": "source",
            "type": "stream",
            "value" : [
                "id": 0,
                "data" : 0,
                "autoPlay" : "true"
            ]
        },
        {
            "audionodetype": "emitter",
            "type": "stream",
            "value" : [
                "id": 1,
                "emitterType" : "global",
            ]
        },
    ],
  "procedurals" [
    {
      "name": "nodegraph1",
      "nodetype": "nodegraph",
      "type": "stream",
      "inputs": [
      ],
      "nodes" [
          {
            "name": "audiosource1",
            "nodetype": "audionode",
            "type": "stream",
            "audionode" : 0,
            "inputs": [
              {
                "name": "audiodata1",
                "nodetype": "audiodata",
                "type": "stream",
                "audiodata": 0
              }]              ,
            "outputs": [
              {
                "name": "out",
                "nodetype": "output",
                "type": "stream"
              }
            ]
          },
          {
            "name": "emitter1",
            "nodetype": "audionode",
            "audionode" : 1,
            "type": "stream",
            "inputs": [
              {
                "name": "in2",
                "nodetype": "input",
                "type": "stream",
                "node": 0,
                "output": 0
              }
            ],
            "outputs": [
              {
                "name": "out",
                "nodetype": "output",
                "type": "stream"
              }
            ]
          }
        ],
        "outputs" [
          {
            "name": "graph_out",
            "nodetype": "output",
            "type": "stream",
            "node": 1,
            "output": 0
          },
        ]
    }
}
```
</details>

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


### 6.3 Pitch shifter node (1 input / 1 output)

Use the node to make the pitch of an audio deeper or higher.


|   |Type|Description|Required|
|---|---|---|---|
|**id**|'integer'|A unique identifier of the gain node in the scene.|Yes|
|**pitch**|'number'|Pitch shift in musical semitones. A value of -12 halves the pitch, while 12 doubles the pitch. A value of 0 will not change the pitch of the audio source.|No|
|**channelInterpretation**|'string'|Channel ordering for speaker channel interpretation are captured <a href="https://webaudio.github.io/web-audio-api/#ChannelOrdering">here</a>. Expected values are "speakers" or "discrete". When the number of channels do not match any of the basic speaker layouts, use
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

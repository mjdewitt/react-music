[![Maintenance Status][maintenance-image]](#maintenance-status)


<h1 align="center">react-music</h1>

<h4 align="center">
  Make music with React!
</h4>

***

![http://i.imgur.com/2t1NPJy.png](http://i.imgur.com/2t1NPJy.png)

## Contents
<!-- MarkdownTOC depth=3 autolink=true bracket=round -->

- [Install](#install)
- [Get Started](#get-started)
- [Basic Concepts](#basic-concepts)
- [Instruments](#instruments)
- [Effects](#effects)
  - [Effect Busses](#effect-busses)
- [LFO](#lfo)
- [API](#api)
  - [Top Level](#top-level)
  - [Instruments](#instruments)
  - [Effects](#effects-1)
  - [Special](#special)
- [Known Issues & Roadmap](#known-issues--roadmap)
- [License](#license)

<!-- /MarkdownTOC -->


## Install

`npm install react-music`

## Get Started

The easiest way to get started is to clone this repo and run `npm start`. The demo song will be running at [http://localhost:3000](http://localhost:3000). You can open up the `/demo/index.js` file and edit your song there, using the API below as reference.

That said, you can import the primitives yourself and run your own build setup if you want.

## Basic Concepts

#### Song

The first thing you want to do is create a `Song` component. This is the controller for your entire beat. It takes a `tempo` prop where you specify a BPM, and an `playing` prop that configures whether the song should play right away, or wait to press the play button. Set up it like so:

```js
<Song tempo={90}>

</Song>
```

#### Sequencer


Your `Sequencer`'s are what you use to define a looping section. They take two props. The first `resolution` is the resolution of steps in your sequence array. This defaults to `16`, which is a sixteenth note. The second is `bars` which is how many bars the sequencer sequences before it loops. You can have multiple sequencers in your song, and the main Song loop is based upon the sequencer with the largest number of bars. Here is an example:

```js
<Song tempo={90}>
  <Sequencer resolution={16} bars={1}>

  </Sequencer>
</Song>
```

Once you have a `Song` and a `Sequencer` component, you can add instruments to your `Sequencer`. Lets take a look at how these work:

## Instruments

#### Sampler

The sampler component is used to play audio samples. To use it, you must at very least provide two props, `sample` and `steps`.`sample` is a path to an audio file, and `steps` is an array of indexes that map to the steps available based upon the `resolution` and `bars` props of your sequencer. So if you wanted a 4/4 kick line, you would do this:

```js
<Song tempo={90}>
  <Sequencer resolution={16} bars={1}>
    <Sampler
	  sample="/samples/kick.wav"
	  steps={[0, 4, 8, 12]}
    />
  </Sequencer>
</Song>
```

You can also provide an array for a step, where the second value is a tuning, from -12 to 12.

#### Synth

The `Synth` component is used to create an oscillator and play it on steps, just like the `Sampler` does. To use it, you must provide two props, `type` and `steps`. Valid types are `sine`, `square`, `triangle` and `sawtooth`. The `Synth` component also takes an `envelope` prop, where you can specify your ASDR settings. The shape of the `step` prop is a bit different for the `Synth` component, as you must specify an array in the format of `[ step, duration, note || [notes] ]`. The `duration` portion specifies duration in steps. The `note` portion is a string of a musical note and octave like "a4" or "c#1", and for chords, can be an array of the same notes. This would look like:

```js
<Song tempo={90}>
  <Sequencer resolution={16} bars={1}>
    <Synth
      type="square"
	  steps={[
	    [0, 2, "c3"],
	    [8, 2, ["c3", "d#3", "f4"]]
	  ]}
    />
  </Sequencer>
</Song>
```

#### Monosynth

The `Monosynth` component is a `Synth` component, but it only plays one note at a time. It also has a `glide` prop that specifies portamento length. So if two notes overlap, the monosynth glides up to the next value on that duration. Check out how:

```js
<Song tempo={90}>
  <Sequencer resolution={16} bars={1}>
    <Monosynth
      glide={0.5}
      type="square"
      steps={[
        [0, 5, "c3"],
        [4, 4, "c4"],
      ]}
    />
  </Sequencer>
</Song>
```

## Effects

There are a ton of new effects added in 1.0.0. You can compose effect chains by wrapping effects around your instruments. Here is an example of how you would do that:

```js
<Song tempo={90}>
  <Sequencer resolution={16} bars={1}>
    <Reverb>
      <Delay>
        <Monosynth
          steps={[
            [0, 4, "c3"],
            [4, 4, "c4"],
          ]}
        />
      </Delay>
    </Reverb>
  </Sequencer>
</Song>
```

### Effect Busses

If you want to define an effects bus, which is a set of effects that multiple instruments can send their output to, this is achieved with the `Bus` component.

First you want to create a `Bus` component, and give it an identifier:

```js
<Song tempo={90}>
  <Bus id="myBus"/>
</Song>
```

Next, wrap your bus with the effect chain you want to make available, similarly to the way you would wrap effects around an instrument. You generally want to do this with effects that have wet/dry control, and set the `dryLevel` to 0:

```js
<Song tempo={90}>
  <Delay dryLevel={0}>
    <Bus id="myBus"/>
  </Delay>
</Song>
```

Finally, to hook an instrument up to your bus, or several busses, add their id's to the `busses` prop on an instrument:

```js
<Song tempo={90}>
  <Delay dryLevel={0}>
    <Bus id="myBus"/>
  </Delay>
  <Sampler
  	busses={['myBus']}
  	sample='/samples/kick.wav'
  	steps={[1,4,8,12]}
  />
</Song>
```

## LFO

You know whats bananas? LFO. Thats what. You can use an oscillator to modify properties of your instruments and effects. This is done with the `LFO` component. Any node that you want to apply LFO to just needs it added as a child. Then you define a `connect` prop that returns a function that lets you select a parent AudioNode property to oscillate. See the following example.

```js
<Song tempo={90}>
  <Synth
    type="square"
    steps={[
      [0, 2, "c3"],
      [8, 2, ["c3", "d#3", "f4"]]
    ]}
  >
    <LFO
      type="sine"
      frequency={0.05}
      connect={(c) => c.gain}
    />
  </Synth>
</Song>
```

## API

### Top Level

---

#### \<Song />
**playing** (_boolean_) : Whether the song should start playing automatically

**tempo** (_number_) : Your song tempo

```js
Props = {
  	children?: any;
  	playing?: boolean;
  	tempo: number;
	};
defaultProps = {
    	playing: false,
    	tempo: 90,
  	}
```	
---

#### \<Sequencer />

**bars** (_number_) : Number of bars in your sequence

**resolution** (_number_) : Step resolution for your sequence

```js
Props = {
  	bars?: number;
  	children?: any;
  	resolution?: number;
	};
defaultProps = {
    bars: 1,
    resolution: 16,
  };
```  
### Instruments

---

#### \<Monosynth />
**busses** (_array_) : An array of `Bus` id strings to send output to

**envelope** (_object_) : An object specifying envelope settings
 ```js
  envelope={{
  attack: 0.1,
  sustain: 0.3,
  decay: 20,
  release: 0.5
}}
```
**gain** (_number_) (0 - 1): A number specifying instrument gain

**glide** (_number_) : Portamento length for overlapping notes

**steps** (_array_) : Array of step arrays for the notes to be played at
 ```js
 steps={[
            [0, 4, "c3"],
            [4, 4, "c4"],
          ]}
```	  
**transpose** (_number_) : Positive or negative number for transposition of notes

**type** (_string_) : Oscillator type. Accepts `square`, `triangle`, `sawtooth` & `sine`

```js
Envelope = {
  	attack?: number;
  	decay?: number;
  	sustain?: number;
  	release?: number;
	};
OscillatorType = 'sine' | 'square' | 'sawtooth' | 'triangle';
Props = {
  	busses: Array<string>;
  	children: any;
  	envelope: Envelope;
  	gain?: number;
  	glide?: number;
  	steps: Array<any>;
  	transpose?: number;
  	type: OscillatorType;
	};
defaultProps = {
    	envelope: {
      	attack: 0.01,
      	decay: 0.2,
      	sustain: 0.2,
      	release: 0.2,
    	},
    gain: 0.5,
    glide: 0.1,
    transpose: 0,
  };	
```


--

#### \<Sampler />
**busses** (_array_) : An array of `Bus` id strings to send output to

**detune** (_number_) : A number (in cents) specifying instrument detune

**gain** (_number_) (0 - 1): A number specifying instrument gain

**sample** (_string_) : Path to an audio file

**steps** (_array_) : Array of step indexes for the sample to be played at. Accepts arrays for steps in order to provide a second argument for index based detune (in between -12 & 12).
 ```js
steps={[0 0, 4 0, 8 0, 12 -12]}
```
 ```js
Props = {
  	busses: Array<string>;
  	children?: any;
  	detune?: number;
  	gain?: number;
  	sample: string;
  	steps: Array<any>;
	};
defaultProps = {
    	detune: 0,
    	gain: 0.5,
  	};
```
--

#### \<Synth />
**busses** (_array_) : An array of `Bus` id strings to send output to

**envelope** (_object_) : An object specifying envelope settings

```js
envelope={{
  attack: 0.1,
  sustain: 0.3,
  decay: 20,
  release: 0.5
}}
```

**gain** (_number_) (float 0 - 1): A number specifying instrument gain

**steps** (_array_) : Array of step arrays for the notes to be played at. Accepts in array in the `[ step, duration, note || [notes] ]` format.

```js
// single note
steps={[
  [0, 2, "a2"]
]}

// chord
steps={[
  [0, 2, ["c2", "e2", "g2"]]
]}
```

**transpose** (_number_) : Positive or negative number for transposition of notes

**type** (_string_) : Oscillator type. Accepts `square`, `triangle`, `sawtooth` & `sine`

```js
Envelope = {
  	attack?: number;
  	decay?: number;
  	sustain?: number;
  	release?: number;
	};

OscillatorType = 'sine' | 'square' | 'sawtooth' | 'triangle';

Props = {
  	busses: Array<string>;
  	children: any;
  	envelope: Envelope;
  	gain?: number;
  	steps: Array<any>;
  	transpose?: number;
  	type: OscillatorType;
	};
defaultProps = {
      	envelope: {
      	attack: 0.01,
      	decay: 0.2,
      	sustain: 0.2,
      	release: 0.2,
    	},
    gain: 0.5,
    transpose: 0,
    };
  ```
### Effects

---

#### \<Bitcrusher />
**bits** (_number_) (int 1 - 16)

**bufferSize** (_number_) (int ex: 4096)

**normfreq** (_number_) (float 0 -1)

```js
propTypes = {
    bits: PropTypes.number,
    bufferSize: PropTypes.number,
    children: PropTypes.node,
    normfreq: PropTypes.number,
  };
defaultProps = {
    bits: 8,
    bufferSize: 256,
    normfreq: 0.1,
    };
  ```    
--

#### \<Chorus />
**bypass** (_number_)

**delay** (_number_)

**feedback** (_number_)

**rate** (_number_)
```js
Props = {
  bypass?: number;
  children?: any;
  delay?: number;
  feedback?: number;
  rate?: number;
};
defaultProps = {
    bypass: 0,
    delay: 0.0045,
    feedback: 0.2,
    rate: 1.5,
  };
  ``` 
--

#### \<Compressor />
**attack** (_number_)

**knee** (_number_)

**ratio** (_number_)

**release** (_number_)

**threshold** (_number_)
```js
Props = {
  attack?: number;
  children?: any;
  knee?: number;
  ratio?: number;
  release?: number;
  threshold?: number;
};
defaultProps = {
    attack: 0.003,
    knee: 32,
    ratio: 12,
    release: 0.25,
    threshold: -24,
  };
  ``` 
--

#### \<Delay />
**bypass** (_number_)

**cutoff** (_number_)

**delayTime** (_number_)

**dryLevel** (_number_)

**feedback** (_number_)

**wetLevel** (_number_)
```js
Props = {
  bypass?: number;
  children?: any;
  cutoff?: number;
  delayTime?: number;
  dryLevel?: number;
  feedback?: number;
  wetLevel?: number;
};
defaultProps = {
    bypass: 0,
    cutoff: 2000,
    delayTime: 150,
    dryLevel: 1,
    feedback: 0.45,
    wetLevel: 0.25,
  };
  ```
--

#### \<Filter />
**Q** (_number_)

**frequency** (_number_)

**gain** (_number_) (float 0 - 1)

**type** (_string_)
```js
type  'lowpass' | 'highpass' | 'bandpass' | 'lowshelf' | 'highshelf' | 'peaking' | 'notch' | 'allpass'; 
Props = {
	children?: any;
  	frequency?: number;
  	gain?: number;
 	type?: BiquadFilterType;
	};     
defaultProps = {
    	frequency: 2000,
    	gain: 0,
    	type: 'lowpass',
  	};
```
--

#### \<Gain />
**amount** (_number_) (float 0 - 1)
```js
Props = {
  amount?: number;
  children?: any;
};
defaultProps = {
    amount: 1.0,
  };
```
--

#### \<MoogFilter />
**bufferSize** (_number_) (int ex: 4096)

**cutoff** (_number_) (float 0 - 1)

**resonance** (_number_) (float 0 - 4)
```js
Props = {
  	bufferSize?: number;
  	children?: any;
  	cutoff?: number;
  	resonance?: number;
	};
defaultProps = {
    	bufferSize: 256,
    	cutoff: 0.065,
    	resonance: 3.5,
  	};
```

--

#### \<Overdrive />
**algorithmIndex** (_number_)

**bypass** (_number_)

**curveAmount** (_number_)

**drive** (_number_)

**outputGain** (_number_) (float 0 - 1)
```js
Props = {
  	algorithmIndex?: number;
  	bypass?: number;
  	children?: any;
  	curveAmount?: number;
  	drive?: number;
  	outputGain?: number;
	};
defaultProps = {
    	algorithmIndex: 0,
    	bypass: 0,
    	curveAmount: 1,
    	drive: 0.7,
    	outputGain: 0.5,
  	};	
```
--

#### \<PingPong />
**delayTimeLeft** (_number_)

**delayTimeRight** (_number_)

**feedback** (_number_)

**wetLevel** (_number_)
```js
Props = {
  	children?: any;
  	delayTimeLeft?: number;
  	delayTimeRight?: number;
  	feedback?: number;
  	wetLevel?: number;
	};
defaultProps = {
    	delayTimeLeft: 150,
    	delayTimeRight: 200,
    	feedback: 0.3,
    	wetLevel: 0.5,
  	};
```
--

#### \<Reverb />
**bypass** (_number_)

**dryLevel** (_number_)

**highCut** (_number_)

**impulse** (_string_)

**level** (_number_)

**lowCut** (_number_)

**wetLevel** (_number_)
```js
Props = {
 	 bypass?: number;
 	 children?: any;
 	 dryLevel?: number;
	 highCut?: number;
	 impulse?: string;
 	 level?: number;
 	 lowCut?: number;
 	 wetLevel?: number;
	};
defaultProps = {
    	bypass: 0,
    	dryLevel: 0.5,
    	highCut: 22050,
    	impulse: 'reverb/room.wav',
    	level: 1,
    	lowCut: 20,
    	wetLevel: 1,
  	};
```
### Special

---

#### \<Analyser />
**fftSize** (_number_) : FFT Size value

**onAudioProcess** (_function_) : Callback function with audio processing data

**smoothingTimeConstant** (_number_) : Smoothing time constant
```js
Props = {
  children?: any;
  fftSize?: number;
  onAudioProcess?: Function;
  smoothingTimeConstant?: number;
};
defaultProps = {
   fftSize: 128,
   onAudioProcess: () => {},
   smoothingTimeConstant: 0.3,
  };
```
--

#### \<Bus />
**gain** (_number_) (float 0 - 1) : A number specifying Bus gain

**id** (_string_) : Bus ID
```js
Props = {
  children?: any;
  gain?: number;
  id: string;
};
defaultProps = {
  gain: 0.5,
  };
```
--

#### \<LFO />
**connect** (_function_) : LFO property selection function

**frequency** (_number_) : LFO frequency

**gain** (_number_) (float 0 - 1) : A number specifying LFO gain

**type** (_string_) : Oscillator type. Accepts `square`, `triangle`, `sawtooth` & `sine`
```js
OscillatorType = 'sine' | 'square' | 'sawtooth' | 'triangle';
Props = {
  children?: any;
  connect: Function;
  frequency?: number;
  gain?: number;
  type?: OscillatorType;
  };
defaultProps = {
  connect: (node) => node.gain,
  frequency: 1,
  gain: 0.5,
  type: 'sine',
  }; 
```

## Known Issues & Roadmap

- Currently only the 4/4 time signature is supported
- `Synth` presets need to be added
- Record/Ouput audio file
- Optional working mixing board alongside viz
- Sampler sample maps


## License

[MIT License](http://opensource.org/licenses/MIT)

### Maintenance Status

**Archived:** This project is no longer maintained by Formidable. We are no longer responding to issues or pull requests unless they relate to security concerns. We encourage interested developers to fork this project and make it their own!

[maintenance-image]: https://img.shields.io/badge/maintenance-archived-red.svg

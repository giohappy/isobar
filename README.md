# Introduction

isobar is a Python library for expressing and constructing musical patterns, designed for use in algorithmic composition. It allows for concise construction, manipulation and transposition of sequences, supporting scalar operations on lazy patterns.

Output can be sent via MIDI, OSC or SocketIO, or written to a .mid file. Input (for interactive systems) can be taken via MIDI, and patterns can be read from a .mid file.

Timing can be synchronised with an external MIDI clock, or generated internally. For fluid control over timings, the internal clock can also be warped by patterns subclassing the `PWarp` class; for example, a `PWRallantando` can be used to generate a gradual, exponential decrease in tempo.

Classes are included for some fairly sophisticated operations. `PLSys` can be used to generate patterns based on the formal grammars of [Lindenmayer Systems](http://en.wikipedia.org/wiki/L-system); `PMarkov` generates first-order Markov chains, accompanied by `MarkovLearner` to build a statistical model from an input pattern (or MIDI input). Numerous pattern generators for chance operations are defined in [pattern/chance.py](isobar/pattern/chance.py).

Most of the major parts of isobar are subclasses of `Pattern`, which implement's Python's iterator protocol. The `next()` method is called to generate the subsequent item in a pattern, with the `StopIteration` exception raised when a pattern is exhausted. Builtins such as `list()`, `sorted()` and `itertools` can thus be used to process the output of a `Pattern`.

# Usage

```python
import isobar as iso

#------------------------------------------------------------------------
# Create a geometric series on a minor scale.
# PingPong plays the series forward then backward. PLoop loops forever.
#------------------------------------------------------------------------
arpeggio = iso.PSeries(0, 2, 6)
arpeggio = iso.PDegree(arpeggio, iso.Scale.minor) + 72
arpeggio = iso.PPingPong(arpeggio)
arpeggio = iso.PLoop(arpeggio)

#------------------------------------------------------------------------
# Create a velocity sequence, with emphasis every 4th note,
# plus a random walk to create gradual dynamic changes.
# Amplitudes are in the MIDI velocity range (0..127).
#------------------------------------------------------------------------
amplitude = iso.PSequence([50, 35, 25, 35]) + iso.PBrown(0, 1, -20, 20)

#------------------------------------------------------------------------
# A Timeline schedules events at a given BPM.
#------------------------------------------------------------------------
timeline = iso.Timeline(120)

#------------------------------------------------------------------------
# Schedule events, with properties mapped to the Pattern objects.
#------------------------------------------------------------------------
timeline.schedule({
    iso.EVENT_NOTE: arpeggio,
    iso.EVENT_DURATION: 0.25,
    iso.EVENT_AMPLITUDE: amplitude
})
```

## Examples

More examples are available in the [examples](examples) directory with this
distribution:

* [00.ex-hello-world.py](examples/00.ex-hello-world.py)
* [01.ex-basics.py](examples/01.ex-basics.py)
* [02.ex-subsequence.py](examples/02.ex-subsequence.py)
* [03.ex-euclidean.py](examples/03.ex-euclidean.py)
* [04.ex-permutations.py](examples/04.ex-permutations.py)
* [05.ex-piano-phase.py](examples/05.ex-piano-phase.py)
* [06.ex-walk.py](examples/06.ex-walk.py)
* [10.ex-lsystem-stochastic.py](examples/10.ex-lsystem-stochastic.py)
* [11.ex-lsystem-rhythm.py](examples/11.ex-lsystem-rhythm.py)
* [12.ex-lsystem-grapher.py](examples/12.ex-lsystem-grapher.py)
* [20.ex-markov-learner.py](examples/20.ex-markov-learner.py)
* [30.ex-midifile-write.py](examples/30.ex-midifile-write.py)

## Classes

Top-level classes:

* [Chord](isobar/chord.py)
* [Key](isobar/key.py)
* [Scale](isobar/scale.py)
* [Timeline](isobar/timeline.py)
* [Clock](isobar/clock.py)

I/O classes:

* [MIDI](isobar/io/midi.py)
* [MIDIFile](isobar/io/midifile.py)
* [OSC](isobar/io/osc.py)
* [SocketIO](isobar/io/socketio.py)

Pattern classes:

    CORE (core.py)
    Pattern          - Abstract superclass of all pattern generators.
    PConstant        - Pattern returning a fixed value
    PRef             - Pattern containing a reference to another pattern
    PDict            - Dict of patterns
    PDictKey         - Request a specified key from a dictionary.
    PIndex           - Request a specified index from an array.
    PConcat          - Concatenate the output of multiple sequences.
    PAdd             - Add elements of two patterns (shorthand: patternA + patternB)
    PSub             - Subtract elements of two patterns (shorthand: patternA - patternB)
    PMul             - Multiply elements of two patterns (shorthand: patternA * patternB)
    PDiv             - Divide elements of two patterns (shorthand: patternA / patternB)
    PFloorDiv        - Integer division of two patterns (shorthand: patternA // patternB)
    PMod             - Modulo elements of two patterns (shorthand: patternA % patternB)
    PPow             - One pattern to the power of another (shorthand: patternA ** patternB)
    PLShift          - Binary left-shift (shorthand: patternA << patternB)
    PRShift          - Binary right-shift (shorthand: patternA << patternB)

    SEQUENCE (sequence.py)
    PSequence        - Sequence of values based on an array
    PSeries          - Arithmetic series, beginning at <start>, increment by <step>
    PRange           - Similar to PSeries, but specify a max/step value.
    PGeom            - Geometric series, beginning at <start>, multiplied by <step>
    PLoop            - Repeats a finite <pattern> for <n> repeats.
    PConcatenate     - Concatenate the output of multiple finite sequences
    PPingPong        - Ping-pong input pattern back and forth N times.
    PCreep           - Loop <length>-note segment, progressing <creep> notes after <count> repeats.
    PStutter         - Play each note of <pattern> <count> times.
    PWrap            - Wrap input note values within <min>, <max>.
    PPermut          - Generate every permutation of <count> input items.
    PDegree          - Map scale index <degree> to MIDI notes in <scale>.
    PSubsequence     - Returns a finite subsequence of an input pattern.
    PImpulse         - Outputs a 1 every <period> events, otherwise 0.
    PReset           - Resets <pattern> each time it receives a zero-crossing from
    PCounter         - Increments a counter by 1 for each zero-crossing in <trigger>.
    PArp             - Arpeggiator.
    PEuclidean       - Generate Euclidean rhythms.
    PDecisionPoint   - Each time its pattern is exhausted, requests a new pattern by calling <fn>.

    CHANCE (chance.py)
    PWhite           - White noise between <min> and <max>.
    PBrown           - Brownian noise, beginning at <value>, step +/-<step>.
    PWalk            - Random walk around list.
    PChoice          - Random selection from <values>
    PWChoice         - Random selection from <values>, weighted by <weights>.
    PShuffle         - Shuffled list.
    PShuffleEvery    - Every <n> steps, take <n> values from <pattern> and reorder.
    PSkip            - Skip events with some probability, 1 - <play>.
    PFlipFlop        - flip a binary bit with some probability.
    PSwitchOne       - Capture <length> input values; repeat, switching two adjacent values <n> times.

    OPERATOR (operator.py)
    PChanged         - Outputs a 1 if the value of a pattern has changed.
    PDiff            - Outputs the difference between the current and previous values of an input pattern
    PAbs             - Absolute value of <input>
    PNorm            - Normalise <input> to [0..1].
    PCollapse        - Skip over any rests in <input>
    PNoRepeats       - Skip over repeated values in <input>
    PMap             - Apply an arbitrary function to an input pattern.
    PMapEnumerated   - Apply arbitrary function to input, passing a counter.
    PLinLin          - Map <input> from linear range [a,b] to linear range [c,d].
    PLinExp          - Map <input> from linear range [a,b] to exponential range [c,d].
    PRound           - Round <input> to N decimal places.
    PIndexOf         - Find index of items from <pattern> in <list>
    PPad             - Pad <pattern> with rests until it reaches length <length>.
    PPadToMultiple   - Pad <pattern> with rests until its length is divisible by <multiple>.

    STATIC (static.py)
    PStaticTimeline  - Returns the position (in beats) of the current timeline.
    PStaticGlobal    - Static global value identified by a string, with OSC listener

    FADE (fade.py)
    PFadeNotewise    - Fade a pattern in/out by introducing notes at a gradual rate.
    PFadeNotewise    - Fade a pattern in/out by gradually introducing random notes.

    MARKOV (markov.py)
    PMarkov          - First-order Markov chain.

    LSYSTEM (lsystem.py)
    PLSystem         - integer sequence derived from Lindenmayer systems

    WARP (warp.py)
    PWInterpolate    - Requests a new target warp value from <pattern> every <length> beats
    PWSine           - Sinosoidal warp, period <length> beats, amplitude +/-<amp>.
    PWRallantando    - Exponential deceleration to <amp> times the current tempo over <length> beats.


## Background

isobar was first designed for the generative sound installation [Variable 4](http://www.variable4.org.uk), in which it was used to generate musical structures in response to changing weather conditions. It was more recently used as the backbone of [The Listening Machine](http://www.thelisteningmachine.org/), taking live input from Twitter and generating musical output from language patterns, streamed live over the internet.

Many of the concepts behind Pattern and its subclasses are inspired by the excellent pattern library of the [SuperCollider](http://supercollider.sf.net) synthesis language.


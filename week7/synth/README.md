

Browser Synth
==============================================================================

![Browser Synth](https://github.com/stewdio/synth/raw/master/media/browser-synth.png "Browser Synth")

__TL;DR__: The demo is up at [http://stewd.io/synth](http://stewd.io/synth). 
Click the pulsing Play button for a 
[jaunty music lesson](http://en.wikipedia.org/wiki/Solf%C3%A8ge).
You can also mouse-over the triggers or use the keyboard keys labeled on each
trigger to play your own tunes.


Hackable
------------------------------------------------------------------------------
Browser Synth is a simple JavaScript framework for making browser-based music
using the WebAudio API. It takes a “batteries included” approach, meaning it 
boots up ready to give you the audio equivalent of “Hello, World!” without too 
much fuss. One line like `synth = new Instrument()` will build a bundle of 
`Trigger` interfaces, each with its own `Voices` for `Notes`; that is, a piano
keyboard that you can begin banging on immediately. But what’s a software 
piano that can’t play itself? Use `synth.scorePlay()` to play the default 
score provided for you. (And yes, you can always write your own scores!)  

The blurb above and descriptions below include some `sample code`. If you’re 
new to hacking around in the browser you may be wondering where that code’s
supposed to go. Well… Are you viewing this in a modern desktop browser? Then 
you can open up your browser’s __JavaScript Console__ and start hacking away 
right now! Here’s how:  

__[Chrome](http://google.com/chrome)__: View → Developer → JavaScript Console, or 
__⌥⌘J__.  
__[Safari](http://apple.com/safari)__: First, 
[enable the Developer menu](http://developer.apple.com/library/safari/#documentation/appleapplications/Conceptual/Safari_Developer_Guide/2SafariDeveloperTools/SafariDeveloperTools.html). 
Then, Develop → Show Error Console, or __⌥⌘C__.  
__[Firefox](http://mozilla.org/)__: Tools → Web Developer → Web Console, or 
__⌥⌘K__.  
__[Opera](http://opera.com/download)__: View → Developer Tools → Opera Dragonfly, 
or __⌥⌘I__, then click on the Console tab.  


Notes
------------------------------------------------------------------------------
Creating a new note is easy: `n = new Note()`. But unless you’re content with
nothing but concert A’s blaring at 440Hz all day, you’re going to want to 
create other notes like so: `new Note('E♭')` or `new Note('2E♭')` for an E♭ 
that’s in the 2nd octave rather than the default 4th octave. So what does that 
__2E♭__ give you anyway? An object like this:
```javascript
{  
    A: 440,           //  What Concert A are we tuned to?  
    hertz: 77.7817…,  //  Frequency of the note.  
    isFlat: true,     //  Set if ♭. Similar: isSharp and isNatural.  
    letter: "E",      //  Explains itself, no?  
    letterIndex: 4,   //  ['ABCDEFG'].indexOf(letter).  
    modifier: "♭",    //  Set to ♭, ♮, or ♯.   
    name: "E♭",       //  Note name. Will include ♮.   
    nameIndex: 7,     //  ['A♭','A♮','B♭','B♮','C♮'…].indexOf(name)  
    nameSimple: "E♭", //  Note name. Will NOT include ♮.  
    octaveIndex: 2,   //  On a standard piano, 0–8.  
    pianoKeyIndex: 19,//  On a standard piano, 0–87.  
    tuning: "EDO12"   //  Default: Equal Division of Octave into 12 steps.  
}
```

__Flexible parameters__  
Sure, you can call `new Note('E♭')` and accept the above default parameters
that come with it. But you can also send an Object to `Note` instead of a 
String and set each of those parameters manually! No specific param is 
required so just send what you need: 
```javascript
new Note({ A: 442, name: 'E♭', octaveIndex: 2 })
```

__By the numbers__  
Can we just throw all this named-note garbage out the window? Yes. Want the 
Devil’s note? Try `new Note(666)`. What does that give you? `{ hertz: 666 }`
I happen to like named notes though. They provide a pretty nice grid to work 
with, eh?

__Easy ASCII__  
It might quell your anxieties to know that `Note` will intelligently convert
the common __#__ (number) into a proper __♯__ (sharp) and will also accept a
lowercase __b__ as a substitue for __♭__ (flat). There’s no need to use __♮__ 
(natural) but it is in the code there should you desire to invoke it.  
  
__Smart conversion__  
If you commit a serious blunder like `new Note('B♯')` don’t stress, `Note` 
will kindly assume you intended `Note('C♮')` instead. 
([There is no B♯](http://en.wikipedia.org/wiki/Homer%27s_Barbershop_Quartet).)
If you happen to be old school German then, yes, you can use __H__ instead of 
__B__. (Similarly, there is no H♯. Weirdo.) 

__Western tunings__  
Right now only western tunings are supported—I’m afraid that’s all I know how 
to work with. All note params pass through `Note.validateWestern()` which does
the above fancy logic. From there I’ve included support for two separate 
tunings: __[Just intonation](http://en.wikipedia.org/wiki/Just_intonation)__ 
and __[Equal temperament](http://en.wikipedia.org/wiki/Equal_temperament)__.
More needs to be written on this topic for sure…

__Bach up a second__  
So all that’s great, but a `Note` is just a mathematical model. (You’ll notice 
there is no `Note.play()` method for example.) It doesn’t make any sound. For 
that we will need a `Voice`.


Voices
------------------------------------------------------------------------------
How do you make a `Note` sing? Give it a `Voice`. Or rather—create a `Voice`
initialized with a `Note` and maybe pass it an `AudioContext` to pipe the 
sound out to. Just as with `Note` the arguments for `Voice` are all optional.
Providing none will yield a `Voice` with a default `Note` of 440Hz:
```javascript
v = new Voice()//  We’re running with defaults.
v.play()//  Listen to that pure 440Hz Concert A.
v.stop()//  Ok, we’ve had enough.
```

__Note arguments__  
`Voice` will pass note-like arguments to `Note`. It doesn’t take an in-state
Liberal Arts degree to imagine what `new Voice('2E♭')` or 
`new Voice({ A: 442, name: 'E♭', octaveIndex: 2 })` might produce then. You 
could even try `new Voice(new Note('2E♭'))` if you’re not into that whole
brevity thing, man.

__Audio arguments__  
If you do not pass an `AudioContext` or `GainNode` to `Voice` it will create
an `AudioContext` for itself. This is convenient because it means `Voice` just
works (batteries-included, eh?) but there are hardware limits on the number of 
`AudioContexts` you can create. We’ll see how to solve this later by creating 
an `Instrument` and passing its `AudioContext` to each `Voice`.

__Only fix what’s Baroque__  
I guess all the above is pretty cool, but having to type `v.play()` and 
`v.stop()` everytime I want to `Voice` a `Note` is kind of a drag. And that’s 
where `Trigger` come in.


Triggers
------------------------------------------------------------------------------
We can dream up a `Note`, give it a `Voice`, but wouldn’t it be great if we 
had some visible DOM Elements and Event Listeners working on our behalf? 
Behold, your default Concert A: `t = new Trigger()`. Simply creating
a new `Trigger` will also construct the DOM bits and listeners for you.
No further fuss necessary.

__Notes & Voices__  
As you may have guessed, `Trigger` will create a `Voice` for you and assign it
a `Note`. Setting this at initialization time is trivial: `new Trigger('E♭')`.
See the `Voice` description above to get an idea of the variation possible 
here. And it’s likewise trivial to alter the `Note` or `Voice` after creation.

__Many Voices__  
Rather than one single voice, `Trigger` is setup to handle a whole Array of 
them. In fact, the default `Trigger` uses two voices: one employs a sine-wave
oscillator at the intended `Note` while a second employs a square-wave 
osciallator running one octave lower for a nice chunky Nintendo sound.
Customizing your instance of `Trigger.createVoices()` is the name of the game!

__Audio arguments__  
Just like `Voice`, `Trigger` is happy to ingest an `AudioContext` or 
`GainNode` argument but will make do without one if it has to. See the above 
`Voice` blurb for more details.

__Many Triggers__  
Throw a few of these together and you have a mini-keyboard. What famous 
movie theme does this keyboard play? Notice how we can optionally add 
keyboard event listeners to bind characters to `Triggers`? Here we’ve 
assigned the characters 1–5 to activate the five triggers respectively.
```javascript
new Trigger('4G').addTriggerChar('1')
new Trigger('4A').addTriggerChar('2')
new Trigger('4F').addTriggerChar('3')
new Trigger('3F').addTriggerChar('4')
new Trigger('4C').addTriggerChar('5')
```
What if we had a convenient way to bundle these `Triggers` together? You 
guessed it: `Instrument` to the rescue.


Instruments
------------------------------------------------------------------------------
How simple is this? `synth = new Instrument()`. You can pass the constructor 
either a DOM Element or a String representing the ID of a DOM Element and it 
will target that for the build. Otherwise it will just create its own. That 
one command gives you a default keyboard of `Triggers` with `Voices` and so 
on. Pretty nifty, eh?

__Triggers__  
Sure, upon creation your instance of `Instrument` will run `build()` on 
itself, creating a default set of `Triggers`. But it is so easy to overwrite 
this function with your own custom keyboard. (You should do this!) There is a 
corresponding `unbuild()` method for removing all of its `Triggers`. And that 
movie-theme keyboard from above? It comes built-in as well:
```javascript
Instrument.prototype.buildCloseEncounters = function(){

    this.unbuild()
    .newTrigger( '4G', '1' )
    .newTrigger( '4A', '2' )
    .newTrigger( '4F', '3' )
    .newTrigger( '3F', '4' )
    .newTrigger( '4C', '5' )
    return this
}
```
The `Instrument.newTrigger()` convenience method creates a new `Trigger`, 
passes it the existing `AudioContext`, and adds keyboard Event Listeners. 
Oh, my!


Scores
------------------------------------------------------------------------------
`Instrument` comes with a built-in score that you might recognize as 
[Do Re Mi](http://en.wikipedia.org/wiki/Solf%C3%A8ge). In the demo you can 
click the pulsing Play button to run it. This is 
equivalent to `Instrument.scorePlay()` in code. Check out the source to see 
how we’re able to compose the melody and harmony separately and 
`Instrument.scoreLoad()` blends them together. 

__Composing__  
Scores are just Arrays ingested three entries at a time: 1. Delay time 
(relative to the previous command), 2. `Trigger` ID to engage, 3. Engagement 
duration. I find it’s easiest to write the durations in fractions like the 
musical notation they are replacing: ¼ = quarter note, ½ = half note, and so 
on. Here’s a sample from the default score:
```javascript
melody = [

    36/4, '4C',  6/4,//  Do[e]  
     6/4, '4D',  2/4,//  a  
     2/4, '4E',  5/4,//  deer  
     6/4, '4C',  2/4,//  A  
     2/4, '4E',  4/4,//  fe  
     4/4, '4C',  3/4,//  male  
     4/4, '4E',  4/4,//  deer  
     …
```


Further
------------------------------------------------------------------------------
In the future it might make more sense to separate Score into its own Class. 
The naming conventions could use some tightening. And there is definitely a 
need for more explanation (and a demo) related to the difference between 
__[Just intonation](http://en.wikipedia.org/wiki/Just_intonation)__ and 
__[Equal temperament](http://en.wikipedia.org/wiki/Equal_temperament)__.
It’s early days.







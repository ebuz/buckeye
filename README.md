
This repository has Python classes for working with the [Buckeye
Corpus](http://buckeyecorpus.osu.edu/). The classes facilitate iterating through
the corpus annotations. They provide cross-references between the `.words`,
`.phones`, and `.log` annotations, and can be used to extract sound clips from
the `.wav` files. The docstrings in `buckeye.py` and `containers.py` describe
how to use the classes in more detail.

### Speaker

A `Speaker` instance is created by pointing to one of the zipped speaker
archives available on the corpus website. These archives have names like
`s01.zip`, `s02.zip`, and `s03.zip`.


    import buckeye
    
    speaker = buckeye.Speaker('s01.zip')

This will open and process the annotations in each of the sub-archives inside
the speaker archive (the tracks, such as `s0101a` and `s0101b`). If an optional
`load_wavs` argument is set to `True` when creating a `Speaker` instance, the
wav files associated with each track will also be loaded into memory. Otherwise,
only the annotations are loaded.

Each `Speaker` instance has the speaker's code-name, sex, age, and interviewer
sex available as attributes.


    print speaker.name, speaker.sex, speaker.age, speaker.interviewer

    s01 f y f
    

The tracks can be accessed through the `tracks` attribute.


    print speaker.tracks

    [<buckeye.Track object at 0x0000000003F61DD8>, <buckeye.Track object at 0x000000000430C668>, <buckeye.Track object at 0x0000000004672BA8>, <buckeye.Track object at 0x0000000005B2E048>, <buckeye.Track object at 0x0000000005D607B8>]
    

The tracks can also be accessed by iterating through the `Speaker` instance.
There is more detail below about accessing the annotations under **Tracks**.


    for track in speaker:
        print track.name,

    s0101a s0101b s0102a s0102b s0103a
    

The `Corpus` class is a convenience for loading and iterating through all of the
speaker archives together. Put all forty speaker archives in one directory, such
as `speakers/`. Point a new `Corpus` instance to this folder.


    corpus = buckeye.Corpus('speakers/')

This instance can be iterated through to provide `Speaker` instances in order.


    for speaker in corpus:
        print speaker.name,

    s01 s02 s03 s04 s05 s06 s07 s08 s09 s10 s11 s12 s13 s14 s15 s16 s17 s18 s19 s20 s21 s22 s23 s24 s25 s26 s27 s28 s29 s30 s31 s32 s33 s34 s35 s36 s37 s38 s39 s40
    

In this case, `corpus` works like a generator, and can only be iterated through
once. If you need to be able to access the `Speaker` instances more than once,
set the optional `cache` argument to `True`.


    corpus_cached = buckeye.Corpus('speakers/', cache=True)
    
    for speaker in corpus_cached:
        print speaker.name,
       
    print
    
    for speaker in corpus_cached:
        print speaker.sex,

    s01 s02 s03 s04 s05 s06 s07 s08 s09 s10 s11 s12 s13 s14 s15 s16 s17 s18 s19 s20 s21 s22 s23 s24 s25 s26 s27 s28 s29 s30 s31 s32 s33 s34 s35 s36 s37 s38 s39 s40
    f f m f f m f f f m m f m f m f f f m f f m m m f f f m m m f m m m m m f m f m
    

If they are cached, the `Speaker` instances can also be accessed by slicing the
`speakers` attribute.


    speaker = corpus_cached.speakers[10]
    
    print speaker.name

    s11
    

If a `Corpus` instance is created with `load_wavs` set to `True`, this argument
will be passed to the `Speaker` instances that it creates. It may take a minute
or two to create a `Corpus` instance with both `cache` and `load_wavs` set to
`True`.

### Track

Each `Track` instance has `words`, `phones`, `log`, `txt`, and `wav` attributes
that contain the corpus data from one sub-archive.

The `txt` attribute holds a list of speaker turns without timestamps, read from
the `.txt` file in the track.

If `load_wavs` is `True`, the `wav` attribute stores the `.wav` file associated
with the track as a `Wave_read` instance, using the Python `wave` library. If
`load_wavs` is `False` (the default), the `Track` instance will not have a `wav`
attribute.

The `words`, `phones`, and `log` attributes store sequences of `Word`, `Phone`,
`LogEntry` instances, respectively, from the entries in the `.words`, `.phones`,
and `.log` files in the track.

#### Words

For example, the first five entries in the `.words` in the first track of the
first speaker can be retrieved like this:


    speaker = corpus_cached.speakers[0]
    
    track = speaker.tracks[0]
    
    five_words = track.words[:5]

Each entry is stored in a `Word` instance, which has attributes for each
annotation in the `.words` file (see the docstring for `Word` in
`containers.py`).


    word = track.words[4]
    
    print word.orthography, word.beg, word.end, word.phonemic, word.phonetic, word.pos, word.dur

    okay 32.216575 32.622045 ['ow', 'k', 'ey'] ['k', 'ay'] NN 0.40547
    

The `Word` instance also has references to the `Phone` instances that belong to
the word. When a `Track` instance is created, it calls an internal
`get_all_phones()` method to add these references to each word, based on the
word's timestamps and the timestamps in the track's `.phones` file.


    print word.phones

    [<containers.Phone object at 0x00000000071503C8>, <containers.Phone object at 0x0000000007150320>]
    


    for phone in word.phones:
        print phone.seg, phone.beg, phone.end, phone.dur

    k 32.216575 32.376593 0.160018
    ay 32.376593 32.622045 0.245452
    

#### Phones

The sequence of entries in the track's `.phones` file (such as `s0101a.phones`)
can also be accessed directly through the `Track` instance's `phones` attribute.
Here are the first five entries in the first `.phones` file.


    five_phones = track.phones[:5]
    
    for phone in five_phones:
        print phone.seg, phone.beg, phone.end, phone.dur

    {B_TRANS} 0.0 0.102385 0.102385
    SIL 0.102385 4.275744 4.173359
    NOISE 4.275744 8.513763 4.238019
    IVER 8.513763 32.216575 23.702812
    k 32.216575 32.376593 0.160018
    

#### Log

The list of entries in the `.log` file can be accessed the same way.


    log = track.log[0]
    
    print log.entry, log.beg, log.end

    <VOICE=modal> 0.0 61.142603
    

The `get_logs()` method of the `Track` class can be called to retrieve the log
entries that overlap with the given timestamps. For example, the log entries
that overlap with the interval from 60 seconds to 62 seconds can be found like
this:


    logs = track.get_logs(60.0, 62.0)
    
    for log in logs:
        print log.entry, log.beg, log.end

    <VOICE=modal> 0.0 61.142603
    <VOICE=creaky> 61.142603 61.397647
    <VOICE=modal> 61.397647 176.705681
    

#### Wavs

Sound clips can be extracted from each `Track` instance if it is created with
`load_wavs=True`.


    speaker = buckeye.Speaker('speakers/s01.zip', load_wavs=True)
    track = speaker[0]
    
    track.clip_wav('myclip.wav', 60.0, 62.0)

This will create a wav file in the current directory called `myclip.wav` which
contains the sound between 60 and 62 seconds in the track audio.

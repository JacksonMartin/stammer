# STAMMER 2
A version of STAMMER that allows you to manually give it an amount of time for each frame to take. 


## Instructions

STAMMER2 can be run from the command line with the command

```sh
python stammer2.py <carrier track> <modulator track> <output file> <frame length>
```

where `<carrier track>` is the path to an audio or video file that frames will be taken from (i.e. Steamed Hams in the above example), `<modulator track>` is the path to an audio or video file that will be reconstructed using the carrier track, `<output file>` is a path to file that will be written to. `<output file>` should have an audio or video file extension (such as `.wav`, `.mp3`, `.mp4`, etc), and <frame length> is the desired time of the audio divisions in seconds. 

### Pure Audio
When working with audio, you give a value in seconds (0.04 for 1/25th of a second, etc) and the audio will be will be divided up into sections of that size.

### Video
When working with videos, the number put in will be used to generate an integer number of frames. This will always round up, so if there are 30 frames per second, a value of 0.04 will round to 2 frames, or 0.0667 seconds. 



# STeAMed haM-ifiER (STAMMER)

If you re-arranged the frames of [Steamed Hams](https://www.youtube.com/watch?v=4jXEuIHY9ic) in the right order, could you produce the [FitnessGram Pacer Test](https://www.youtube.com/watch?v=Y82jDHRrswc)?

~video goes here~

STAMMER is a python utility for recreating any audio source with the frames of any other audio or video source.
For each frame in the audio source being recreated, STAMMER selects the frame in the source audio with the most similar spectrum, adjusts its volume as needed, and inserts that frame into the output. Despite its name, STAMMER works on any audio, not just Steamed Hams.

## Instructions

STAMMER requires python 3, and the following python libraries:

```
numpy
scipy
```

It also requires the command line utility `ffmpeg`.

STAMMER can be run from the command line with the command

```sh
python stammer.py <carrier track> <modulator track> <output file>
```

where `<carrier track>` is the path to an audio or video file that frames will be taken from (i.e. Steamed Hams in the above example), `<modulator track>` is the path to an audio or video file that will be reconstructed using the carrier track, and `<output file>` is a path to file that will be written to. `<output file>` should have an audio or video file extension (such as `.wav`, `.mp3`, `.mp4`, etc).

### Warnings
At the start of its operation, STAMMER creates a directory called `temp`, where it writes files to work. At the end of its operation, STAMMER will delete this directory. However, if the program is forcefully terminated or crashes during operation, this directory will still exist.

Another case where `temp` unexpectedly remains is when trying to run two instances of STAMMER on the same machine at once; the result is that they will race to overwrite eachother's files.

STAMMER isn't designed to cope with this, so STAMMER will only work when `temp` doesn't already exist. Otherwise, it will warn you to _delete the directory yourself._

There are plans to solve this in future versions of STAMMER by keeping necessary data in memory instead of on disk, and/or creating uniquely-named temporary folders.

## Why and How

For several years, making increasingly outlandish edits of the skit "Steamed Hams" from *The Simpsons* has been a [trend online](https://knowyourmeme.com/memes/steamed-hams). This project was primarily inspired by an edit of "Steamed Hams" in which [the frames were sorted by pitch](https://www.youtube.com/watch?v=iWFRKZek0FI). It really is worth a listen: the choppy untintelligible vocals, steadily rising in pitch, builds tension in a strange way. Even chopped to the granularity of a frame, the music stabs and dialogue from the skit is still recognizable.

I've messed around with audio in numpy enough that this got me thinking: what else could I do with these frames? The idea of turning "Steamed Hams" into another song by manipulating its frequencies isn't completely novel: [other videos](https://www.youtube.com/watch?v=QUb3stxzTpE) have done it using a [vocoder](https://www.youtube.com/watch?v=QUb3stxzTpE). Roughly speaking vocoder divides the carrier signal (the other song) into frequency bands, and then sets the amplitude of each band to the amplitude of the modulator signal ("Steamed Hams") at that point in time. The result is an amalgamation of the two signals, where the pitch of the carrier is "sung" through the timbre of the modulator.

Like the vocoder, STAMMER divides the carrier and modulator signal into frequency bands, and calculates the amplitude of each band. The algorithm is as follows:

Let the carrier $C$ be broken into frames $C_i, 0 \leq i < n_c$ for each of its frames, and the modulator $M$ be broken into frames $M_i, 0 \leq i < n_m$ for each of the frames. We replace each frame in $M$ with the frame of $C$ where the spectrum $c_i$ of $C_i$ is most similar to the spectrum $m_i$ of $M_i$. 

The spectrum can be calculated from the frame using the Fast Fourier Transform. This algorithm returns a spectrum with far higher resolution than we need for these purposes, especially at high frequencies. To better reflect the frequencies that the human ear is sensitive to, we average the spectra of nearby frequencies into "bins," using larger bins for higher frequencies. 

To give the algorithm more options, for a cleaner-sounding result, I decided to allow STAMMER to change the volume of carrier frames to match the volume of modulator frames. This isn't quite as involved as in a vocoder, where we change the volume of the frequencies in each bin independently. Instead, we are merely linearly scaling the spectrum vector. 

After normalizing, we want to minimize the angle between the two vectors, which maximizes their similarity. In other words, we find $c_j$ that maximizes the dot product $\frac{c_j}{|c_j|}\cdot\frac{m_i}{|m_i|}$. For future work, it may be worth looking at [Accuracy limitations and the measurement of errors in the stochastic simulation of chemically reacting systems](https://www.sciencedirect.com/science/article/pii/S0021999105003074?casa_token=-OjUKZ73eCwAAAAA:2-eLNPyv6uj7Kucqmt-ex8wKHWZQVh7A9vcFX1UKqP2QnGyovpxg4jFkxUMeAiuiuS_0be4zzQ) for more advanced ways of comparing histograms than merely averaging nearby frequency lines into bins and calculating the dot products.


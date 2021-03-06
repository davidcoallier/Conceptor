# Conceptor
Audio Analysis by Conceptor 

# Usage instruction for pattern recognition

```
import conceptor.recognition as recog

new_recogniser = recog.Recognizer()

new_recogniser.train(training_data)

results = new_recognizer.predict(test_data)

```


# Usage Example for speaker recognitions

See the IPython Notebook:
http://nbviewer.ipython.org/github/littleowen/Conceptor/blob/master/Speaker.ipynb



# Emotion and Tone Recognition

See my blog post for the documentation and usage instructions:
http://reservoir-conceptors.blogspot.kr/2015/07/emotion-recognition-and-tone.html



# Python Speaker Identification Module written for the RedHen Audio Analysis Pipeline
The source code for this module is in the folder called "speaker", The following changes are made compared with the previous version:

1. Shift from Python3 to Python2

2. Replaced GMM from Sklearn by GMM from PyCASP

3. Added functions to recognize features directly, so that it is ready for the shared features from the pipeline.

4. Return the log likelihood of each prediction so that one can make rejections on untrained classes, which is useful for searching speakers in audio files.

Gender identifier as a usage example:

```
import speaker.recognition as SR
Gender = SR.GMMRec() # Create a new recognizer

```

1. Training:

Note: It is highly recommneded to get the training features from audio signals that do not contain any silence, you can use the remove_silence function provided in the module:

```
import scipy.io.wavfile as wav
from speaker.silence import remove_silence
(samplerate, signal) = wav.read(audio_path)
clean_signal = remove_silence(samplerate, signal)
```


```
import numpy as np

# Here we use mfcc as the audio features, but in theory, other audio features should work as well, e.g. lpc
female_mfcc = np.array(get_female_mfcc()) # female_mfcc.shape = (N1, D); N1 vectors and D dimension
male_mfcc = np.array(get_male_mfcc()) # male_mfcc.shape = (N2, D);
Gender.enroll('Female', female_mfcc) # enroll the female audio features
Gender.enroll('Male', male_mfcc) # enroll the male audio features
Gender.train() # train the GMMs with PyCASP
Gender.dump('gender.model') # save the trained model into a file named "gender.model" for future use

```

2. Testing:

```
Gender = SR.GMMRec.load('gender.model') # this is not necessary if you just trained the model
test_mfcc = np.array(get_test_mfcc()) # test_mfcc.shape = (N3, D)
(result, log_lkld) = Gender.predict(test_mfcc) # predict the speaker, where result is the most porbabel speaker label, and log_lkld is the log likelihood for test_mfcc to be from the recognized speaker. 

```

See https://github.com/littleowen/Conceptor/blob/master/ModuleTest.ipynb for a concrete example.


# SpeakerID.py program for the RedHen pipeline

This program (SpeakerID.py) takes use of the speaker ID module from above and speaker diarization results to output .spk file that has consistent format with other RedHen output files.

Usage Example:

```
python SpeakerID.py <trained_model> <wav_file> <segmentation_file> <metainfo_file>

```

a. "trained_model" is a trained speaker GMM model, such as the "gender.model" generated in ModuleTest.ipynb by "Gender.dump('gender.model')";

b. "wav_file" is audio file in .wav format, only .wav is accepted to reduce the dependencies of this program, for .mp4 format, please convert it using ffmpeg by typing the following command in the terminal:

```
ffmpeg -i audio.mp4 -ab 160k -ac 2 -ar 44100 -vn audio.wav
```

c. "segmentation_file" is the output generated by the PyCasp's speaker diarization program, wich has a .rttm format. An example is provided in this reporsitory.

d. "metainfo_file" is a reference output file from the RedHen Lab that contains meta info of the video, such as the .seg files or .tpt files. Examples are provided in this reporsitory.  

As a result, this program generate a .spk file (an example is provided) which has the same format as that of the reference file provided, except that the data section is replaced by speaker id data, which uses the syntax

```
0:00:50|0:01:17|Person=Name|Log Likelihood=-19.6577659268

```
These are starting time| ending time| Speaker's Name| Log Likelihood, log likelihood indicates how likely is the true speaker to be the predicted speaker, you can use this information to filter out the untrained speakers and search for target speakers. One thing to notice is that this value is dependent on the trained model, which is indicated in the legend of the output file, like this:
```
SPK_01|2015-08-20 05:04|Source_Program=SpeakerID.py gender.model|Source_Person=He Xu
```
, where the Sounce_Program section also includes the GMM model used.







# NylonRNN
By Andreas Bugler

### For COMP_SCI 397 Computational Creativity with Bryan Pardo at Northwestern University

NylonRNN generates short pieces for classical guitar via an LSTM network. [code here](https://github.com/abugler/NylonRNN)

## Training dataset

The training dataset for this project was found on Reddit, specifically on r/datasets. Thanks to u/midi_man to collecting this dataset.  The reddit post can be found [here](https://www.reddit.com/r/datasets/comments/3akhxy/the_largest_midi_collection_on_the_internet/).

In this dataset, the "Classical_Guitar_classicalguitarmidi.com_MIDIRip" folder was used, and the final training dataset was filtered from the folder.  For a song to make it into the final training set, it has to have the following attributes:
 - No more than 250 bpm. The midi parsing library experienced trouble generating a piano roll for especially fast songs.
 - All notes had to be inside the range E2 and B5. E2 and B5 are the lowest and highest fretted notes on a standard classical guitar.
 - The shortest note of each song had to be either a 1/32nd note or a 1/16th note tripet.  More information about this can be found in the encoding. 
 - Songs with rare time signatures such as 17/8, 21/8, or 7/4 were removed.
 - The midi track must only consist of a single guitar.

In addition, every song with tempo changes where split into multiple tracks.  Any segments less than four measures were removed.

This results in 712 songs for training, filtered from a training dataset of 3,519.

## Encoding

Each midi segment is encoded to a matrix with shape (50, t)

Each column c in [0, t-1] represents a timestep, which is 1/24 of a quarter note.  This encoding allows us to encode both a 1/32nd note and a 1/16th note triplet. However, during decoding, if notes 1/24 of a beat are generated, they may show up as 1/64th notes in music notation software. 

Rows 0-43 of the matrix we call the piano roll. If a 1 exists in row n, then the midi pitch represented by the integer 40 + n is being played at this timestep. 0 if otherwise. 40 is E2, the lowest note of the guitar, and 83 is the highest note, B5. 

Rows 44-49 of the matrix we call the attack matrix. If a 1 exists in **row 44**, then the **lowest note** played during this timestep begins in this timestep.  Likewise, if a 1 exists in **row 45-49**, then the **2nd-6th lowest notes** played this timestep begins in this timestep. The rationale for the attack matrix is to differentiate between a note being held, and multiple notes being played in succession. 

An example may be found below:

Lets say we have a whole note, and four quarter notes.

![whole](src\whole_note.png) ![quarter](src\quarter_notes.png)

On the piano roll, both would be represented by 96 consecutive 1s in row 20. However how would we differentiate between the two examples? The whole note will be additionally represented by a 1 in the first row of the attack matrix when the whole note begins, while the quarter notes will be additionally represented by a 1 in the first row every 24 timesteps. 

In this chord:

![chord](src\wacky_chords.png)
The Attack matrix (simplified to one timestep equivalent to a quarter note) for this chord would be:

[[1, 1, 0, 0],

 [1, 0, 0, 0],

 [0, 0, 1, 0]]

## Network Architecture and Training

The Neural Network contains the following:
 - 3 LSTM layers consisting of 512 units each
 - Following the LSTM Layers, a fully connected layer with sigmoid activation

The network is then trained with 10000 epochs with mini-batches of 50. Each batch consists of 50 encoded matrices. Each column is forward-fed into the network, and the loss is measured between the network output and the next column.  The loss function is Binary Cross Entropy.  

The model architecture and training regimen is modeled after FolkRNN. You may find FolkRNN [here](https://folkrnn.org).

The model was implemented in PyTorch. 

## Results

Training is still taking place. Some preliminary results may be found below:

5000 Epochs trained on solely Aguado_12valses_Op1_No12.mid MIDI file
![experimental](src\experimental_track_12_11.png)
[Download](midi\experimental_1000_epochs_aguado.mid)

1000 Epochs trained on solely Aguado_12valses_Op1_No12.mid MIDI file
![cadence](src\cadence_12_9.png)
[Download](midi\experimental_5000_epochs_aguado.mid)




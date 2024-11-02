# imagemusick
Convert WAV to BMP, apply ImageMagick effect(s) and convert it back to WAV.

Obvious hack made with ChatGPT.

Will use lots of resources with minutes-long WAV's.

Created by me but found out it was already invented and played about:

https://www.youtube.com/watch?v=ZvLR4t_E9dI


Can yield to interesting and actually nice-sounding results!


## PREREQUISITES
- Bash with sox and imagemagick ("apt-get install" them) 
- "input.wav" file to be processed
- some way to retrieve resulting WAV and BMP files (scp or Filezilla maybe?) (if remote)

## BASH SCRIPT
```
#!/bin/bash

# Step 1: Extract the WAV header
dd if=input.wav of=wav_header.bin bs=1 count=44

# Step 2: Calculate size (dimensions) based on the input file size (minus header)
file_size=$(stat -c%s "input.wav")
data_size=$((file_size - 44))  # Subtract header size (44 bytes)
dimension=$(echo "scale=0; sqrt($data_size)"+1 | bc)  # Calculate the square root + 1

# Step 3: Convert WAV (audio data only) to BMP using calculated dimensions
dd if=input.wav bs=1 skip=44 | \
convert -size "${dimension}x${dimension}" -depth 8 gray:- audio.bmp

# Step 4: Apply ImageMagick effects provided via command line
convert audio.bmp "$1" audio_edited.bmp

# Step 5: Convert edited BMP back to raw audio data
convert audio_edited.bmp gray:- | \
sox -t raw -r 44100 -e unsigned-integer -b 8 -c 1 - audio_data.raw

# Step 6: Combine header with audio data to recreate a valid WAV file
cat wav_header.bin audio_data.raw > output.wav

# Cleanup intermediate files
rm wav_header.bin audio.bmp audio_data.raw

# Play output
aplay output.wav
```

## SOME COOL EXAMPLES

./do.sh "-evaluate cos 5"

./do.sh "-blur 0x6 -equalize"

./do.sh "-resize 10% -sample 1000% -equalize"


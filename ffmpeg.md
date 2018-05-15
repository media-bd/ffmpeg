### Video Resize
```sh
ffmpeg -i media.mp4 -vf scale=48:28 scaled-48x28.mp4
````

### Play File
```sh
ffplay scaled-48x28.mp4 -x 192 -y 54
```

### Extract Video File Portion
```sh
ffmpeg -i infile.mp4 -ss 1 -to 60 outfile.mp4 [Extract from 1 to 60 second]
```sh

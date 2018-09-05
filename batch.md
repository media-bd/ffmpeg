## Batch Convertion all files in a directory
```sh
for i in *.wmv;
  do name=`echo "${i%.*}"`;
  echo $name;
  ffmpeg -i "$i" "${name}.mp4";
done
notify-send -u normal -t 5000 "Batch Conversion Complete"
```

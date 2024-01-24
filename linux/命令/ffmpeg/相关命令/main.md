## main

```
ffmpeg -i digitalman.mp4 -i digitalman-alpha.mp4 -filter_complex [0:v][1:v]alphamerge -c:v qtrle digital-alpha-merge.mov

ffmpeg -i digitalman.mp4 -i digitalman-alpha.mp4 -filter_complex "[0:v][alpha]alphamerge" -c:v qtrle digital-alpha-merge2.mov
```


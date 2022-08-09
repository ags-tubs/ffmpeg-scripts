
# ffmpeg-scripts

Collection of ffmpeg scripts and command lines used by us in various settings.

## ffmpeg-uncompressed-ip-video
  * sdi video input via BMD DeckLink card
  * needs ffmpeg compiled with Decklink support
  * needs 10G NIC depending on chosen resolution & framerate
  * outputs uncompressed videofeed on second pc/monitor (other options possible)

### video-rx (start this first!)
```
ffplay -fs -f nut -i tcp://<ip-of-rx-pc>:7000?fifo_size=2000000\&listen
```

### video-tx
```
ffmpeg -raw_format yuv422p10 -format_code Hp50 -f decklink -i 'DeckLink HD Extreme 3D+' -map 0:1 -c:v copy -f nut tcp://<ip-of-rx-pc>:7000
```


## ffmpeg-scope for CCU-operators
  * sdi video input via BMD DeckLink card
  * needs ffmpeg compiled with Decklink support
  * outputs fullscreen view of original video, vectorscope and waveforms (tested on debian)
  * optimized for 16:10 Monitors (1920x1200)

```
ffmpeg -format_code Hi50 -f decklink -i 'DeckLink HD Extreme 3D+' -raw_format yuv420p -vf "format=pix_fmts=yuv420p,yadif,split=4[o][v][w][p]; [o]scale=1312:736[O]; [v]vectorscope=b=1:m=gray:g=green,scale=592:592,pad=608:736:16:72[V];[w]waveform=filter=lowpass:scale=ire:graticule=green:flags=numbers+dots,scale=1312:448,pad=1312:464:0:10[W]; [p]waveform=filter=lowpass:components=7:mode=column:scale=ire:graticule=green:flags=numbers+dots:display=stack,scale=592:448,pad=608:464:8:10[P], [O][V]hstack[st], [W][P]hstack[nd], [st][nd]vstack,format=pix_fmts=yuv420p" -window_fullscreen 1 -f sdl scope
```

![screenshot-ffmpeg-software-scope](src/screenhot-ffmpeg-scopes.png)

  * the yuv to rgb implementation in ffmpeg is not optimized and produces many late frames on our machine; a rgb waveform is possible with the following command

```
ffmpeg -format_code Hi50 -f decklink -i 'DeckLink HD Extreme 3D+' -raw_format yuv420p -vf "format=pix_fmts=yuv420p,yadif,split=4[o][v][w][p]; [o]scale=1312:736[O]; [v]vectorscope=b=1:m=gray:g=green,scale=592:592,pad=608:736:16:72[V];[w]waveform=filter=lowpass:scale=ire:graticule=green:flags=numbers+dots,scale=1312:448,pad=1312:464:0:10[W]; [p]format=gbrp,waveform=filter=lowpass:components=7:mode=column:scale=ire:graticule=green:flags=numbers+dots:display=stack,scale=592:448,pad=608:464:8:10[P], [O][V]hstack[st], [W][P]hstack[nd], [st][nd]vstack,format=pix_fmts=yuv420p" -window_fullscreen 1 -f sdl scope
```
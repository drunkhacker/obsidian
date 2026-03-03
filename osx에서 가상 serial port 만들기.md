```
socat -d -d -d pty,raw pty,raw
```


이후에 `/dev/ttys020` `/dev/ttys021` 같은게 생기는데 screen을 이용하여 둘을 연결하면 됨.

`screen /dev/ttys021 19200`
`screen /dev/ttys021 19200`


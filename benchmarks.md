***LPMS-BENCH***

* To benchmark how long it takes to transcode one or more renditions, use the tool located here: [lpms-bench](https://github.com/mk-livepeer/lpms-bench)

* Clone the repository, `cd` into it, and build:

```bash
git clone https://github.com/mk-livepeer/lpms-bench
cd lpms-bench
go build .
```

* This benchmark takes an HLS stream as input.  An example command line that transcodes four renditions using one GPU is as follows:

```bash
./lpms-bench hls/bbb.m3u8 one/  4 P720p30fps16x9,P576p30fps16x9,P360p30fps16x9,P240p30fps16x9 nv 0
```

To see the expected command-line arguments, run `lpms-bench` alone:

```bash
lpms-bench
```

***STREAMTESTER***

* To benchmark real-time concurrent streaming using the streamtester utility, use the tool located here: [stream-tester](https://github.com/livepeer/stream-tester)

The `stream-tester` utility expects the B/O/T network to be running before invoking the benchmark.

* For example, to test streaming four renditions, run the following command:

```
./streamtester -host localhost -rtmp 1935 -media 8936 -profiles 2 -repeat 1 -sim 4 ~/official_test_source_2s_keys_24pfs.mp4
```

For more information about `stream-tester` see [README.md](https://github.com/livepeer/stream-tester/blob/master/README.md)

# adbreak3
Fast SCTE-35 Cue Creation.
adbreak3 Generates SCTE-35 Cues for Sidecar files.
```smalltalk
a@fu:~/x9k3$ adbreak3 -h
usage: adbreak3 [-h] [-d DURATION] [-e EVENT_ID] [-i] [-o] [-p PTS] [-P]
                [-s SIDECAR] [-v]

options:
  -h, --help            show this help message and exit
  -d DURATION, --duration DURATION
                        Set duration of ad break. [ default: 60.0 ]
  -e EVENT_ID, --event-id EVENT_ID
                        Set event id for ad break. [ default: 1 ]
  -i, --cue-in-only     Only make a cue-in SCTE-35 cue [ default: False ]
  -o, --cue-out-only    Only make a cue-out SCTE-35 cue [ default: False ]
  -p PTS, --pts PTS     Set start pts for ad break. Not setting pts will
                        generate a Splice Immediate CUE-OUT. [ default: 0.0 ]
  -P, --preroll         Add SCTE data four seconds before splice point. Used
                        with MPEGTS. [ default: False ]
  -s SIDECAR, --sidecar SIDECAR
                        Sidecar file of SCTE-35 (pts,cue) pairs. [ default:
                        sidecar.txt ]
  -v, --version         Show version



```

## Install 
1. `git clone https://github.com/superkabuki/adbreak3`
2. `python3 -mpip install threefive`
3.  `cd adbreak3`
4.  `install adbrerak3 /usr/local/bin`

## Use

* Running `adbreak3` with no args will generate a Splice Insert Cue, with `Splice Immediate` and `Out of Network` set. A CUE_OUT.
* The Cue data is displayed, and the base64 SCTE-35 string is written to sidecar.txt.
```json
a@fu:~$ adbreak3
{
    "info_section": {
        "table_id": "0xfc",
        "section_syntax_indicator": false,
        "private": false,
        "sap_type": "0x03",
        "sap_details": "No Sap Type",
        "section_length": 32,
        "protocol_version": 0,
        "encrypted_packet": false,
        "encryption_algorithm": 0,
        "pts_adjustment_ticks": 0,
        "pts_adjustment": 0.0,
        "cw_index": "0x00",
        "tier": "0x0fff",
        "splice_command_length": 15,
        "splice_command_type": 5,
        "descriptor_loop_length": 0,
        "crc": "0xc38e9258"
    },
    "command": {
        "command_length": 15,
        "command_type": 5,
        "name": "Splice Insert",
        "break_auto_return": true,
        "break_duration": 60.0,
        "break_duration_ticks": 5400000,
        "splice_event_id": 1,
        "splice_event_cancel_indicator": false,
        "out_of_network_indicator": true,
        "program_splice_flag": true,
        "duration_flag": true,
        "splice_immediate_flag": true,
        "event_id_compliance_flag": true,
        "unique_program_id": 1,
        "avail_num": 0,
        "avail_expected": 0
    },
    "descriptors": []
}
```
```lua
a@fu:~$ cat sidecar.txt

0.0,/DAgAAAAAAAAAP/wDwUAAAABf//+AFJlwAABAAAAAMOOklg=
```

# So? What do I do with a sidecar file?
### Use it to inject SCTE-35 into MPEGTS or HLS. 

### MPEGTS
* Use it with threefive's SuperKabuki packet injection engine

* generate a sidecar file
* If the sidecar file exists, new SCTE-35 data will be appended.
```sh
a@fu:~/x9k3$ adbreak3 -p 33.3 -d 45.4 -P

Writing to sidecar file: sidecar.txt

		CUE-OUT   PTS:33.3   Id:1   Duration: 45.4
		CUE-IN    PTS:78.7   Id:2

a@fu:~/x9k3$ adbreak3 -p 118.11 -d 15.98 -P

Writing to sidecar file: sidecar.txt

		CUE-OUT   PTS:118.11   Id:1   Duration: 15.98
		CUE-IN    PTS:134.09   Id:2

a@fu:~/x9k3$ 
```
* use [threefive](https://github.com/superkabuki/threefive) to add it to MPEGTS
```sh
a@fu:~/x9k3$ threefive inject  -i input.ts -s sidecar.txt -o output.ts

Output File:	output.ts

Inserted Cue:
	@34.766667, /DAlAAAAAAAAAP/wFAUAAAABf+/+AC27CP4APljwAAEAAAAAW6byOQ==

Inserted Cue:
	@79.366667, /DAgAAAAAAAAAP/wDwUAAAACf0/+AGwT+AACAAAAAOgvjTs=

Inserted Cue:
	@119.766667, /DAlAAAAAAAAAP/wFAUAAAABf+/+AKIzDP4AFfH4AAEAAAAAQ1KVCg==

Inserted Cue:
	@134.533333, /DAgAAAAAAAAAP/wDwUAAAACf0/+ALglBAACAAAAAGQRXqA=
a@fu:~/x9k3$ 
```
### HLS
* use [x9k3](https://github.com/superkabuki/x9k3) to add it to hls

* make a sidecar file
```py3
a@fu:~/x9k3$ adbreak3 -p 34 -d 60 -e 5

Writing to sidecar file: sidecar.txt

		CUE-OUT   PTS:34.0   Id:5   Duration: 60.0
		CUE-IN    PTS:94.0   Id:6

a@fu:~/x9k3$ adbreak3 -p 200 -d 30 -e 7

Writing to sidecar file: sidecar.txt

		CUE-OUT   PTS:200.0   Id:7   Duration: 30.0
		CUE-IN    PTS:230.0   Id:8

a@fu:~/x9k3$ adbreak3 -p 411.45 -d 56.346 -e 9

Writing to sidecar file: sidecar.txt

		CUE-OUT   PTS:411.45   Id:9   Duration: 56.346
		CUE-IN    PTS:467.796   Id:10

```

* call x9k3
```js
a@fu:~/x9k3$ x9k3 -i bk-30-60.ts -o output_dir -t 6 -s sidecar.txt
input = bk-30-60.ts
byterange = False
continue_m3u8 = False
delete = False
live = False
no_discontinuity = False
no_throttle = False
output_dir = output_dir
program_date_time = False
replay = False
sidecar_file = sidecar.txt
shulga = False
time = 6.0
hls_tag = x_cue
window_size = 5
version = False
loading  34.0,/DAlAAAAAAAAAP/wFAUAAAAFf+/+AC6xIP4AUmXAAAUAAAAA96thJQ==
loading  94.0,/DAgAAAAAAAAAP/wDwUAAAAGf0/+AIEW4AAGAAAAAF3ctkA=
loading  200.0,/DAlAAAAAAAAAP/wFAUAAAAHf+/+ARKogP4AKTLgAAcAAAAAsCdxSQ==
loading  230.0,/DAgAAAAAAAAAP/wDwUAAAAIf0/+ATvbYAAIAAAAAAe4Y1E=
loading  411.45,/DAlAAAAAAAAAP/wFAUAAAAJf+/+AjUKZP4ATWEkAAkAAAAAX0r8lQ==
loading  467.796,/DAgAAAAAAAAAP/wDwUAAAAKf0/+AoJriAAKAAAAAIB1qZc=
output_dir/seg0.ts:   start: 1.46667   end: 8.533333   duration: 7.06667
output_dir/seg1.ts:   start: 8.53333   end: 16.266667   duration: 7.73333
output_dir/seg2.ts:   start: 16.2667   end: 24.033333   duration: 7.7666


```
```js
a@fu:~/x9k3$ grep -2 'CUE' output_dir/index.m3u8
seg4.ts
#EXT-X-DISCONTINUITY
#EXT-X-CUE-OUT:60.0
#EXTINF:6.0,
seg5.ts
#EXT-X-CUE-OUT-CONT:6.000000/60.0
#EXTINF:6.0,
seg6.ts
#EXT-X-CUE-OUT-CONT:12.000000/60.0
#EXTINF:6.933333,
seg7.ts
#EXT-X-CUE-OUT-CONT:18.933333/60.0
#EXTINF:7.733333,
seg8.ts
#EXT-X-CUE-OUT-CONT:26.666666/60.0
#EXTINF:6.433334,
....
```


![image](https://github.com/futzu/adbreak2/assets/52701496/109a9e49-9aa0-43fa-8c97-3da12f105a33)

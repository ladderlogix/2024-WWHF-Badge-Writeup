---
tags:
  - writeups
---
----

# Preface

I want to thank everyone who put on the conference and make the badge. I had a blast and very thankful for my team.

# What is the Badge

The badge is powered by an esp32 chip that controls the display and various lights. There is also an nfc tag on the chip.

# How to Dump Firmware

One of the first steps to hacking the badge is dumping the firmware. One tool to do this is esptool.

## Install Esptool
```
pip install esptool
```

To dump the flash we can use the following command. We will need to supply a com port.

## Find COM Port

### Windows

Open Device manger by pressing Window's Key + X then clicking on device manger


### Linux

### Mac





## Getting Info on the chip

To better understand the chip infront of us we will use the flash_id command to further understand what type of chip is in front of us.

```bash
python -m esptool --port {COMPORT} flash_id
```

##  Dumping Firmware

Reading the [docs](https://docs.espressif.com/projects/esptool/en/latest/esp32/esptool/basic-commands.html#:~:text=esptool.py%20%2Dp%20PORT%20%2Db%20460800%20read_flash%200%20ALL%20flash_contents.bin) we will use the command read flash

```bash
python -m esptool --port {COMPORT} -b 460800 read_flash 0 ALL badge_flash.bin
```

# How to Analyze Firmware

I will be using [esp32 image parser](https://github.com/tenable/esp32_image_parser) to analyze, instead of [esp32knife](https://github.com/BlackVS/esp32knife) because I could not esp32knife to recognize my dump or get it to dump my firmware

## Installing esp32 image parser

### Download the repo
```bash
git clone https://github.com/tenable/esp32_image_parser
```
### Install Requirements
```bash
python -m pip install -r requiremnets.txt
```

## Showing the dump partitions

```bash
python esp32_image_parser.py show_partitions badge_flash.bin
```

### Annotated Output
```
└─$ python esp32_image_parser.py show_partitions badge_flash.bin
reading partition table...
entry 0:
  label      : nvs       -> Non-voliatle Storage
  offset     : 0x9000
  length     : 20480
  type       : 1 [DATA]
  sub type   : 2 [WIFI]

entry 1:
  label      : otadata
  offset     : 0xe000
  length     : 8192
  type       : 1 [DATA]
  sub type   : 0 [OTA]

entry 2:
  label      : app0       -> Core 0 Program
  offset     : 0x10000
  length     : 3342336
  type       : 0 [APP]
  sub type   : 16 [ota_0]

entry 3:
  label      : app1       -> Core 1 Program
  offset     : 0x340000
  length     : 3342336
  type       : 0 [APP]
  sub type   : 17 [ota_1]

entry 4:
  label      : spiffs     -> File Systems
  offset     : 0x670000
  length     : 1572864
  type       : 1 [DATA]
  sub type   : 130 [unknown]

entry 5:
  label      : coredump   -> Crash Dumps
  offset     : 0x7f0000
  length     : 65536
  type       : 1 [DATA]
  sub type   : 3 [unknown]

MD5sum:
467eb896e29d1aa938c557f4cdfc4c5b
Done
```

## Pulling Data Out of the Partitions
### Reading Non-Volatile Storage

Esp32 Image Parser can dump nvs data

```bash
python esp32_image_parser.py show_partitions badge_flash.bin
```

#### Truncated Output
```bash
└─$ python esp32_image_parser.py dump_nvs badge.bin -partition nvs
Dumping partition 'nvs' to nvs_out.bin
Page 0
  page state : FULL
  page seq no. : 0
  page version : 2
  crc 32 : 3115986308
  page entry state bitmap (decoded) : 222222222222222222222222202222222222222222222222222222222222222222222222222222222222222222222222222222222222202222222222222233
  Entry 0
  Bitmap State : Written
    Written Entry 0
      NS Index : 0
      Type : U8
      Span : 1
      ChunkIndex : 255
      Key : badge_config
      Data (U8) : 1

  Entry 1
  Bitmap State : Written
    Written Entry 1
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 3
      ChunkIndex : 255
      Key : ServerPath
      String :
        Size : 38
        Data : https://wwhf2024.s3.amazonaws.com/v2/

  Entry 4
  Bitmap State : Written
    Written Entry 4
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : FirmwareName
      String :
        Size : 13
        Data : wwhf2024.bin

  Entry 6
  Bitmap State : Written
    Written Entry 6
      NS Index : 1
          NS : badge_config
      Type : I32
      Span : 1
      ChunkIndex : 255
      Key : HardwareType
      Data (I32) : 4

  Entry 7
  Bitmap State : Written
    Written Entry 7
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : SSIDName
      String :
        Size : 12
        Data : WWHF Badges

  Entry 9
  Bitmap State : Written
    Written Entry 9
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : SSIDPassword
      String :
        Size : 18
        Data : 77HackTheEsp3too@

  Entry 11
  Bitmap State : Written
    Written Entry 11
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 3
      ChunkIndex : 255
      Key : MQTTBroker
      String :
        Size : 34
        Data : w6e1deed.ala.us-east-1.emqxsl.com

  Entry 14
  Bitmap State : Written
    Written Entry 14
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : MQTTBroadcast
      String :
        Size : 17
        Data : broadcast/action

  Entry 16
  Bitmap State : Written
    Written Entry 16
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : MQTTEvents
      String :
        Size : 14
        Data : events/device

  Entry 18
  Bitmap State : Written
    Written Entry 18
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : MQTTUsername
      String :
        Size : 7
        Data : badges

  Entry 20
  Bitmap State : Written
    Written Entry 20
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : MQTTPassword
      String :
        Size : 20
        Data : TPY_net1pvg*ywf.cjk

  Entry 22
  Bitmap State : Written
    Written Entry 22
      NS Index : 1
          NS : badge_config
      Type : I32
      Span : 1
      ChunkIndex : 255
      Key : MQTTPort
      Data (I32) : 8883

  Entry 23
  Bitmap State : Written
    Written Entry 23
      NS Index : 1
          NS : badge_config
      Type : STR
      Span : 2
      ChunkIndex : 255
      Key : DeviceName
      String :
        Size : 14
        Data : HaySupernova2
```

### Extracting App0/1 + Other Partitions
While not the most need is possible to pull out app0 using this program. This could be used to convert to an elf but at this time no program that I know can convert this with out errors.

```bash
└─$ python esp32_image_parser.py dump_partition badge_flash.bin -partition app0
Dumping partition 'app0' to app0_out.bin
```
# Reviewing Creds found in NVS

We see two big systems of interest in NVS

# Flags

## Flag 1
There is two ways to solve this either by finding the string in the dump or physical analyzing the badge
### Physical
1. Begin by powering your badge and then recording the LED sequences flashing on the UFO.
2. Transcribe the 8 LEDS to binary strings 
``` 
01010011 
01001001 
01000111 
01111011 
01100011 
00110100 
01101110 
01110100 
01011111 
01110011 
01110100 
00110000 
01110000 
01011111 
01110100 
01101000 
00110011 
01011111 
01110011 
00110001 
01100111 
01101110 
00110100 
01101100 
01111101 
```
4. Construct a [CyberChef recipe: FromBinary_8](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)&input=MDEwMTAwMTEKMDEwMDEwMDEKMDEwMDAxMTEKMDExMTEwMTEKMDExMDAwMTEKMDAxMTAxMDAKMDExMDExMTAKMDExMTAxMDAKMDEwMTExMTEKMDExMTAwMTEKMDExMTAxMDAKMDAxMTAwMDAKMDExMTAwMDAKMDEwMTExMTEKMDExMTAxMDAKMDExMDEwMDAKMDAxMTAwMTEKMDEwMTExMTEKMDExMTAwMTEKMDAxMTAwMDEKMDExMDAxMTEKMDExMDExMTAKMDAxMTAxMDAKMDExMDExMDAKMDExMTExMDE)

### Firmware Strings
I threw in the bin file found from the S3 bucket into a disassembler.

#### IDA Pro
This is probably the best tool to analyze the binary but with the high cost it may be unavaible to most. I searched for `{` in strings to hunt down the flag.
![[Pasted image 20241011115829.png]]

#### Ghidra
![[Pasted image 20241011120335.png]]
You will want to set your language to `Xtensa Little Endian`
![[Pasted image 20241011121721.png]]
It is still hard to pull out the string on this but if you scroll down you will see you string with a message

#### Binary Ninja
This is not the best for decomplation but strings are on point
![[Pasted image 20241011120206.png]]


## Flag 2


# Scoreboard Messing Around
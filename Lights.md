## Task 1

A company is using 46 moveable lights, but each is spinning wildly and uncontrollably. One of the lights, the light directly above a podium, is consistently more red than the others. An attached packet capture shows the ongoing state of the lights. The manufacturer's list of values and how they correspond to the light controls is also provided.

Start
Initially, look at the packet captures and assess what the protocol looks like. Opening up any packet on the packet capture looks like this:

<img width="1847" height="807" alt="Pasted image 20260917120450" src="https://github.com/user-attachments/assets/672327c5-955d-42ed-8dae-32f73b58c94c" />


This gives you the protocol, and what the protocol looks like on the wire. As you can see, there are many different values. If you continue looking at different packets, the values of the application (DMX Channels) continue to be random. Now, look at the attached manufacturer's specifiacations: 

<img width="661" height="856" alt="Pasted image 20260917092642" src="https://github.com/user-attachments/assets/5b565209-b4ee-4892-8959-7ffa1593d9bc" />

Looking at the specifications, there are 11 values, with the 6th value being red. After 11 values, the next value is destined for the next device. 

The next step is to look at the packet capture, and find a consistently very high value. In our case, when we look hard enough, we find a consistent 97% in one of the values, whereas every other value seems random. This is probably our culprit. Identify where the value is in the list, and you'll find that it should be the 281st value. 281/11 =~25.5. Since the decimals represent values in the light, 25 is the light number in the list.


## Task 2

Now, we're being asked to craft a whole new packet to get the lights under control: The company has provided the following specifications for the lighting configuration:
- The light over the podium should be set to white at full intensity and zoom, and pointed straight down (50% pan and 50% tilt)
- The remaining lights shoud be set to the CEO's favorite shade of blue-green, which can be approximated with 70% blue and 25% green, at half intensity, 10% zoom, and pointed oat the ceiling for indirect light (50% pan and 0% tilt).
- In order to keep the lights from resetting, the packets must be sent at no less than 10% hertz.
- Send the proper control traffic to 192.168.100.5:6454.

The shortuct way to solve this is to give an LLM the specifications, the manufacturer's instructions, and tell it package it up to send it to the address listed. Here is the manual breakdown:

We need to first translate each request from their byte or decimal values to their hex values to match the Wireshark capture. If we list out the requirements:

Podium:

| Spec                        | Hex Value |
| --------------------------- | --------- |
| 50% pan                     | 80        |
| No continuous pan movement  | 80        |
| No continuous tilt movement | 00        |
| No red                      | 00        |
| No Green                    | 00        |
| No blue                     | 00        |
| Full white                  | ff        |
| Open shutter (light on)     | 20-3f     |
| Dimmer Intensity            | ff        |
| Zoom                        | ff        |

All other lights:

| Spec                        | Hex value |
| --------------------------- | --------- |
| 50% pan                     | 80        |
| No continuous pan movement  | 00        |
| No continuous tilt movement | 00        |
| No red                      | 00        |
| Green: 25%                  | 42        |
| Blue: 70%                   | 70        |
| No white                    | 00        |
| Shutter open                | 20-3f     |
| Half intensity              | 80        |
| 10% zoom                    | 1c        |
|                             |           |

Once putting this into an LLM, we can produce a Python program with these values to send to 192.168.100.5:6454 via UDP. Keep in mind that we need our packets to look like what we've already seen--as in, we need both a header and footer. In Wireshark, if we highlight the application, we get all the hex values for the entire application: 

<img width="1828" height="558" alt="Pasted image 20260917102039" src="https://github.com/user-attachments/assets/b6079332-c01a-4eae-ab81-77f5521cd42f" />

If we highlight the actual bytes, we get the rest of the payload. The difference between them is how we can apply the header as hex values in Python. The footer is simply padding to finish out the last hex value (0x1f1): 00 00 00 00 00 00

You should be able to get the flag by using the following Python script:


`import socket`
`import time`

`HOST = "192.168.100.5"`
`PORT = 6454`
`REPETITIONS = 1000  # Adjust number of iterations as needed`
`DELAY = .05`

`header = bytes.fromhex("41 72 74 2d 4e 65 74 00 00 50 00 0e 00 00 00 00 02 00")`
`default_bytes = bytes.fromhex("80 00 00 00 00 42 b5 00 00 80 1c")`
`special_bytes = bytes.fromhex("80 80 00 00 00 00 00 ff 00 ff ff")`
`footer = bytes.fromhex("00 00 00 00 00 00")`

#`Construct the payload combining all 46 lights (25th light gets special hex)`
`full_payload = bytearray()`
`full_payload.extend(header)`
`for light_num in range(1, 47):`
    `if light_num == 25:`
        `full_payload.extend(special_bytes)`
    `else:`
        `full_payload.extend(default_bytes)`
`full_payload.extend(footer)`

`with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:`
    `s.sendto(bytes(full_payload), (HOST, PORT))`
    `print(f"Transmission sent for50`
    `iteration {i}")`
    `if i < REPETITIONS:`
        `time.sleep(DELAY)`



## Task 3

Now, the light is flickering, and you think the flickering may be being used to transmit data (You're given an MP4 file to assess white/black flickering). Analyze it and decode it.

The flickering could mean one of several things, but those familiar with Morse Code would recognize the patterns of the flickering (short blips, long blips, and pauses between characters). The best way to figure this out is to write down short/longblips, and the pauses and match them up with a Morse Code Decoder.

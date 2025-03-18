# **PicoCTF 2025 Write-Up**
> # Introduction
> The picoCTF 2025 Competition, organized by Carnegie Mellon University, was a cybersecurity competition where participants tried solving a designated number of problems within a specific timeframe. The competition took place from March 7, 2025 to March 17, 2025, and was available to anyone older than 13 years old with different leaderboards for Global, US Middle/High Schools, US Middle Schools, African HS/Undergraduate, and JP Students. [The Center of Excellence in Cybersecurity Research, Education, and Outreach](https://www.ncat.edu/research/centers/creo/index.php) (based out of North Carolina A&T State University in Greensboro, NC) had two participating teams. We are [Z3R0_D4Y](https://play.picoctf.org/teams/13242):
> 

|                    Member Name                     |                   Role                   |                  picoCTF 2025 Portfolio                  |
|:--------------------------------------------------:|:----------------------------------------:|:--------------------------------------------------------:|
| [Jaden Andrews](https://jadenandrews831.github.io) | Reverse Engineering, Binary Exploitation | [Portfolio](https://play.picoctf.org/participants/86339) |
|                   Elijah Flythe                    |             Web Exploitation             | [Portfolio](https://play.picoctf.org/participants/97291) |
|              Kaciopey Ikounga Moulolo              |       Cryptography, General Skills       | [Portfolio](https://play.picoctf.org/participants/86757) |
|                   Delvan Paulino                   |        Forensics, General Skills         | [Portfolio](https://play.picoctf.org/participants/86756) |


Z3R0_D4Y ended the competition with **2,310** with **19/41** challenges solved, putting us in the top **20%** of all participating teams. This write-up comprises of the CTF challenges, their points values, and detailed steps for solving each challenge. Solutions are organized based on their category and point value.



---
# **General Skills**


---
# **Forensics**


---
# **Cryptography**


---
# **Web Exploitation**


---
# **Reverse Engineering**
## Flag Hunters
**Points:** 75
**Solved By:** Jaden Andrews
**Difficulty:** Easy

### Solution:
![image](https://hackmd.io/_uploads/BJld-NvnJx.png)
1. In this challenge, you are given the source code for the server (lyric_reader.py), but the flag is hidden within a file called "flag.txt", which is not given. In order to solve the challenge, download the source code and create a test flag file with some recognizeable text:

```
$ echo "flag text goes here" > flag.txt
$ cat flag.txt
flag text goes here
```

2. Next, run the source code and observe the output:

```
$ python lyric_reader.py
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
Brute force the lock or craft that regex,
Flag on the horizon, what challenge is next?

We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd:
```

Song lyrics are printed to stdout, and the program waits for user input. Use a test string to see what happens:
```
...
Crowd: test

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
Hex and headers, carving out clues,
Resurrect the hidden, it's forensics we choose.
Disk dumps and packet dumps, follow the trail,
Buried deep in the noise, but we will prevail.

We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test

...

We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test

...

We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test

...
```
Observe that the test string is repeated within the output until execution of the program has concluded. 

3. To determine how this user input is being handled, inspect the source code:
> 3.1 - The flag is read from a file `flag.txt` into a variable `flag`:

```
# lyric_reader.py
...
5: # Read in flag from file
6: flag = open('flag.txt', 'r').read()
...
```
> `flag` is then added to the end of another string, `secret_intro`:
 ```
# lyric_reader.py
8:  secret_intro = \
9:  '''Pico warriors rising, puzzles laid bare,
10: Solving each challenge with precision and flair.
11: With unity and skill, flags we deliver,
12: The ether’s ours to conquer, '''\
13: + flag + '\n'
 ```
> Finally, this is all added to `song_flag_hunters` and passed to `reader(song, startLabel)`:
```
# lyric_reader.py
...
16: song_flag_hunters = secret_intro +\
17: '''
18: 
19: [REFRAIN]
20: We’re flag hunters in the ether, lighting up the grid,
21: No puzzle too dark, no challenge too hid.
22: With every exploit we trigger, every byte we decrypt,
23: We’re chasing that victory, and we’ll never quit.
24: CROWD (Singalong here!);
25: RETURN
26: 
27: [VERSE1]
28: Command line wizards, we’re starting it right,
29: Spawning shells in the terminal, hacking all night.
30: Scripts and searches, grep through the void,
...
34: 
35: REFRAIN;
36: 
37: Echoes in memory, packets in trace,
38: Digging through the remnants to uncover with haste.
...
43: 
44: REFRAIN;
...
132: reader(song_flag_hunters, '[VERSE1]')
'''
```
Notice, these song lyrics were printed to stdout without the `secret_intro` when the program was run:

> 3.2 - `song_flag_hunters` is passed to `reader(song, startLabel)` as `song`.
```
# lyric_reader.py
...
87: def reader(song, startLabel):
88:   lip = 0
89:   start = 0
90:   refrain = 0
91:   refrain_return = 0
92:   finished = False
93: 
94:   # Get list of lyric lines
95:   song_lines = song.splitlines()
96:
97: # Find startLabel, refrain and refrain return
98:   for i in range(0, len(song_lines)):
99:     if song_lines[i] == startLabel:        # '[VERSE1]' was passed as 'startLabel', which is why the output skips the 'secret_intro'
100:       start = i + 1
...
111: for line in song_lines[lip].split(';'):    # Every line of the song is further split if it contains a semicolon
...
117:     elif re.match(r"CROWD.*", line):
118:         crowd = input('Crowd: ')        # User input is requested
...
121:     elif re.match(r"RETURN [0-9]+", line):    # If the command is 'RETURN <number>', 
122:         lip = int(line.split()[1])            # The program starts reading 'song_flag_hunters' at line <number>
...
```

4. To print the `secret_intro`, give some user input that will cause the program to read it:
> 4.1 - First, use a test input, like `test;`, which has a semicolon at the end:
```
$ python lyric_reader.py
...
Crowd: test;
...
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test
...
Crowd: test
...
```
> Notice that the semicolon from the input has disappeared. This is because `;` is a control character, as the program will split the line wherever one is found and conduct different operations based on each section of the line it deliminates. 
> 
> 4.2 - Use an input with the operation `RETURN 0`, so the program knows to read from the beginning:
```
$ python lyric_reader.py
...

Crowd: test;RETURN 0
...
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test
Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, flag text goes here
...
```
The string value from the local `flag.txt` is found in the output, so a useful payload has been found.

5. Verify the solution by submitting it to the server. 
```
$ nc verbal-sleep.picoctf.net 59014
...
Crowd: test;RETURN 0

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
...
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
Crowd: test
Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, picoCTF{flag_was_found_here}
```

---
# **Binary Exploitation**
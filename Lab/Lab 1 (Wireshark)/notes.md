analyse hex sent (impt)

# 2.6
Application: http
Transport:  tcp
Network: ipv4
Data Link: ethernet II

# 2.7
dst MAC:  12:01:00:00:00:01
src MAC: c8:6e:08:67:4a:53

# 2.8
src IP: 10.134.192.92
dst IP: 146.190.62.39

# 2.9
src port: 34281 
dst port: 80

# 3.3
tcp hex dump:
85 e9 00 50 e8 70 d2 2c 0f c6 e7 e6 50 18 00 ff 9d 5d 00 00

source port: 85 e9
destination port: 00 50

seq number: 1 (relative sequence number)
seq number (raw): 3899707948
bytes: e8 70 d2 2c

ack number: 1 (relative ack number)
ack number (raw): 2646937734
bytes: 0f c6 e7 e6

header length: 20 bytes
bytes: 50

flags: 0x018 (PSH, ACK)
bytes: 50 18 (they highlighted the 50 from header length as well)

window: 255
bytes: 00 ff

checksum: 0x9d5d
bytes: 9d 5d

urgent pointer: 0
bytes: 00 00

# 3.4
ip hex dump:
45 00 01 a3 76 10 40 00 80 06 00 00 0a 86 c0 5c 92 be 3e 27

version: 4 
bytes: 5

header length: 20 bytes (5)
bytes: 5 

together: 45

differentiated services field: 0x00 (DSCP: C50, ECN: Not-ECT)
bytes: 00

total length: 419
bytes: 01 a3

identification: 0x7610 (30224)
bytes: 76 10

flags: 0x2 (dont fragment)
fragment offset: 0
bytes: 40

ttl: 128
bytes: 80

protocol: tcp (6)
bytes: 06

header checksum: 0x0000 (validation disabled)
bytes: 00 00

src address: 10.134.192.92
bytes: 0a 86 c0 5c

dst address: 146.190.62.39
bytes: 92 be 3e 27

# 3.5
ethernet (covered in lab 2b)
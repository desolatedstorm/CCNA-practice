https://subnetipv4.com
## calculating subnets (e.g 192.168.10.50/27)

### step 1: find the interesting octet
- **/0 to /8**: 1st octet
- **/9 to /16**: 2nd octet
- **/17 to /24**: 3rd octet
- **/25 to /32**: **4th octet** ← We are here (because 27 is between 25 and 32).

### step 2: calculate magic number
formula = $2^{remaining bits}$
subnet = /27
remaining bits = 32 - 27 = 5
magic number = $2^5$ = **32**

### step 3: find network address (start)
find the multiples of your magic number (32) until you get one thats just under (50)
so network address = 192.168.10.32

### step 4: find next subnet (ceiling)
take network address and add magic number: 32 + 32 = 64
next subnet = 192.168.10.64

### step 5: find broadcast
one number before next subnet
broadcast: 192.168.10.63

### step 6: first and last host
first host: network address + 1: 192.168.10.33
last host: broadcast - 1: 192.168.10.62

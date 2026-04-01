r1:
int g0/0/0
ip add 172.17.9.113 255.255.255.252
no shut
int g0/0/1
ip add 192.168.1.1 255.255.255.0
no shut

203.149.210.224/29 -> 203.149.210.225-203.149.210.230

static NAT:
ip route 0.0.0.0 0.0.0.0 172.17.9.114
ip nat inside source static 192.168.1.20 203.149.210.225
int g0/0/1
ip nat inside
int g0/0/0
ip nat outside

dynamic NAT:
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool my_pool 203.149.210.226 203.149.210.230 netmask 255.255.255.255.248
ip nat inside source list 1 pool my_pool

NAT/PAT pool overload:
no ip nat inside source list 1 pool my_pool
ip nat inside source list 1 pool my_pool overload
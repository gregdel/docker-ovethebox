# OverTheBox virtual lab

This tool is made for development purposes only.


## Requirements

* docker (1.12.0+)
* kvm (qemu-kvm libvirt-bin on debian)

## Run

### General idea

* Docker will create a bridge network
* Docker will create FAI boxes and clients on the same switch (bridge)
* kvm will provide full virtualisation of the overthebox, it will be linked to the same switch

### Steps

Note: you can use a custom prefix for your network:

```
sudo LAB_NETWORK_PREFIX=10.0 ./lab.sh
```

To get the 10.0.0.0/16 network


* Start modem1: ```sudo ./lab.sh --modem 1```
* Setup the wan IP on modem1: ```sudo ./lab.sh --wan modem1 _failover_ip_1_```
* Start the kvm: ```cd kvm && ./setup.sh```
* Once the otb has an IP, stop dnsmasq on modem1: ```docker exec modem1 supervisorctl stop dnsmasq```
* Start modem2: ```sudo ./lab.sh --modem 2```
* Setup the wan IP on modem2 ```sudo ./lab.sh --wan modem2 _failover_ip_2_```
* Force the otb to scan for a new network: ```pkill -USR1 udhcpc```

### Setup dnat

You can setup a dnat to access your OTB via ssh and https.

```
sudo ./lab.sh --dnat modem1 _local_ip_ _remote_ip_
```

### Setup tc - rate / latency

You can setup the latency and the rate of each modem using the ```setup_tc.sh``` script.

```
./setup_tc.sh modem2 1mbit 200ms
```

## Register your device using the API

### Create an app

Create an app on this page: https://eu.api.ovh.com/createApp/
Store the credentials in ~/.ovh.conf, here is the proper syntax: https://github.com/ovh/go-ovh#use-the-api-on-behalf-of-a-user

Launch the cli whith the ```generateCk``` option to create a consumer key. And add it to the the ```.ovh.conf``` file.

```
cd cli
go run *.go --generateCk
```

Get the list of you services:


```
go run *.go --listServices
```


Get the device_id from kvm:

```
uci show overthebox.me.device_id
```

Link the device with the service

```
go run *.go -deviceId YOUR_DEVICE_ID -serviceId YOUR_SERVICE_ID
```

Confirm the service on the device (from v0.4.24), run this in the kvm

```
overthebox_confirm_service
```

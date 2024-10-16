# TP-Link

## Description

TP-Link is a Singaporean electronics manufacturer of network equipment and smart home products. TP-Link products include high speed cable modems, mobile phones, ADSL, range extenders, routers, switches, IP cameras, power-line adapters, print servers, media converters, wireless adapters, power banks, USB hubs, smart home devices, and home robots.

## Directory

- [TP-Link](#tp-link)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Port Forwarding](#port-forwarding)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)

## References

- [TP-Link](https://en.wikipedia.org/wiki/TP-Link)

---

## Port Forwarding

### Description

This details how to set up port forwarding to route external traffic to a local service within the network.

### References

- [Port forwarding: how to set up virtual server on TP-Link wireless router?](https://www.tp-link.com/us/support/faq/1379)

### Steps

1. Log into the home router's web interface on a web browser using its IP address (i.e. `192.168.0.1`).

2. In the web interface, navigate to the **Advanced** section.

3. On the left hand side of the interface, expand the **NAT Forwarding** group and select the **Port Forwarding** menu item.

4. In the Port Forwarding view, click the **Add** button.

5. In the **Add a Port Forwarding Entry** form, configure the following:

   - Service Name: Set any suitable name that best describes the service or rule (i.e. `http`)
   - Device IP Address: Enter the private IP address of the server hosting the service (i.e. `192.168.0.106`)
   - External Port: Enter the port number you wish to allocate for the service externally (i.e. `8080`)
   - Internal Port: Enter the port number the service is listening on internally (i.e. `80`)
   - Protocol: Expand the dropdown and select the protocol the service requires (i.e. `TCP`)
   - Enable This Entry: Toggle or check this box to enable the port forwarding rule

    Click the **Save** button.

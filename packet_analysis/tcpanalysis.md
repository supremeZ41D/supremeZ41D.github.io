
# Google CA Server Traffic

## Introduction

This is an example of the test computer reaching out to a Google Cloud for the Certificate Authority Service. The computer ends up downloading a certificate and a Certificate Revocation List. 

- _File_: Privateca_NetskopeBWAN.pcapng
- _Conditions_: Recently powered-up computer with no user interaction. It has the Netskope agent installed and enabled.

## 3-Way Handshake

- **Client Side** (Mushaisa Comp):

![PrivateCA Client-side 3-Way Handshake](tcp_images/privateca_3wayClient.png)

- **Server Side** (Google CA Server):

![PrivateCA Server-side 3-Way Handshake](tcp_images/privateca_3wayServer.png)

## Initial Stats

- _Packets_: 280
- _Destination Port_: 80 (HTTP)
- _Total Time_: 91.3 seconds
- _TCP Conversation Completeness_: 31 (with data)
- _Bad TCP Packets_: 4/280.
	- _DUP ACKs_: 1
	- _TCP KeepAlives_: 3

## Analysis

-  The client sends an HTTP GET request to the Google CA Sever asking for a CA certificate.

![PrivateCA Get CAcert](tcp_images/privateca_get_cacert.png)

- Two server-side MSSs and 52 bytes later, the client was able to download the CA certificate.

![PrivateCA Certificate Download](tcp_images/privateca_cacrt_download.png)

- Then, the client sends an HTTP GET request to the Google CA Server asking for the Certificate Revocation List.

![PrivateCA Get CRL](tcp_images/privateca_get_crl.png)

- 0.5 seconds and a little under 200 almost-fullsize-server-side-MSSs later, the server manages to send the Certificate Revocation List.

![PrivateCA CRL Download](tcp_images/privateca_crl_download.png)

- Finally, some keep-alives are exchanged and the stream ends with graceful TCP FIN+ACKs.

![PrivateCA Stream End](tcp_images/privateca_stream_end.png)

port : HTTP port is usually 80 while HTTPS is 443.

In HTTP, data is transmitted as a raw data. no encryption is taking place. so there is nothing like data security while using HTTP.

for HTTP, first TCP connection is established then data (Raw data) sharing takes place.


In HTTPS, first TCP connection is established and then TLS handshaking takes place.

Refer  [this image](https://www.researchgate.net/figure/HTTPS-message-sequence-diagram-with-detailed-TLS-handshaking-steps_fig1_306187575) ,  [also this image](https://drive.google.com/file/d/1kuMQATx-4bAN8uoiYJCVShwZUZLLqppS/view?usp=drive_link) for complete TLS 1.2 handshaking cycle.

also refer [this video](https://youtu.be/ZkL10eoG1PY?si=heFFMXkIvccnxwQi) for thorough understanding of TLS 1.2.


Refer [this video](https://youtu.be/JA0vaIb4158?si=6OvrS_NF3C4asteN) for TLS 1.3 Handshaking and what are the changes from TLS 1.2.
# RoHC-in-NS-3
Implementation of Robust Header Compression for Network SImulator3

Introduction
In streaming applications, the overhead of IP, UDP and RTP is 40 bytes for IPv4 or 60 bytes for
IPv6. For VoIP, this corresponds to around 60% of the total amount of data sent. Such large
overheads may be tolerable in local wired links where capacity is often not an issue, but are
excessive for wide area networks and wireless systems where bandwidth is scarce.
RoHC compresses these 40 bytes or 60 bytes of overhead typically into only one or three bytes,
by placing a compressor before the link that has limited capacity, and a decompressor after that
link. The compressor converts the large overhead to only a few bytes, while the decompressor
does the opposite. The current implementation of NS3
does not support Robust Header
Compression at the Pdcp Layer.

Installation of RoHC library
We are currently using a free and open source library for RoHC https://rohclib.
org/ . Follow the
instruction given in the documentation for the installation of the library.





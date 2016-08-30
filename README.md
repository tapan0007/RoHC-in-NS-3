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

Patch the RoHC library with NS3
Open “ABSOLUTEPATHTONS3/ src/lte/wscript” and add the
dependencies for the RoHC library.
bld.env.append_value("CXXFLAGS", "I/usr/include")
bld.env.append_value("LINKFLAGS", ["L/usr/lib", "L/usr/lib/lrohc_decomp", "L/usr/lib/lrohc_comp","L/
usr/lib/lrohc_common"])
bld.env.append_value("LIB", ["rohc", "rohc_comp", "rohc_decomp", "rohc_common"])


Using the RoHC Library Functionalities

Include the RoHC header files.
Add the following header files in “ABSOLUTEPATHTONS3/src/lte/model/ltepdcp.h”

#include <rohc/rohc_comp.h>
#include <rohc/rohc_decomp.h>
#include <rohc/rohc.h>


Create a Compressor and Decompressor Object:
Declare the objects in “ABSOLUTEPATHTONS3/src/lte/model/ltepdcp.h” in “LtePdcp” Class.

struct rohc_comp *compressor;
struct rohc_decomp *decompressor;

Initialise the Constructor and Destructor with the RoHC functionalities :
In our case, we used the IPCompression
Profile Only . The Library also provides other
functionalities like TCP, UDP, RTP EPS Header compression etc.
Add the below code in the LtePdcp Contructor in the “ABSOLUTEPATHTONS3/src/lte/model/ltepdcp.cc” file.

static int gen_random_num(const struct rohc_comp *const comp, void *const user_context){
return rand();
}
compressor = rohc_comp_new2(ROHC_SMALL_CID, ROHC_SMALL_CID_MAX,
gen_random_num, NULL);
decompressor = rohc_alloc_decompressor(compressor);

Compress the IP Packet received at the PDCP layer from the sending side :
The RoHC compressor function expects a Packet of type “struct rohc_buf” which is
defined in the rohc header file. So create a rohc_buf packet corresponding to the Packet
received at the PDCP layer and compress the packet using
The following changes are made in DoTransmitPdu()

unsigned char ip_buff[BUFFER_SIZE];
struct rohc_buf ip_packet = rohc_buf_init_empty(ip_buff, BUFFER_SIZE);
unsigned char rohc_buffer[BUFFER_SIZE];
struct rohc_buf rohc_packet = rohc_buf_init_empty(rohc_buffer, BUFFER_SIZE);
struct iphdr * ip_header;
ip_header = (struct iphdr *) rohc_buf_data(ip_packet);
ip_header->version = 4;
ip_header -> ihl = 5;
ip_packet.len += ip_header->ihl * 4;
ip_header->tos = p_ip.GetTos();
ip_header ->tot_len = htons(p >GetSize());
ip_header ->id = p_ip.GetIdentification();
ip_header -> frag_off = p_ip.GetFragmentOffset();
ip_header -> ttl = (unsigned char)p_ip.GetTtl();
ip_header -> protocol = p_ip.GetProtocol();
ip_header -> saddr = htonl(p_ip.GetSource().Get());
ip_header -> daddr = htonl(p_ip.GetDestination().Get());
ip_header ->check = p_ip.GetChecksum();
sum = ip_checksum(m_tempbuff2, 20);
rohc_buf_append(&ip_packet, m_tempbuff1, p1 >
GetSize()); //m_tempbuff1 contains the payload
rohc_compress4(compressor, ip_packet, &rohc_packet);

Note : IP Checksum needs to be enabled. The RoHC library expects an IP packet with the
calculated Checksum. By default, IP checksum is disabled. Also, the below function can be used to
calculate the checksum externally.

uint16_t
ip_checksum(void* vdata,size_t length)
{
char* data=(char*)vdata;
uint32_t acc=0xffff;
size_t i;
for (i=0;i + 1 < length; i+=2)
{
uint16_t word;
memcpy(&word,data+i,2);
acc+=ntohs(word);
if (acc>0xffff)
{
acc=
0xffff;
}
}
if (length&1)
{
uint16_t word=0;
memcpy(&word,data+length1,1)
;
acc+=ntohs(word);
if (acc>0xffff)
{
acc=
0xffff;
}
}
return htons(~acc);
}

Send the Compressed packet to RLC Layer .
p = new Packet(buff, rohc_packet.len);
Here, buff is the byte stream for the compressed RoHC Packet.


Decompress the RoHC Packet at the PDCP layer from the receiving side.
The decompressor
function converts the received packet into byte array.
Decompress function is called to convert into the original IP packet.
The following changes are made in DoReceivePdu()

status = rohc_decompress3(decomp, rohc_packet, &ip_packet,
rcvd_feedback, feedback_send);

The IP packet we receive from the library is converted into NS3
packet by assigning
Individual fields.


p_ip.SetProtocol(ip_header >protocol);
p_ip.SetTtl(ip_header >ttl);
p_ip.SetTos(ip_header >tos);
p_ip.SetPayloadSize(20);
p_ip.SetFragmentOffset((uint16_t) ip_header >frag_off);
p_ip.SetDestination(Ipv4Address(ntohl(ip_header >daddr)));
p_ip.SetSource(Ipv4Address(ntohl(ip_header >saddr)));
p_ip.SetIdentification((uint16_t) ip_header >id);


Send the decompressed
IP Packet to the above IP Layer.
Ptr <Packet> p2 = new Packet(temp_ip_buff, x);
p = p2 -> Copy();
p -> AddHeader(p_ip);
uint8_t m_tempbuff1[BUFFER_SIZE];
p ->CopyData(m_tempbuff1, p -> GetSize());







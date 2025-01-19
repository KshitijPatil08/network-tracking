# network-tracking

Imports:
import dpkt
import socket
import pygeoip

dpkt: A Python library used for parsing network packet data. It allows us to extract information like IP addresses from packet capture (.pcap) files.
socket: A standard Python library used for networking. In this code, it is used to convert binary IP addresses into human-readable strings.
pygeoip: A library for handling GeoIP lookups. It uses the GeoLiteCity.dat database to retrieve geographical information (e.g., latitude, longitude) based on IP addresses.


GeoIP Initialization:
gi = pygeoip.GeoIP('GeoLiteCity.dat')
Purpose: Initializes a GeoIP object using the GeoLiteCity.dat file. This file contains geographical data for IP address lookups.
Usage: The gi object is used to retrieve location information (latitude, longitude, etc.) for given IP addresses.

Function retKML:
def retKML(dstip, srcip):
Purpose: Generates a KML (Keyhole Markup Language) snippet for mapping the connection between two IP addresses.

Parameters:
dstip: Destination IP address.
srcip: Source IP address.

GeoIP Lookups:
dst = gi.record_by_name(dstip)
src = gi.record_by_name('x.xxx.xxx.xxx')
gi.record_by_name: Retrieves geographical information (latitude, longitude, city, etc.) for the given IP address.
dst: Geographical data for the destination IP (dstip).
src: Geographical data for the source IP. The string 'x.xxx.xxx.xxx' is a placeholder that needs to be replaced with the actual source IP address.

Try Block for Geo Data Extraction:
try:
    dstlongitude = dst['longitude']
    dstlatitude = dst['latitude']
    srclongitude = src['longitude']
    srclatitude = src['latitude']
    
Purpose: Extracts longitude and latitude for both destination and source IPs from their respective GeoIP data.
dstlongitude & dstlatitude: Longitude and latitude of the destination IP.
srclongitude & srclatitude: Longitude and latitude of the source IP.

KML String Generation:
kml = (
    '<Placemark>\n'
    '<name>%s</name>\n'
    '<extrude>1</extrude>\n'
    '<tessellate>1</tessellate>\n'
    '<styleUrl>#transBluePoly</styleUrl>\n'
    '<LineString>\n'
    '<coordinates>%6f,%6f\n%6f,%6f</coordinates>\n'
    '</LineString>\n'
    '</Placemark>\n'
) % (dstip, dstlongitude, dstlatitude, srclongitude, srclatitude)

Purpose: Creates a KML <Placemark> entry representing the connection between the source and destination IPs.
<name>: Includes the destination IP (dstip) as the name.
<coordinates>: Specifies the coordinates (longitude, latitude) for the source and destination IPs, creating a line connecting the two.

Error Handling:
except:
    return ''
    
Purpose: If there is an error (e.g., invalid or missing GeoIP data), it returns an empty string instead of a KML entry.

Function: plotIPs

def plotIPs(pcap):

Purpose: Iterates through a .pcap file, extracts IP addresses, and generates KML data for each connection.
Parameter:
pcap: A parsed .pcap file object containing network packets.

KML Points Initialization:
kmlPts = ''
Purpose: A string to accumulate KML <Placemark> entries for all IP connections.

Packet Iteration:

for (ts, buf) in pcap:

Purpose: Iterates through each packet in the .pcap file.
ts: The timestamp of the packet.
buf: The raw packet data.

Packet Parsing:

dpkt.ethernet.Ethernet(buf): Parses the Ethernet frame from the raw packet data.
eth.data: Extracts the IP layer from the Ethernet frame.
socket.inet_ntoa: Converts the binary IP addresses (ip.src and ip.dst) into human-readable strings.

Generate KML for IP Connection:
retKML(dst, src): Generates the KML string for the connection between src and dst.
kmlPts = kmlPts + KML: Appends the generated KML to the cumulative kmlPts string.

Function: main

Open .pcap File
f = open('wire.pcap', 'rb'): The Wireshark capture Network traffic in this file.
pcap = dpkt.pcap.Reader(f)
Purpose: Opens the .pcap file (wire.pcap) in binary mode and initializes a dpkt.pcap.Reader object to parse it.

KML Header and Footer:

kmlheader = '<?xml version="1.0" encoding="UTF-8"?> \n<kml xmlns="http://www.opengis.net/kml/2.2">\n<Document>\n' \
'<Style id="transBluePoly">' \
'<LineStyle>' \
'<width>2.0</width>' \
'<color>#0000ff</color>' \
'</LineStyle>' \
'</Style>'
kmlfooter = '</Document>\n</kml>\n'

KML Header (kmlheader): Specifies the XML declaration, KML namespace, and a style definition for the lines connecting IPs.
<color>: Specifies the line color (#0000ff = blue).
<width>: Sets the line width to 2.0.
KML Footer (kmlfooter): Closes the KML <Document> and <kml> tags.
Generate Complete KML Document

kmldoc = kmlheader + plotIPs(pcap) + kmlfooter
Purpose: Combines the KML header, the generated placemarks (plotIPs output), and the footer into a complete KML document.
Output the KML.

print(kmldoc)
Purpose: Prints the complete KML document to the console. This can be redirected to a file for use in mapping tools (e.g., Google Earth).

if __name__ == '__main__':
    main()
Purpose: Ensures that the main() function runs only when the script is executed directly.

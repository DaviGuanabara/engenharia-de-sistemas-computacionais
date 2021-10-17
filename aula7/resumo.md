Início do segundo Bimestre do curso: Internet.

## Introdução


A Internet é uma rede mundial de computadores, que interconecta bilhões de dispositivos *end systems* (sistemas finais), ou *hosts* (anfitriões). Seu funcionamento é análogo ao


End systems are connected together by a network of communication links and
packet switches. We’ll see in Section 1.2 that there are many types of communication
links, which are made up of different types of physical media, including coaxial
cable, copper wire, optical fiber, and radio spectrum.

with the transmission rate of a link measured in bits/second

as packets



Packet-switched networks (which transport packets) are in many ways
similar to transportation networks of highways, roads, and intersections (which
transport vehicles). Consider, for example, a factory that needs to move a large
amount of cargo to some destination warehouse located thousands of kilometers
away. At the factory, the cargo is segmented and loaded into a fleet of trucks.
Each of the trucks then independently travels through the network of highways,
roads, and intersections to the destination warehouse. At the destination warehouse,
the cargo is unloaded and grouped with the rest of the cargo arriving
from the same shipment. Thus, in many ways, packets are analogous to trucks,
communication links are analogous to highways and roads, packet switches are
analogous to intersections, and end systems are analogous to buildings. Just as
a truck takes a path through the transportation network, a packet takes a path
through a computer network.
End systems access the Internet


Internet Service Providers (ISPs),



The Transmission
Control Protocol (TCP) and the Internet Protocol (IP) are two of the most important
protocols in the Internet.

Internet standards
are developed by the Internet Engineering Task Force (IETF) [IETF 2020]. The IETF
standards documents are called requests for comments (RFCs).

The applications are said to be distributed applications,
since they involve multiple end systems that exchange data with each other. Importantly






Now, because you are developing a distributed
Internet application, the programs running on the different end systems will
need to send data to each other. And here we get to a central issue—one that leads
to the alternative way of describing the Internet as a platform for applications. How
does one program running on one end system instruct the Internet to deliver data to
another program running on another end system?



Suppose Alice wants to send
a letter to Bob using the postal service. Alice, of course, can’t just write the letter
(the data) and drop the letter out her window. Instead, the postal service requires
that Alice put the letter in an envelope; write Bob’s full name, address, and zip
code in the center of the envelope; seal the envelope; put a stamp in the upperright-
hand corner of the envelope; and finally, drop the envelope into an official
postal service mailbox. Thus, the postal service has its own “postal service interface,”
or set of rules, that Alice must follow to have the postal service deliver her
letter to Bob. In a similar manner, the Internet has a socket interface that the program
sending data must follow to have the Internet deliver the data to the program
that will receive the data.
The postal service, of course, provides more than one service to its customers.
It provides express delivery, reception confirmation, ordinary use, and many
more services. In a similar manner, the Internet provides multiple services to its
applications.

computer networks.
Recall from the previous section that in computer networking jargon, the computers
and other devices connected to the Internet are often referred to as end systems.
They are referred to as end systems because they sit at the edge of the Internet,
as shown in Figure 1.3.


Hosts
are sometimes further divided into two categories: clients and servers.


Today, the two most prevalent types of broadband residential access are
digital subscriber line (DSL) and cable.


The residential telephone line carries both data and traditional telephone signals
simultaneously, which are encoded at different frequencies:
• A high-speed downstream channel, in the 50 kHz to 1 MHz band
• A medium-speed upstream channel, in the 4 kHz to 50 kHz band
• An ordinary two-way telephone channel, in the 0 to 4 kHz band




Because the downstream and upstream rates are different,
the access is said to be asymmetric


Engineers have expressly designed DSL
for short distances between the home and the CO; generally, if the residence is not
located within 5 to 10 miles of the CO, the residence must resort to an alternative
form of Internet access


While DSL makes use of the telco’s existing local telephone infrastructure,
cable Internet access makes use of the cable television company’s existing cable
television infrastructure.


fiber to the home (FTTH) [Fiber Broadband 2020]. As the
name suggests, the FTTH concept is simple—provide an optical fiber path from
the CO directly to the home. FTTH can potentially provide Internet access rates in
the gigabits per second range.



In the previous subsection, we gave an overview of some of the most important
network access technologies in the Internet. As we described these technologies,
we also indicated the physical media used. For example, we said that HFC uses a
combination of fiber cable and coaxial cable. We said that DSL and Ethernet use
copper wire. And we said that mobile access networks use the radio spectrum.



Thus our bit, when traveling from source to destination, passes
through a series of transmitter-receiver pairs. For each transmitter-receiver pair,
the bit is sent by propagating electromagnetic waves or optical pulses across a
physical medium.

Physical media fall into two categories: guided media and unguided
media. With guided media, the waves are guided along a solid medium, such as
a fiber-optic cable, a twisted-pair copper wire, or a coaxial cable. With unguided
media, the waves propagate in the atmosphere and in outer space, such as in a
wireless LAN or a digital satellite channel.



A communication satellite links two or more Earth-based microwave transmitter/
receivers, known as ground stations. The satellite receives transmissions on
one frequency band, regenerates the signal using a repeater (discussed below),
and transmits the signal on another frequency. Two types of satellites are used
in communications: geostationary satellites and low-earth orbiting (LEO)
satellites.
Geostationary satellites permanently remain above the same spot on Earth.
This stationary presence is achieved by placing the satellite in orbit at 36,000 kilometers
above Earth’s surface. This huge distance from ground station through
satellite back to ground station introduces a substantial signal propagation delay
of 280 milliseconds. Nevertheless, satellite links, which can operate at speeds of
hundreds of Mbps, are often used in areas without access to DSL or cable-based
Internet access.
LEO satellites are placed much closer to Earth and do not remain permanently
above one spot on Earth. They rotate around Earth (just as the Moon does) and may
communicate with each other, as well as with ground stations. To provide continuous
coverage to an area, many satellites need to be placed in orbit. There are currently
many low-altitude communication systems in development. LEO satellite technology
may be used for Internet access sometime in the future.

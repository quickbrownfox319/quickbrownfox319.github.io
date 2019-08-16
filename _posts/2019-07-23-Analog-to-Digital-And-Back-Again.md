### Overview
I've been playing a lot recently with WiFi and let me tell you, WiFi is freaking *weird*. It's safe to say that usually when we connect to hot spot "FREE COFFEE WIFI!!!" we don't normally think of all the intricacies of how it works on the protocol or RF level. This complex type of networking allows us to reliably connect our devices to each other and the internet as we know it without having to deal with crazy lines of wires everywhere. However, this also opens opportunities for abuse by malicious actors if we're not careful. I'm aiming to write a series of posts starting with this one providing an overview of wireless communication and networking, along with how to use Linux utilities and tools to sniff and craft our own packets, whether for good or evil.

### Analog to Digital and Back Again
Say you're in a convention filled with people, each trying to hold a conversation. If everyone is speaking their own language and directly addressing their conversation partner, it is fairly easy to communicate. Our brains are in this case converting the analog sound waves into words we understand, and our mouths are converting our thoughts into spoken words. However, if we continue to cram more people into this room, the noise makes it difficult to understand each other, and conversations start to bleed into each other. In this case, it makes more sense if people move into a larger room where they have space to talk.

How does this translate into wireless communication? Your computer is basically trying to have a conversation with another computer in a very crowded wireless space. When your wireless card transmits data, it packages it from the computer's language of 1's and 0's into a special electromagnetic encoding that prevents it from getting confused with the other electrical noise in the air in what signal processing people call modulation. The receiving wireless card catches these electromagnetic waves in the air and demodulates them back into 1's and 0's. The guidebook that defines the rules the computers follow in order to communicate is called the IEEE 802.11 standard. The letters following 802.11 such as 'a', 'b', and 'g' are amendments to the 802.11 standard that refine the definition of how devices communicate as newer technologies emerge.

The FCC has allowed devices (such as WiFi enabled ones) to communicate without a license in the frequency bands of 2.4-2.5GHZ, and 5.725-5.875GHz, also know as part of the Industrial, Scientific, and Medical (ISM) bands. If you look around your room, you can probably count at least two or three wireless devices communicating with each other over WiFi. As you can imagine, if all wireless devices talk at the same time over the same 2.4GHz band, it gets very noisy very quickly, causing packets to get dropped and your internet to stutter.
![2.4GHz vs 5.8GHz channels](https://cdn.reolink.com/images/blog/home-security/wifi-5-8ghz-channels.jpg)
The 5.8GHz band is a much wider space, and as a result allows more devices to communicate freely without as much fear of overlap and interruptions. It is also faster as the bandwidth, or how wide your signal can be, can be up to 40MHz compared to 20MHz in the 2.4GHz band. This means you can cram more data into a wider bandwidth, increasing how much you can send at that moment. DSSS and FHSS are more robust modulation schemes but are also slower whereas OFDM can carry more information but at a shorter range. Note: If you have ever sniff packets, you may notice that most beacon packets are sent using DSSS/FHSS in 2.4GHz because of the robustness. Most wireless routers and access points also have to use DSSS in 2.4GHz for legacy WiFi devices which can slow the entire network down.

### Further Reading
This post is a bit brief because there's a lot of information that can easily be strewn together. I would have liked to cover software defined radios as it's very relevant, but that deserves an entire post on it's own. Here are some good resources that probably explain everything I just said, but better.

https://www.tutorialspoint.com/wi-fi/index.htm

https://www.elttam.com.au/blog/intro-sdr-and-rf-analysis/

https://en.wikibooks.org/wiki/Communication_Systems/What_is_Modulation%3F

https://en.wikipedia.org/wiki/List_of_WLAN_channels

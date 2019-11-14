# Unifi UAP Pro

The Unifi UAP Pro is an 802.11a/b/g/n/ac dual-band access point capable of 450-1300 Mbps speeds.
I got it to put into my homelab, but I also wanted to see what was under the hood.

One of the first things I did when I received my Unifi UAP Pro was take it apart. The construction quality of Ubiquiti products is really impressive. The UAP Pro is held together by clips nested inside below the outer lip that you can find by prying at it with a spudger. You can use multiple plastic spudgers to get a toe-hold by jamming them in until you get the first clip unhooked. After that, it's pretty simple to pop the rest of the clips off. Below are high resolution, focus-stacked photos I took of the internal board.

![Top of Unifi UAP Pro board](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/ubiquiti-uap-pro-top.jpg)


Some things of interest here:

  * [QCA9880 802.11n 2.4GHz radio](https://www.qualcomm.com/news/releases/2012/06/05/qualcomm-atheros-launches-80211ac-networking-platform-drive-gigabit)
  * QCA9563 802.11ac 5.8GHz radio SoC, that creates a combined dual-band system with the QCA9880. 
  * QCA8334 L2 switch
  * [Winbond W971GG6SB-25](https://www.digikey.com/product-detail/en/winbond-electronics/W971GG6SB-25-TR/W971GG6SB-25CT-ND/8017314) SDRAM
  * What appears to be three unpopulated UART pins
  * Funky looking antennas


![Bottom of Unifi UAP Pro board](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/ubiquiti-uap-pro-bottom.jpg)


Most interesting thing (to me) is the big fat [Macronix MX25L12835FMI-10G](https://www.digikey.com/product-detail/en/macronix/MX25L12835FMI-10G/1092-1147-ND/4211592) flash chip, probably used to hold firmware.


If you want to reverse the firmware, you luckily don't need to dump the flash or intercept traffic as you can download it directly from [Unifi's website](https://www.ui.com/download/unifi/).

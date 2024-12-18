# The Cartridge Header

Each cartridge contains a header, located at the address range `$0100`—`$014F`.
The cartridge header provides the following information about the game itself and the hardware it expects to run on:

## 0100-0103 — Entry point

After displaying the Nintendo logo, the built-in [boot ROM](<#Power-Up Sequence>) jumps to the address `$0100`, which should then jump to the actual main program in the cartridge.
Most commercial games fill this 4-byte area with a [`nop` instruction](https://rgbds.gbdev.io/docs/v0.5.2/gbz80.7#NOP) followed by a [`jp $0150`](https://rgbds.gbdev.io/docs/v0.5.2/gbz80.7#JP_n16).

## 0104-0133 — Nintendo logo

This area contains a bitmap image that is displayed when the Game Boy is powered on.
It must match the following (hexadecimal) dump, otherwise [the boot ROM](<#Power-Up Sequence>) won't allow the game to run:

```
CE ED 66 66 CC 0D 00 0B 03 73 00 83 00 0C 00 0D
00 08 11 1F 88 89 00 0E DC CC 6E E6 DD DD D9 99
BB BB 67 63 6E 0E EC CC DD DC 99 9F BB B9 33 3E
```

The way the pixels are encoded is as follows: ([more visual aid](https://github.com/ISSOtm/gb-bootroms/blob/2dce25910043ce2ad1d1d3691436f2c7aabbda00/src/dmg.asm#L259-L269))

- The bytes `$0104`—`$011B` encode the top half of the logo while the bytes `$011C`–`$0133` encode the bottom half.
- For each half, each nibble encodes 4 pixels (the MSB corresponds to the leftmost pixel, the LSB to the rightmost); a pixel is lit if the corresponding bit is set.
- The 4-pixel "groups" are laid out top to bottom, left to right.
- Finally, the monochrome models upscale the entire thing by a factor of 2 (leading to somewhat chunky pixels).

The Game Boy's boot procedure [first displays the logo and then checks](<#Bypass>) that it matches the dump above.
If it doesn't, the boot ROM **locks itself up**.

The CGB and later models [only check the top half of the logo](Power_Up_Sequence.html?highlight=half#behavior) (the first `$18` bytes).

## 0134-0143 — Title

These bytes contain the title of the game in upper case ASCII.
If the title is less than 16 characters long, the remaining bytes should be padded with `$00`s.

Parts of this area actually have a different meaning on later cartridges, reducing the actual title size to 15 (`$0134`–`$0142`) or 11 (`$0134`–`$013E`) characters; see below.

## 013F-0142 — Manufacturer code

In older cartridges these bytes were part of the Title (see above).
In newer cartridges they contain a 4-character manufacturer code (in uppercase ASCII).
The purpose of the manufacturer code is unknown.

## 0143 — CGB flag

In older cartridges this byte was part of the Title (see above).
The CGB and later models interpret this byte to decide whether to enable Color mode ("CGB Mode") or to fall back to monochrome compatibility mode ("Non-CGB Mode").

Typical values are:

Value | Meaning
------|----------------------------------------------------------------------------------------------------
`$80` | The game supports CGB enhancements, but is backwards compatible with monochrome Game Boys
`$C0` | The game works on CGB only (the hardware ignores bit 6, so this really functions the same as `$80`)

Values with bit 7 and either bit 2 or 3 set will switch the Game Boy into a special non-CGB-mode called "PGB mode".

:::tip Research needed

The PGB mode is not well researched or documented yet.
Help is welcome!

:::

## 0144–0145 — New licensee code

This area contains a two-character ASCII "licensee code" indicating the game's publisher.
It is only meaningful if the [Old licensee](<#014B — Old licensee code>) is exactly `$33` (which is the case for essentially all games made after the SGB was released); otherwise, the old code must be considered.

All known new licensee codes:

Code | Publisher
-----|-------------------------
`00`  | None
`01`  | [Nintendo](https://en.wikipedia.org/wiki/Nintendo)
`02`  | [Rocket Games](https://en.wikipedia.org/wiki/Datel)[^rocket]
`08`  | [Capcom](https://en.wikipedia.org/wiki/Capcom)
`09`  | [HOT-B](https://www.mobygames.com/company/3889/hot-b-co-ltd/)
`0A`  | [Jaleco](https://en.wikipedia.org/wiki/Jaleco)
`0B`  | [Coconuts Japan Entertainment](https://www.mobygames.com/company/6388/coconuts-japan-entertainment-co-ltd/)
`0G`  | [Use Corporation](https://www.mobygames.com/company/17878/use-corporation/)
`0H`  | [Starfish Inc.](https://www.mobygames.com/company/9671/starfish-inc/)
`0K`  | [Shingakusha](https://gamefaqs.gamespot.com/games/company/74037-shingakusha)
`0L`  | [Warashi](https://en.wikipedia.org/wiki/Warashi)
`0N`  | [NOWPRO](https://en.wikipedia.org/wiki/Now_Production)
`0P`  | [NetVillage](https://www.mobygames.com/company/14979/fonfun-corporation/)[^village]
`0Q`  | [IE Institute](https://www.mobygames.com/company/10846/ie-institute-co-ltd/)
`13`  | [Electronic Arts Victor](https://www.mobygames.com/company/6003/electronic-arts-victor-inc/)[^eav]
`18`  | [Hudson Soft](https://en.wikipedia.org/wiki/Hudson_Soft)
`19`  | [B-AI](https://www.giantbomb.com/b-ai/3010-8160)
`1A`  | [Yanoman](https://www.mobygames.com/company/4856/yanoman-corporation/)
`1H`  | [Yojigen](https://en.wikipedia.org/wiki/Yojigen)
`1M`  | [Microcabin Corporation](https://en.wikipedia.org/wiki/Microcabin)
`1N`  | [DaZZ](https://www.mobygames.com/company/3836/dazz-co-ltd/)
`1P`  | [Creatures Inc.](https://en.wikipedia.org/wiki/Creatures_Inc.)[^creatures]
`1Q`  | [TDK Core](https://en.wikipedia.org/wiki/TDK)
`20`  | [KSS](https://en.wikipedia.org/wiki/KSS_(company))
`22`  | [Planning Office WADA](https://www.mobygames.com/company/15869/planning-office-wada)/[VR-1 Japan](https://www.mobygames.com/company/8578/vr-1-japan-inc/)[^vr1]
`28`  | [KEMCO](https://en.wikipedia.org/wiki/Kemco)[^kemco]
`2D`  | [Visit](https://www.mobygames.com/company/22168/visit-co-ltd/)
`2H`  | [Ubi Soft Entertainment (Japan)](https://en.wikipedia.org/wiki/Ubisoft)[^ubisoft]
`2K`  | [NEC Interchannel](https://en.wikipedia.org/wiki/Interchannel)
`2L`  | [TAM](https://www.mobygames.com/company/10516/tam-inc/)
`2M`  | [Jorudan](https://en.wikipedia.org/wiki/Jorudan)
`2N`  | [Smilesoft](https://en.wikipedia.org/wiki/Smilesoft)
`2P`  | [The Pokémon Company](https://en.wikipedia.org/wiki/The_Pok%C3%A9mon_Company)
`30`  | [Viacom New Media](https://www.mobygames.com/company/447/viacom-new-media/)
`34`  | [Magifact](https://www.mobygames.com/company/14892/magifact/)
`35`  | [HECT](https://en.wikipedia.org/wiki/Hect)
`36`  | [Codemasters](https://en.wikipedia.org/wiki/Codemasters)
`37`  | [GAGA Communications Inc.](https://www.mobygames.com/company/3751/gaga-communications-inc/)
`38`  | [Laguna](https://www.mobygames.com/company/6051/laguna-videospiele-vertriebs-marketing-gmbh/)/[Infogrames Deutschland](https://en.wikipedia.org/wiki/Atari_SA#Growth_through_acquisition_(1996%E2%80%932000))[^laguna]
`39`  | [Telstar Fun and Games](https://www.mobygames.com/company/9491/telstar-fun-and-games/)/[Evolution Entertainment](https://www.mobygames.com/company/12699/evolution-entertainment-srl/)[^event]
`3E`  | [Gremlin Graphics](https://en.wikipedia.org/wiki/Gremlin_Interactive)[^gremlin]
`3:`  | [Nintendo](https://en.wikipedia.org/wiki/Nintendo)/[Mani](https://www.mobygames.com/company/6058/mani-ltd/)
`41`  | [Ubi Soft Entertainment](https://en.wikipedia.org/wiki/Ubisoft)[^ubisoft]
`42`  | [Sunsoft](https://en.wikipedia.org/wiki/Sun_Corporation#Video_games) ([Project S-11](https://en.wikipedia.org/wiki/Project_S-11))
`47`  | [Spectrum HoloByte](https://en.wikipedia.org/wiki/Spectrum_HoloByte)
`4D`  | [Malibu Games](https://en.wikipedia.org/wiki/THQ)[^malibu]
`4F`  | [Eidos Interactive](https://en.wikipedia.org/wiki/Eidos_Interactive)
`4G`  | [Playmates](https://en.wikipedia.org/wiki/Playmates_Toys#Playmates_Interactive_Entertainment)
`4J`  | [Fox Interactive](https://en.wikipedia.org/wiki/Fox_Interactive)
`4K`  | [Time Warner Interactive](https://en.wikipedia.org/wiki/Time_Warner_Interactive)
`4S`  | [Black Pearl Software](https://www.mobygames.com/company/2217/black-pearl-software/)[^blackpearl]
`4X`  | [GT Interactive Software](https://en.wikipedia.org/wiki/Atari,_Inc._(1993%E2%80%93present)#As_GT_Interactive)
`4Y`  | [Rare](https://en.wikipedia.org/wiki/Rare_(company))
`4Z`  | [Crave Entertainment](https://en.wikipedia.org/wiki/Crave_Entertainment)
`50`  | [Absolute Entertainment](https://en.wikipedia.org/wiki/Absolute_Entertainment)
`51`  | [Acclaim Entertainment](https://en.wikipedia.org/wiki/Acclaim_Entertainment)
`52`  | [Activision](https://en.wikipedia.org/wiki/Activision)
`54`  | [GameTek](https://en.wikipedia.org/wiki/GameTek)/[Take2 Interactive Software Europe](https://en.wikipedia.org/wiki/Take-Two_Interactive)[^take2]
`55`  | [Hi Tech Entertainment](https://en.wikipedia.org/wiki/Hi_Tech_Expressions)[^hitech]
`56`  | [LJN](https://en.wikipedia.org/wiki/LJN)
`58`  | [Mattel Media/Mattel Interactive](https://en.wikipedia.org/wiki/Mattel_Interactive)[^mattel]
`5A`  | [Mindscape](https://en.wikipedia.org/wiki/Mindscape_(company))/[Red Orb Entertainment](https://en.wikipedia.org/wiki/Red_Orb_Entertainment)[^tlc]
`5D`  | [Williams Entertainment](https://en.wikipedia.org/wiki/WMS_Industries)/[Midway](https://en.wikipedia.org/wiki/Midway_Games)[^wms]
`5F`  | [ASC Games](https://en.wikipedia.org/wiki/ASC_Games)
`5G`  | [Majesco Sales, Inc.](https://en.wikipedia.org/wiki/Majesco_Entertainment)
`5H`  | [The 3DO Company](https://en.wikipedia.org/wiki/The_3DO_Company)
`5K`  | [Hasbro Interactive](https://en.wikipedia.org/wiki/Atari_Interactive)[^hasbro]
`5L`  | [NewKidCo](https://en.wikipedia.org/wiki/NewKidCo)
`5M`  | [Telegames](https://en.wikipedia.org/wiki/Telegames)
`5N`  | [Metro3D](https://en.wikipedia.org/wiki/Metro3D)
`5P`  | [Vatical Entertainment](https://www.mobygames.com/company/1789/vatical-entertainment-llc/)
`5Q`  | [LEGO Media/LEGO Software](https://en.wikipedia.org/wiki/The_Lego_Group#Lego_Interactive)[^lego]
`5T`  | [Cryo Interactive](https://en.wikipedia.org/wiki/Cryo_Interactive)
`5V`  | [Agetec, Inc.](https://en.wikipedia.org/wiki/Agetec)
`5W`  | [Red Storm Entertainment](https://en.wikipedia.org/wiki/Red_Storm_Entertainment)
`5X`  | [Microïds](https://en.wikipedia.org/wiki/Microids)[^microids]
`5Z`  | [Conspiracy Entertainment](https://en.wikipedia.org/wiki/Conspiracy_Entertainment)/[Classified Games](https://www.mobygames.com/company/3545/classified-games/)
`60`  | [Titus Interactive Studios](https://en.wikipedia.org/wiki/Titus_Interactive)
`61`  | [Virgin Interactive](https://en.wikipedia.org/wiki/Virgin_Interactive_Entertainment)[^virgin]
`64`  | [LucasArts Entertainment](https://en.wikipedia.org/wiki/Lucasfilm_Games)[^lucasarts]
`67`  | [Ocean](https://en.wikipedia.org/wiki/Ocean_Software)
`69`  | [Electronic Arts](https://en.wikipedia.org/wiki/Electronic_Arts)
`6F`  | [Electro Brain](https://en.wikipedia.org/wiki/Electro_Brain)
`6G`  | [The Learning Company](https://en.wikipedia.org/wiki/The_Learning_Company)
`6H`  | [BBC Worldwide](https://en.wikipedia.org/wiki/BBC_Worldwide)
`6J`  | [Software 2000](https://en.wikipedia.org/wiki/Software_2000)
`6L`  | [Bay Area Multimedia/BAM! Entertainment](https://en.wikipedia.org/wiki/BAM!_Entertainment)[^bam]
`6M`  | [Studio 3 Interactive Software](https://en.wikipedia.org/wiki/System_3_(company))[^studio3]
`6N`  | [Midas Interactive Entertainment](https://www.mobygames.com/company/1919/midas-interactive-entertainment-ltd/)
`6P`  | [Ravensburger Interactive Media](https://en.wikipedia.org/wiki/Ravensburger#Ravensburger_Interactive_Media)
`6Q`  | [Classified Games](https://www.mobygames.com/company/3545/classified-games/)
`6R`  | [Sound Source Interactive](https://en.wikipedia.org/wiki/TDK_Mediactive)[^sound]
`6S`  | [TDK Recording Media Europe S.A.](https://en.wikipedia.org/wiki/TDK)/[TDK Mediactive](https://en.wikipedia.org/wiki/TDK_Mediactive)[^tdk]
`6T`  | [Interactive Imagination](https://www.mobygames.com/company/5245/interactive-imagination/)
`6U`  | [DreamCatcher Interactive](https://en.wikipedia.org/wiki/DreamCatcher_Interactive)
`6V`  | [JoWooD Productions Software AG](https://en.wikipedia.org/wiki/JoWooD)
`6X`  | [Wanadoo Edition](https://en.wikipedia.org/wiki/Wanadoo)[^wanadoo]
`6Y`  | [Light & Shadow Production](https://www.mobygames.com/company/3106/light-shadow-production/)
`6Z`  | [ITE Media](https://en.wikipedia.org/wiki/Interactive_Television_Entertainment)
`70`  | [Infogrames](https://en.wikipedia.org/wiki/Atari_SA)[^atari]
`71`  | [Interplay](https://en.wikipedia.org/wiki/Interplay_Entertainment)[^interplay]
`72`  | [JVC Music Europe](https://en.wikipedia.org/wiki/Victor_Entertainment)[^jvc]
`75`  | [SCi Ltd.](https://en.wikipedia.org/wiki/SCi_Games)[^sci]
`78`  | [THQ](https://en.wikipedia.org/wiki/THQ)
`79`  | [Accolade](https://en.wikipedia.org/wiki/Accolade_(company))
(check)`7D`  | [Sierra On-Line](https://en.wikipedia.org/wiki/Sierra_Entertainment)/[Vivendi Universal Interactive Publishing](https://en.wikipedia.org/wiki/Vivendi_Games)/[Universal Interactive Studios](https://en.wikipedia.org/wiki/Universal_Interactive)
`7F`  | [KEMCO](https://en.wikipedia.org/wiki/Kemco)[^kemco]
`7G`  | [Rage Software](https://en.wikipedia.org/wiki/Rage_Software)
`7H`  | [Encore Software](https://en.wikipedia.org/wiki/Encore,_Inc.)
`7K`  | [KIDDINX Entertainment](https://www.mobygames.com/company/5752/kiddinx-entertainment-gmbh/)
`7L`  | [Simon & Schuster Interactive](https://en.wikipedia.org/wiki/Simon_%26_Schuster#Former_imprints)
`87`  | [Tsukuda Original](https://www.mobygames.com/company/19432/tsukuda-original-coltd/)
`8B`  | [Bullet-Proof Software (BPS)](https://en.wikipedia.org/wiki/Blue_Planet_Software)[^blueplanet]
`8C`  | [Vic Tokai](https://en.wikipedia.org/wiki/Tokai_Communications)[^tokaicomm]
`8F`  | [I'MAX](https://en.wikipedia.org/wiki/I%27MAX)[^imax]
`8J`  | [Kadokawa Shoten](https://en.wikipedia.org/wiki/Kadokawa_Shoten)
`8K`  | [Japan System Supply](https://www.mobygames.com/company/3509/japan-system-supply/)
`8M`  | [CyberFront](https://www.mobygames.com/company/1866/cyberfront-corporation/)
`8N`  | [Success](https://en.wikipedia.org/wiki/Success_(company))
`8P`  | [Sega](https://en.wikipedia.org/wiki/Sega)
`91`  | [Chunsoft](https://en.wikipedia.org/wiki/Spike_Chunsoft)[^spike]
`92`  | [Video System](https://www.mobygames.com/company/2706/video-system-co-ltd/)
`93`  | [BEC](https://en.wikipedia.org/wiki/B.B._Studio)[^bec]
`99`  | [Pack-In-Video](https://en.wikipedia.org/wiki/Pack-In-Video)/[Victor Interactive Software](https://en.wikipedia.org/wiki/Victor_Interactive_Software)[^packin]
`9A`  | [Nichibutsu](https://en.wikipedia.org/wiki/Nihon_Bussan)[^nichibutsu]
`9B`  | [Tecmo](https://en.wikipedia.org/wiki/Tecmo)
`9C`  | [Imagineer](https://en.wikipedia.org/wiki/Imagineer_(Japanese_company))
`9H`  | [Bottom Up](https://www.mobygames.com/company/3699/bottom-up/)
`9K`  | [Syscom Entertainment](https://www.mobygames.com/company/3230/syscom-entertainment-inc/)
`9L`  | [Hasbro Japan](https://www.mobygames.com/company/13993/hasbro-japan-ltd/)
`9M`  | [Jaguar](https://www.mobygames.com/company/10935/jaguar/)
`9N`  | [Marvelous Entertainment](https://en.wikipedia.org/wiki/Marvelous_Entertainment)
`A0`  | [Telenet Japan](https://en.wikipedia.org/wiki/Telenet_Japan)
`A1`  | [Hori Electric](https://ja.wikipedia.org/wiki/ホリ_(ゲーム周辺機器メーカー))[^hori]
`A4`  | [Konami](https://en.wikipedia.org/wiki/Konami)
`A7`  | [Takara](https://en.wikipedia.org/wiki/Takara)
`A9`  | [Technōs Japan Corp.](https://en.wikipedia.org/wiki/Technōs_Japan)
`AD`  | [Toho](https://en.wikipedia.org/wiki/Toho)
`AF`  | [Namco](https://en.wikipedia.org/wiki/Namco)
`AH`  | [J・Wing](https://www.mobygames.com/company/12243/j-wing-co-ltd/)
`AK`  | [KID](https://en.wikipedia.org/wiki/KID)
`AL`  | [Media Factory](https://en.wikipedia.org/wiki/Media_Factory)
`AM`  | [BIOX](https://www.mobygames.com/company/11841/biox-co-ltd/)
`AN`  | [LAYUP](https://www.mobygames.com/company/25452/layup-co-ltd/)
`AP`  | [Infogrames Hudson](https://en.wikipedia.org/wiki/List_of_Atari_SA_subsidiaries#Atari_Japan_KK)[^atarikk]
`AQ`  | [Ludic Inc.](https://www.mobygames.com/company/12166/ludic/)[^ludic]
`B0`  | [Acclaim Japan](https://www.mobygames.com/company/6050/acclaim-japan-ltd/)
`B1`  | [ASCII Corporation](https://en.wikipedia.org/wiki/ASCII_Corporation)[^ascii]
`B2`  | [Bandai](https://en.wikipedia.org/wiki/Bandai)
`B4`  | [Enix](https://en.wikipedia.org/wiki/Enix)
`BA`  | [Culture Brain](https://en.wikipedia.org/wiki/Culture_Brain)
`BB`  | [Sunsoft](https://en.wikipedia.org/wiki/Sun_Corporation#Video_games)
`BF`  | [Sammy](https://en.wikipedia.org/wiki/Sammy_Corporation)
`BG`  | [Magical Company](https://en.wikipedia.org/wiki/Magical_Company)
`BJ`  | [Compile](https://en.wikipedia.org/wiki/Compile_(company))
`BL`  | [MTO](https://en.wikipedia.org/wiki/MTO_(video_game_company))
`BM`  | [XING Entertainment](https://www.mobygames.com/company/5123/xing-entertainment/)
`BN`  | [Sunrise Interactive](https://en.wikipedia.org/wiki/Bandai_Namco_Filmworks#Sunrise)[^sunrise]
`BP`  | [Global A Entertainment](https://en.wikipedia.org/wiki/GAE_(company))[^gae]
`C0`  | [Taito](https://en.wikipedia.org/wiki/Taito)
`C6`  | [Tonkin House](https://en.wikipedia.org/wiki/Tonkin_House)
`C8`  | [Koei](https://en.wikipedia.org/wiki/Koei)
`CB`  | [VAP](https://en.wikipedia.org/wiki/VAP,_Inc.)
`CE`  | [Pony Canyon](https://en.wikipedia.org/wiki/Pony_Canyon)
`CF`  | [Angel](https://www.mobygames.com/company/5083/angel)
`CJ`  | [BOSS Communications](https://gamefaqs.gamespot.com/games/company/73501-boss-communications)
`CK`  | [Axela](https://www.mobygames.com/company/6484/axela/)
`CN`  | [NEC Interchannel](https://en.wikipedia.org/wiki/Interchannel)[^interchannel]
`CP`  | [Enterbrain](https://en.wikipedia.org/wiki/Enterbrain)
`D4`  | [ASK](https://en.wikipedia.org/wiki/ASK_Group)
`D6`  | [Naxat Soft](https://en.wikipedia.org/wiki/Kaga_Create)[^kaga]
`D9`  | [Banpresto](https://en.wikipedia.org/wiki/Banpresto)
`DA`  | [TOMY](https://en.wikipedia.org/wiki/Tomy)
`DD`  | [NCS](https://www.mobygames.com/company/4490/ncs-corporation/)
`DE`  | [Human Entertainment](https://en.wikipedia.org/wiki/Human_Entertainment)
`DF`  | [Altron Corporation](https://www.mobygames.com/company/3808/altron-corporation/)
`DH`  | [Gaps Inc.](https://www.mobygames.com/company/12006/gaps-inc/)
`DJ`  | [Epoch](https://en.wikipedia.org/wiki/Epoch_Co.)/[Shogakukan](https://en.wikipedia.org/wiki/Shogakukan)
`DK`  | [Kodansha](https://en.wikipedia.org/wiki/Kodansha)
`DL`  | [Digital Kids](https://www.mobygames.com/company/10615/ubisoft-nagoya/)[^digitalkids]
`DN`  | [ELF](https://en.wikipedia.org/wiki/ELF_Corporation)
`DP`  | [Prime Systems Corporation](https://www.mobygames.com/company/35172/sunrise-technology-corporation/)[^prime]
`E2`  | [Yutaka](https://en.wikipedia.org/wiki/Yutaka_(video_game_company))[^yutaka]
`E5`  | [Epoch](https://en.wikipedia.org/wiki/Epoch_Co.)
`E7`  | [Athena](https://en.wikipedia.org/wiki/Athena_(game_developer))
`E8`  | [Asmik/Asmik Ace Entertainment](https://en.wikipedia.org/wiki/Asmik_Ace)[^asmik]
`E9`  | [Natsume](https://en.wikipedia.org/wiki/Natsume_Inc.)
`EA`  | [King Records](https://en.wikipedia.org/wiki/King_Records_(Japan))
`EB`  | [Atlus](https://en.wikipedia.org/wiki/Atlus)
`EJ`  | [See old licensee code](<# 014B — Old licensee code>)
`EL`  | [Spike](https://en.wikipedia.org/wiki/Spike_(company))[^spike]
`EN`  | [AlphaDream Corporation](https://en.wikipedia.org/wiki/AlphaDream)
`EP`  | [Sting Entertainment](https://en.wikipedia.org/wiki/Sting_Entertainment)
`EQ`  | [Omega Micott](https://www.mobygames.com/company/5934/omega-micott-inc/)
`FB`  | [Psygnosis](https://en.wikipedia.org/wiki/Psygnosis)
`FG`  | [Jupiter Corporation](https://en.wikipedia.org/wiki/Jupiter_Corporation) [Bomberman Selection](https://en.wikipedia.org/wiki/Bomberman_GB#Other_releases)
`GC`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)
`MK`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)
`RX`  | [Hitek](https://bootleggames.fandom.com/wiki/Hitek)/[Li Cheng](https://bootleggames.fandom.com/wiki/Guangzhou_Li_Cheng_Industry_%26_Trade_Co) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)) ([Terrifying 9/11](https://bootleggames.fandom.com/wiki/Terrifying_911))
`S5`  | [SouthPeak Interactive](https://en.wikipedia.org/wiki/SouthPeak_Games)
`XX`  | [Rocket Games](https://en.wikipedia.org/wiki/Datel)[^rocket]
`YY`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)
`--`  | [Infogrames](https://en.wikipedia.org/wiki/Atari_SA)[^atari] ([Die Maus - Verrueckte Olympiade](https://www.mobygames.com/game/18740/die-maus-verruckte-olympiade/) beta)
`-2`  | [Gowin](https://bootleggames.fandom.com/wiki/Gowin) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki))
`@7`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)
` >`  | [Nintendo](https://en.wikipedia.org/wiki/Nintendo) ([Donkey Kong Land III](https://en.wikipedia.org/wiki/Donkey_Kong_Land_III) beta)
` 1`  | [Sachen](https://bootleggames.fandom.com/wiki/Sachen) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki))
` 9`  | [Gowin](https://bootleggames.fandom.com/wiki/Gowin) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki))

## 0146 — SGB flag

This byte specifies whether the game supports SGB functions.
The SGB will ignore any [command packets](<#Command Packet Transfers>) if this byte is set to a value other than `$03` (typically `$00`).

## 0147 — Cartridge type

This byte indicates what kind of hardware is present on the cartridge — most notably its [mapper](#MBCs).

Code  | Type
------|--------------------------------
`$00` | ROM ONLY
`$01` | MBC1
`$02` | MBC1+RAM
`$03` | MBC1+RAM+BATTERY
`$05` | MBC2
`$06` | MBC2+BATTERY
`$08` | ROM+RAM [^rom_ram]
`$09` | ROM+RAM+BATTERY [^rom_ram]
`$0B` | MMM01
`$0C` | MMM01+RAM
`$0D` | MMM01+RAM+BATTERY
`$0F` | MBC3+TIMER+BATTERY
`$10` | MBC3+TIMER+RAM+BATTERY [^mbc30]
`$11` | MBC3
`$12` | MBC3+RAM [^mbc30]
`$13` | MBC3+RAM+BATTERY [^mbc30]
`$19` | MBC5
`$1A` | MBC5+RAM
`$1B` | MBC5+RAM+BATTERY
`$1C` | MBC5+RUMBLE
`$1D` | MBC5+RUMBLE+RAM
`$1E` | MBC5+RUMBLE+RAM+BATTERY
`$20` | MBC6
`$22` | MBC7+SENSOR+RUMBLE+RAM+BATTERY
`$FC` | POCKET CAMERA
`$FD` | BANDAI TAMA5
`$FE` | HuC3
`$FF` | HuC1+RAM+BATTERY

[^rom_ram]:
No licensed cartridge makes use of this option. The exact behavior is unknown.

[^mbc30]:
MBC3 with 64 KiB of SRAM refers to MBC30, used only in _Pocket Monsters: Crystal Version_ (the Japanese version of _Pokémon Crystal Version_).

## 0148 — ROM size

This byte indicates how much ROM is present on the cartridge.
In most cases, the ROM size is given by `32 KiB × (1 << <value>)`:

Value | ROM size  | Number of ROM banks
------|-----------|----------------------
`$00` |  32 KiB   | 2 (no banking)
`$01` |  64 KiB   | 4
`$02` | 128 KiB   | 8
`$03` | 256 KiB   | 16
`$04` | 512 KiB   | 32
`$05` |   1 MiB   | 64
`$06` |   2 MiB   | 128
`$07` |   4 MiB   | 256
`$08` |   8 MiB   | 512
`$52` | 1.1 MiB   | 72 [^weird_rom_sizes]
`$53` | 1.2 MiB   | 80 [^weird_rom_sizes]
`$54` | 1.5 MiB   | 96 [^weird_rom_sizes]

[^weird_rom_sizes]:
Only listed in unofficial docs. No cartridges or ROM files using these sizes are known.
As the other ROM sizes are all powers of 2, these are likely inaccurate.
The source of these values is unknown.

## 0149 — RAM size

This byte indicates how much RAM is present on the cartridge, if any.

If the [cartridge type](<#0147 — Cartridge type>) does not include "RAM" in its name, this should be set to 0.
This includes MBC2, since its 512 × 4 bits of memory are built directly into the mapper.

Code  | SRAM size | Comment
------|-----------|-----------------------
`$00` |   0       | No RAM
`$01` |   –       | Unused [^2kib_sram]
`$02` |   8 KiB   |  1 bank
`$03` |  32 KiB   |  4 banks of 8 KiB each
`$04` | 128 KiB   | 16 banks of 8 KiB each
`$05` |  64 KiB   |  8 banks of 8 KiB each

[^2kib_sram]:
Listed in various unofficial docs as 2 KiB.
However, a 2 KiB RAM chip was never used in a cartridge.
The source of this value is unknown.

Various "PD" ROMs ("Public Domain" homebrew ROMs, generally tagged with `(PD)` in the filename) are known to use the `$01` RAM Size tag, but this is believed to have been a mistake with early homebrew tools, and the PD ROMs often don't use cartridge RAM at all.

## 014A — Destination code

This byte specifies whether this version of the game is intended to be sold in Japan or elsewhere.

Only two values are defined:

Code  | Destination
------|------------------------------
`$00` | Japan (and possibly overseas)
`$01` | Overseas only

## 014B — Old licensee code

This byte is used in older (pre-SGB) cartridges to specify the game's publisher.
However, the value `$33` indicates that the [New licensee codes](<#0144–0145 — New licensee code>) must be considered instead.
(The SGB will ignore any [command packets](<#Command Packet Transfers>) unless this value is `$33`.)

Here is a list of known Old licensee codes ([source](https://raw.githubusercontent.com/gb-archive/salvage/master/txt-files/gbrom.txt)).

HEX   | Licensee
------|------------
`00`  | None
`01`  | [Nintendo](https://en.wikipedia.org/wiki/Nintendo)
`08`  | [Capcom](https://en.wikipedia.org/wiki/Capcom)
`09`  | [HOT-B](https://www.mobygames.com/company/3889/hot-b-co-ltd/)
`0A`  | [Jaleco](https://en.wikipedia.org/wiki/Jaleco)
`0B`  | [Coconuts Japan Entertainment](https://www.mobygames.com/company/6388/coconuts-japan-entertainment-co-ltd/)
`0C`  | [Elite Systems](https://en.wikipedia.org/wiki/Elite_Systems)
`10`  | [Banalex](https://www.mobygames.com/company/15929/banalex/)
`12`  | [Infocom](https://www.mobygames.com/company/14903/nissho-iwai-infocom-systems-co-ltd/)
`13`  | [Electronic Arts](https://en.wikipedia.org/wiki/Electronic_Arts)
`17`  | [Sachen](https://bootleggames.fandom.com/wiki/Sachen) ([Street Heroes](https://bootleggames.fandom.com/wiki/Street_Heroes)) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki))
`18`  | [Hudson Soft](https://en.wikipedia.org/wiki/Hudson_Soft)
`19`  | [ITC Entertainment](https://en.wikipedia.org/wiki/ITC_Entertainment)
`1A`  | [Yanoman](https://www.mobygames.com/company/4856/yanoman-corporation/)
`1D`  | [Japan Clary Business](https://www.mobygames.com/company/7639/japan-clary-business/)
`1F`  | [Virgin Games](https://en.wikipedia.org/wiki/Virgin_Interactive_Entertainment)[^virgin]
`20`  | [Xploder](https://en.wikipedia.org/wiki/Xploder)
`21`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)) ([Guaishou Go! Go! II](https://bootleggames.fandom.com/wiki/Monster_Go!_Go!_Go!!#Monster_Go!_Go!_II))
`23`  | [Micro World](https://www.mobygames.com/company/6164/micro-world/)
`24`  | [PCM Complete](https://www.mobygames.com/company/9489/pcm-complete)
`25`  | [San-X](https://en.wikipedia.org/wiki/San-X)
`28`  | [Kotobuki System/KEMCO](https://en.wikipedia.org/wiki/Kemco)[^kemco]
`29`  | [SETA](https://en.wikipedia.org/wiki/SETA_Corporation)
`2B`  | [Irem](https://en.wikipedia.org/wiki/Irem) ([Kizuchida Quiz da Gen-san da!](https://gamefaqs.gamespot.com/gameboy/569781-kizuchida-quiz-da-gen-san-da))
`2D`  | [Visit](https://www.mobygames.com/company/22168/visit-co-ltd/)
`33`  | Indicates that the [New licensee code](<#0144–0145 — New licensee code>) should be used instead.
`34`  | [Konami](https://en.wikipedia.org/wiki/Konami)
`35`  | [HECT](https://en.wikipedia.org/wiki/Hect)/[DTMC](https://www.mobygames.com/company/2916/dtmc-inc/)[^dtmc]
`38`  | [Capcom](https://en.wikipedia.org/wiki/Capcom)
`3C`  | [Empire Interactive](https://en.wikipedia.org/wiki/Empire_Interactive)
`3D`  | [Loriciel](https://en.wikipedia.org/wiki/Loriciel)
`3E`  | [Gremlin Graphics](https://en.wikipedia.org/wiki/Gremlin_Interactive)[^gremlin]
`41`  | [Ubi Soft](https://en.wikipedia.org/wiki/Ubisoft)[^ubisoft]
`47`  | [Spectrum HoloByte](https://en.wikipedia.org/wiki/Spectrum_HoloByte)
`48`  | [Fabtek](https://en.wikipedia.org/wiki/Fabtek)
`49`  | [Irem](https://en.wikipedia.org/wiki/Irem)
`4A`  | [Virgin Games](https://en.wikipedia.org/wiki/Virgin_Interactive_Entertainment)[^virgin]
`4B`  | [Makon Soft](https://bootleggames.fandom.com/wiki/Makon_Soft) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)) ([Rockman 8](https://bootleggames.fandom.com/wiki/Rockman_8))
`4D`  | [Malibu Games](https://en.wikipedia.org/wiki/THQ)[^malibu]
`4F`  | [U.S. Gold](https://en.wikipedia.org/wiki/U.S._Gold)[^usgold]
`50`  | [Absolute Entertainment](https://en.wikipedia.org/wiki/Absolute_Entertainment)
`51`  | [Acclaim Entertainment](https://en.wikipedia.org/wiki/Acclaim_Entertainment)
`52`  | [Activision](https://en.wikipedia.org/wiki/Activision)
`53`  | [American Sammy Corporation](https://en.wikipedia.org/wiki/Sammy_Corporation)
`54`  | [GameTek](https://en.wikipedia.org/wiki/GameTek)[^take2]
`55`  | [Hi Tech Entertainment](https://en.wikipedia.org/wiki/Hi_Tech_Expressions)[^hitech]
`56`  | [LJN](https://en.wikipedia.org/wiki/LJN)
`57`  | [Matchbox](https://en.wikipedia.org/wiki/Matchbox_(brand))
`59`  | [Milton Bradley Company](https://en.wikipedia.org/wiki/Milton_Bradley_Company)
`5A`  | [Mindscape](https://en.wikipedia.org/wiki/Mindscape_(company))[^tlc]
`5B`  | [Romstar](https://en.wikipedia.org/wiki/Romstar)
`5C`  | [Taxan](https://en.wikipedia.org/wiki/Taxan)
`5D`  | [Tradewest](https://en.wikipedia.org/wiki/Tradewest)[^wms]
`5E`  | [INTV](https://en.wikipedia.org/wiki/Intellivision)[^intv]
`5F`  | [ASC Games](https://en.wikipedia.org/wiki/ASC_Games)
`60`  | [Titus Interactive](https://en.wikipedia.org/wiki/Titus_Interactive)
`61`  | [Arcadia Systems](https://en.wikipedia.org/wiki/Mastertronic#arcadia)/[Virgin Games/Virgin Interactive](https://en.wikipedia.org/wiki/Virgin_Interactive_Entertainment)[^virgin]
`65`  | [Activision](https://en.wikipedia.org/wiki/Activision) ([Death Track](https://gamefaqs.gamespot.com/gameboy/284231-death-track))
`67`  | [Ocean Software](https://en.wikipedia.org/wiki/Ocean_Software)
`69`  | [Electronic Arts](https://en.wikipedia.org/wiki/Electronic_Arts)
`6B`  | [Beam Software](https://en.wikipedia.org/wiki/Beam_Software)[^beam]
`6E`  | [Elite Systems](https://en.wikipedia.org/wiki/Elite_Systems)
`6F`  | [Electro Brain](https://en.wikipedia.org/wiki/Electro_Brain)
`70`  | [Infogrames](https://en.wikipedia.org/wiki/Atari_SA)[^atari]
`71`  | [Interplay Productions](https://en.wikipedia.org/wiki/Interplay_Entertainment)[^interplay]
`72`  | [Victor Musical Industries/JVC Musical Industries](https://en.wikipedia.org/wiki/Victor_Entertainment)[^jvcus]
`73`  | [Parker Brothers](https://en.wikipedia.org/wiki/Parker_Brothers#Video_games)
`75`  | [The Sales Curve Limited](https://en.wikipedia.org/wiki/SCi_Games)[^sci]
`78`  | [THQ](https://en.wikipedia.org/wiki/THQ)
`79`  | [Accolade](https://en.wikipedia.org/wiki/Accolade_(company))[^infogrames]
`7A`  | [Triffix Entertainment](https://www.mobygames.com/company/4307/triffix-entertainment-inc)
`7C`  | [MicroProse Software](https://en.wikipedia.org/wiki/MicroProse)
`7F`  | [Kotobuki System/KEMCO](https://en.wikipedia.org/wiki/Kemco)[^kemco]
`80`  | [Misawa Entertainment](https://www.mobygames.com/company/8225/misawa-entertainment-coltd)
`82`  | [Namco](https://en.wikipedia.org/wiki/Namco)
`83`  | [G. Amusements](https://www.mobygames.com/company/8256/lozcg-amusements-co-ltd/)[^lozc]
`86`  | [Tokuma Shoten Intermedia](https://en.wikipedia.org/wiki/Tokuma_Shoten)
`8B`  | [Bullet-Proof Software](https://en.wikipedia.org/wiki/Blue_Planet_Software)[^blueplanet]
`8C`  | [Vic Tokai](https://en.wikipedia.org/wiki/Tokai_Communications)[^tokaicomm]
`8E`  | [Character Soft](https://www.mobygames.com/company/5153/character-soft-co-ltd/)
`8F`  | [I'Max](https://en.wikipedia.org/wiki/I%27MAX)[^imax]
`92`  | [Video System](https://www.mobygames.com/company/2706/video-system-co-ltd/)
`93`  | [BEC](https://en.wikipedia.org/wiki/B.B._Studio)[^bec]
`95`  | [Varie](https://en.wikipedia.org/wiki/Varie)
`96`  | [Yonezawa](https://en.wikipedia.org/wiki/Sega_Fave)[^segabuy]/S'Pal
`97`  | [KANEKO](https://en.wikipedia.org/wiki/Kaneko)
`99`  | [Pack-In-Video](https://en.wikipedia.org/wiki/Pack-In-Video)[^packin]
`9A`  | [Nichibutsu](https://en.wikipedia.org/wiki/Nihon_Bussan)[^nichibutsu]
`9B`  | [Tecmo](https://en.wikipedia.org/wiki/Tecmo)
`9C`  | [Imagineer](https://en.wikipedia.org/wiki/Imagineer_(Japanese_company))
`9E`  | [Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)
`9F`  | [Nova](https://www.mobygames.com/company/8042/nova-co-ltd/)
`A4`  | [Konami](https://en.wikipedia.org/wiki/Konami)
`A6`  | [Kawada](https://www.mobygames.com/company/8110/kawada-co-ltd/)
`A7`  | [Takara](https://en.wikipedia.org/wiki/Takara)
`A9`  | [Technōs Japan Corp.](https://en.wikipedia.org/wiki/Technōs_Japan)
`AA`  | [Victor Musical Industries/Victor Entertainment](https://en.wikipedia.org/wiki/Victor_Entertainment)[^victor]
`AC`  | [Toei Animation](https://en.wikipedia.org/wiki/Toei_Animation)
`AD`  | [Toho](https://en.wikipedia.org/wiki/Toho)
`AF`  | [Namco](https://en.wikipedia.org/wiki/Namco)
`B0`  | [Acclaim Japan](https://www.mobygames.com/company/6050/acclaim-japan-ltd/)
`B1`  | [ASCII Corporation](https://en.wikipedia.org/wiki/ASCII_Corporation)[^ascii]
`B2`  | [Bandai](https://en.wikipedia.org/wiki/Bandai)
`B4`  | [Enix](https://en.wikipedia.org/wiki/Enix)
`B6`  | [HAL Laboratory](https://en.wikipedia.org/wiki/HAL_Laboratory)
`B7`  | [SNK](https://en.wikipedia.org/wiki/SNK)
`B9`  | [Pony Canyon](https://en.wikipedia.org/wiki/Pony_Canyon)
`BA`  | [Culture Brain](https://en.wikipedia.org/wiki/Culture_Brain)
`BB`  | [Sunsoft](https://en.wikipedia.org/wiki/Sun_Corporation#Video_games)
`BD`  | [Sony Imagesoft/Sony Electronic Publishing](https://en.wikipedia.org/wiki/Sony_Imagesoft)[^imagesoft]/[Epic/Sony Records](https://en.wikipedia.org/wiki/Epic_Records_Japan)[^epic]
`BF`  | [Sammy](https://en.wikipedia.org/wiki/Sammy_Corporation)
`C0`  | [Taito](https://en.wikipedia.org/wiki/Taito)
`C2`  | [Kotobuki System/KEMCO](https://en.wikipedia.org/wiki/Kemco)[^kemco]
`C3`  | [Square](https://en.wikipedia.org/wiki/Square_(video_game_company))
`C4`  | [Tokuma Shoten Intermedia](https://en.wikipedia.org/wiki/Tokuma_Shoten)
`C5`  | [Data East](https://en.wikipedia.org/wiki/Data_East)
`C6`  | [Tonkin House](https://en.wikipedia.org/wiki/Tonkin_House)
`C8`  | [Koei](https://en.wikipedia.org/wiki/Koei)
`C9`  | [UPL](https://en.wikipedia.org/wiki/UPL_Co.,_Ltd)
`CA`  | [Ultra Games](https://en.wikipedia.org/wiki/Ultra_Games)/[Konami](https://en.wikipedia.org/wiki/Konami)
`CB`  | [VAP](https://en.wikipedia.org/wiki/VAP,_Inc.)
`CC`  | [Use Corporation](https://www.mobygames.com/company/17878/use-corporation/)
`CD`  | [Meldac](https://en.wikipedia.org/wiki/Meldac)
`CE`  | [Pony Canyon](https://en.wikipedia.org/wiki/Pony_Canyon)/[FCI](https://en.wikipedia.org/wiki/Fujisankei_Communications_International)[^fci]
`CF`  | [Angel](https://www.mobygames.com/company/5083/angel)
`D0`  | [Taito](https://en.wikipedia.org/wiki/Taito)
`D1`  | [SOFEL](https://en.wikipedia.org/wiki/SOFEL)
`D2`  | [Quest](https://en.wikipedia.org/wiki/Quest_Corporation)
`D3`  | [Sigma Enterprises](https://www.mobygames.com/company/5001/sigma-enterprises-inc)
`D4`  | [ASK Kodansha](https://www.mobygames.com/company/5166/ask-co-ltd/)
`D6`  | [Naxat Soft](https://en.wikipedia.org/wiki/Kaga_Create)[^kaga]
`D7`  | [Copya System](https://www.mobygames.com/company/8864/shangri-la-corporation/)[^copya]
`D9`  | [Banpresto](https://en.wikipedia.org/wiki/Banpresto)
`DA`  | [Tomy](https://en.wikipedia.org/wiki/Tomy)
`DB`  | Hiro/Acclaim/[LJN](https://en.wikipedia.org/wiki/LJN)
`DD`  | [NCS](https://www.mobygames.com/company/4490/ncs-corporation/)
`DE`  | [Human Entertainment](https://en.wikipedia.org/wiki/Human_Entertainment)
`DF`  | [Altron Corporation](https://www.mobygames.com/company/3808/altron-corporation/)
`E0`  | [Jaleco](https://en.wikipedia.org/wiki/Jaleco)
`E1`  | [Towa Chiki](https://en.wikipedia.org/wiki/Towa_Chiki)
`E2`  | [Bandai Shinsei/Yutaka](https://en.wikipedia.org/wiki/Yutaka_(video_game_company))[^yutaka]
`E3`  | [Varie](https://en.wikipedia.org/wiki/Varie)
`E4`  | [T&E Soft](https://en.wikipedia.org/wiki/T%26E_Soft)
`E5`  | [Epoch](https://en.wikipedia.org/wiki/Epoch_Co.)
`E7`  | [Athena](https://en.wikipedia.org/wiki/Athena_(game_developer))
`E8`  | [Asmik](https://en.wikipedia.org/wiki/Asmik_Ace)/[Asmik Corporation of America](https://www.mobygames.com/company/9089/asmik-corp-of-america/)[^asmik]
`E9`  | [Natsume](https://en.wikipedia.org/wiki/Natsume_Inc.)
`EA`  | [King Records](https://en.wikipedia.org/wiki/King_Records_(Japan))
`EB`  | [Atlus](https://en.wikipedia.org/wiki/Atlus)
`EC`  | [Epic/Sony Records](https://en.wikipedia.org/wiki/Epic_Records_Japan)[^epic]
`EE`  | [IGS](https://en.wikipedia.org/wiki/Information_Global_Service)
`F0`  | [A Wave](https://www.mobygames.com/company/9123/a-wave-inc/)
`F1`  | [Makon Soft](https://bootleggames.fandom.com/wiki/Makon_Soft) ([Unlicensed](https://bootleggames.fandom.com/wiki/BootlegGames_Wiki)) ([Super Donkey Kong 3](https://bootleggames.fandom.com/wiki/Super_Donkey_Kong_3))
`F3`  | [Extreme Entertainment](https://www.mobygames.com/company/4221/extreme-entertainment-group-inc)

[^rocket]: Datel in (a thinly-veiled) disguise. Not a lot of information available but [this article](https://www.nesworld.com/article.php?system=gbx&data=gbc-rocketgames) compiles most of it.

[^village]: NetVillage's game division was called [GameVillage](https://www.mobygames.com/company/4501/gamevillage/), but [Cross Hunter](https://gamefaqs.gamespot.com/gbc/578708-cross-hunter) uses both names. NetVillage was rebranded to [Fonfun](https://ja.wikipedia.org/wiki/Fonfun) in 2006.

[^eav]: A joint venture between [Electronic Arts](https://en.wikipedia.org/wiki/Electronic_Arts) and [Victor Musical Industries](<[^victor]>) to release Japanese versons of EA games from 1992 to 1998.

[^vr1]: [VR-1, Inc.](https://www.mobygames.com/company/2404/vr1-entertainment-inc/) acquired Planning Office WADA (POW) in 1999, and rebranded them as VR-1 Japan. VR-1 later merged with [Jaleco USA](https://en.wikipedia.org/wiki/Jaleco#History) to form Jaleco Entertainment in 2002. 

[^kemco]: Originally founded as Kotobuki System Co., Ltd. in 1984. The name "KEMCO" comes from the initials of parent company **K**otobuki **E**ngineering & **M**anufacturing **C**o., Ltd. The company didn't officially change its name until 2004.

[^ubisoft]: Later known as [Ubisoft](https://en.wikipedia.org/wiki/Ubisoft).

[^laguna]: Game division for Bomico, later merged into Infogrames Deutschland as part of Infogrames' acquisition of Bomico. The transition can be seen in the [Sea Battle](https://www.mobygames.com/game/69462/sea-battle/) beta (originally called "Battle Ships"), where both logos appear and Laguna's [new licensee code](<#0144–0145 — New licensee code>) is used, and in the beta for [Otto's Ottifanten: Baby Bruno's Alptraum](https://www.mobygames.com/game/55817/ottos-ottifanten-baby-brunos-alptraum/), where the publisher is branded as Infogrames Deutschland but still uses the Laguna licensee code. Infogrames Deutschland were later sold to [Namco Bandai Games](https://en.wikipedia.org/wiki/Bandai_Namco_Entertainment).

[^event]: This [new licensee code](<#0144–0145 — New licensee code>) was originally used for Telstar Fun and Games, until their parent company, [Telstar Electronics Studios](https://www.mobygames.com/company/1415/telstar-electronic-studios-ltd/), was effectively acquired by [Take-Two Interactive](<^take2>) in 1999. From 2000 onwards, it appears Take-Two merged Telstar into one of their subidiaries, so Evolution Entertainment (stylised as "EVENT") started using this licensee code instead.

[^gremlin]: Later known as Gremlin Interactive, and sold to Infogrames in 1999.

[^malibu]: Malibu Games was a subsidiary of THQ. Not be confused with Malibu Interactive (a subsidiary of
 Malibu Comics) who collaborated with Malibu Games on some SNES games ([Sports Illustrated Championship 
Football & Baseball]
(https://en.wikipedia.org/wiki/Sports_Illustrated:_Championship_Football_%26_Baseball) (known as *All-American Championship Football in Europe*) and 
[Time Trax](https://en.wikipedia.org/wiki/Time_Trax_(video_game)). Malibu Interactive don't appear to 
have any publishing credits for Game Boy/Game Boy Color.

[^blackpearl]: A subsidiary of [THQ](https://en.wikipedia.org/wiki/THQ#Subsidiaries), Black Pearl was originally used only for Sega games. From 1995 until the start of its dissolution in 1997, the Black Pearl Software name was used for 8-bit and 16-bit games, now including Nintendo.

[^take2]: GameTek went out of business at the end of 1998, with most of their assets acquired by Take-Two Interactive Software and rebranded as Take-Two Interactive Software Europe. "Take-Two Interactive" is sometimes stylised as "Take 2 Interactive".

[^hitech]: Hi Tech Expressions was later known as Hi Tech Entertainment.

[^mattel]: Originally founded as Mattel Media in 1996. Later known as Mattel Interactive as of 1999.

[^tlc]: Mindscape and [Broderbund](https://en.wikipedia.org/wiki/Broderbund) (Red Orb's parent company) were purchased by [The Learning Company](https://en.wikipedia.org/wiki/The_Learning_Company) but the Mindscape and Red Orb names continued to be used for publishing.

[^wms]: [WMS Industries](https://en.wikipedia.org/wiki/WMS_Industries) purchased Midway in 1988 and Tradewest in 1994. WMS later rebranded Midway to [Midway Amusement Games, LLC](https://en.wikipedia.org/wiki/Midway_Games#Publishing_and_distribution) for their arcade division, and in 1996, rebranded Tradewest to Williams Entertainment, Inc. for their home console adaptions. Williams Entertainment was later rebranded to Midway Home Entertainment. Midway was spun off into its own company in 1998, under the name [Midway Games Inc.](https://en.wikipedia.org/wiki/Midway_Games), with both Midway Amusement Games and Midway Home Entertainment becoming its subsidiaries.

[^hasbro]: Later sold to Infogrames, who renamed it to Infogrames Interactive. It was then renamed to Atari Interactive when Infogrames rebranded to [Atari SA](https://en.wikipedia.org/wiki/Atari_SA).

[^lego]: Originally called LEGO Media, then LEGO Software, and finally, LEGO Interactive.

[^microids]: Later known as MC2-Microïds (after merging with MC2), then Microids (without the [diaresis](https://en.wikipedia.org/wiki/Diaeresis_(diacritic))).

[^virgin]: Arcadia Systems' parent company, [Mastertronic](https://en.wikipedia.org/wiki/Mastertronic), was bought by [Virgin](https://en.wikipedia.org/wiki/Virgin_Interactive_Entertainment) and merged to form Virgin Mastertronic Ltd., under which it continued using the Virgin Games label. The publishing side of Virgin Mastertronic was later renamed Virgin Interactive Entertainment, while the game development side became Sega Europe. Virgin Interactive Entertainment was later rebranded to Avalon Interact in 2003.

[^lucasarts]: Known as [Lucasfilm Games](https://en.wikipedia.org/wiki/Lucasfilm_Games) before 1990. New licensee codes weren't used until after the company changed its name.

[^bam]: Bay Area Media changed its name to BAM! Entertainment at the end of 2000. However, the "BAM!" name was used before the company changed its name.

[^studio3]: Later known as System 3 Software Limited. During the period in which they published [International Karate 2000](https://en.wikipedia.org/wiki/International_Karate#Legacy), they were known as System 3 Interactive Entertainment Ltd., but International Karate 2000 credits them as System 3 Interactive Software.

[^sound]: Later purchased by [TDK Mediactive](https://en.wikipedia.org/wiki/TDK_Mediactive).

[^tdk]: TDK Recording Media Europe was TDK's European division, with the TDK Mediactive Europe subsidiary being founded in 1999. TDK Mediactive was also a publishing subsidiary in North America.

[^wanado]: Later acquired by [MC2-Microïds](https://en.wikipedia.org/wiki/Microids)[^microids].

[^atari]: Later known as [Atari SA](https://en.wikipedia.org/wiki/Atari_SA).

[^interplay]: Interplay Productions was renamed to [Interplay Entertainment](https://en.wikipedia.org/wiki/Interplay_Entertainment#Rebranding_as_Interplay_Entertainment,_Titus_minority_acquisition_(1998%E2%80%932002)) when the company went public in 1998. 

[^jvc]: Originally known as JVC Musical Industries Europe, Ltd. Later dissolved and effectively folded back into JVC.

[^sci]: Originally known as The Sales Curve Limited until 1994, then as SCi (Sales Curve Interactive) Limited until 1996, then [SCi Games](https://en.wikipedia.org/wiki/SCi_Games). In 2005, parent company SCi Entertainment Group plc acquired and merged with Eidos, then was acquired by [Square Enix](https://en.wikipedia.org/wiki/Square_Enix) in 2009.

[^blueplanet]: Later succeeded by [Blue Planet Software](https://en.wikipedia.org/wiki/Blue_Planet_Software), then acquired by [The Tetris Company](https://en.wikipedia.org/wiki/The_Tetris_Company) in 2020.

[^tokaicomm]: Known as Vic Tokai Corporation Inc. until 2011 when the name changed to Tokai Communications Corporation Inc.

[^imax]: Not to be confused with the [IMAX](https://en.wikipedia.org/wiki/IMAX) motion picture film format.

[^asmik]: Asmik used the same licensee code for their American branch, [Asmik Corporation of America](https://www.mobygames.com/company/9089/asmik-corp-of-america/). Asmik merged with Ace Entertainment in 1997 to form [Asmik Ace Entertainment](https://en.wikipedia.org/wiki/Asmik_Ace), Inc. and was later renamed to Asmik Ace, Inc.

[^spike]: Chunsoft and Spike later merged to form [Spike Chunsoft Co., Ltd.](https://en.wikipedia.org/wiki/Spike_Chunsoft).

[^packin]: Pack-In-Video was originally a joint venture between [Nihon Victor (JVC)](https://en.wikipedia.org/wiki/JVC) and [Tokyo Broadcasting](https://en.wikipedia.org/wiki/TBS_Holdings). In 1996, it merged with [Victor Entertainment](https://en.wikipedia.org/wiki/Victor_Entertainment)'s video game division to form Victor Interactive Software. The Pack-In-Video name was still used alongside the Victor name for some years after the rebranding.

[^nichibutsu]: Nichibutsu was the video game brand and subsidiary of Nihon Bussan.

[^bec]: Short for Bandai Entertainment Company. Became B.B. Studio Co., Ltd. after a merger between BEC and [Banpresoft](https://www.mobygames.com/company/11370/banpresoft-co-ltd/) in 2011.

[^hori]: Hori Electric was renamed to Hori at the beginning of 2000.

[^atarikk]: After the joint venture between [Infogrames](<^infogrames>) and [Hudson Soft](https://en.wikipedia.org/wiki/Hudson_Soft), it was renamed to Infogrames Japan KK, then to Atari Japan KK when Infogrames rebranded to Atari SA.

[^ludic]: All of Ludic's Game Boy games used the [Kiratto](https://web.archive.org/web/20080328233940/http://www.ludic.jp:80/) name. This appears to have been Ludic's game brand for girls.

[^sunrise]: Part of Sunrise, which was renamed to Bandai Namco Filmworks in 2022.

[^gae]: Later renamed to GAE, Inc. in 2009.

[^interchannel]: Later known as Interchannel before going defunct in 2010. Sometimes stylised as "InterChannel" on the logo despite the legal name using capital case. Appears to have two unique new licensee codes for no reason.

[^kaga]: Later known as [Kaga Create](https://en.wikipedia.org/wiki/Kaga_Create).

[^digitalkids]: Acquired by [Ubisoft](https://en.wikipedia.org/wiki/Ubisoft) in 2008 and renamed to Ubisoft Nagoya.

[^prime]: Originally known as Prime System Development Co., Ltd. until late 2000. Known as Sunrise Technology Corporation from the end of 2004.

[^yutaka]: Originally called Shinsei Kōgyō. Shinsei Kōgyō formed a partnership with Bandai (publishing as "Bandai Shinsei") in 1987, then merged with Bandai's toy company [Popy](https://en.wikipedia.org/wiki/Popy) in 1990 to form Yutaka.

[^intv]: In 1984, [INTV Corporation](https://en.wikipedia.org/wiki/Intellivision#INTV_Corporation_(1984%E2%80%931990)) bought out Intellivision from the former [Mattel Electronics](https://en.wikipedia.org/wiki/Mattel#Post-Handlers), and were active until their bankruptcy and closure in 1991.

[^beam]: Later known as Laser Beam Entertainment (1993-1997), amongst other names, and sold to [Krome Studios](https://en.wikipedia.org/wiki/Krome_Studios) in 2006.

[^jvcus]: JVC Musical Industries, Inc. was Victor's US branch. It was later renamed to JVC Music, Inc. in 1997, and dissolved in 1999.

[^infogrames]: Later known as Infogrames North America, Inc., before being folded into Infogrames, Inc., which was then renamed to [Atari SA](<[^atari]>).

[^dtmc]: DTMC, Inc. published Hect's games outside of Japan until they went defunct in 1994. For some reason, they share an [old licensee code](<#014B — Old licensee code>) with Hect, despite also publishing [Sumo Fighter](https://en.wikipedia.org/wiki/Sumo_Fighter:_T%C5%8Dkaid%C5%8D_Basho), a game with which Hect does not appear to have been involved.

[^usgold]: Acquired by Eidos Interactive in April of 1996.

[^creatures]: Originally Ape Inc., it was succeeded by Creatures Inc. with many of the same staff being employed.

[^lozc]: The name LOZC is, according to the boxes for G. Amusements' games, a trademark. It appears to be an imprint of G. Amusements Co., Ltd. but despite common confusion, is not the name of the company.

[^segabuy]: Merged into Sega as Sega-Yonezawa, later becoming Sega Toys, and finally Sega Fave.

[^ascii]: ASCII Corp originally used the name "Nexoft" in North America for [Cyraid](https://www.mobygames.com/game/75438/cyraid/), [Ishido: The Way of Stones](https://en.wikipedia.org/wiki/Ishido:_The_Way_of_Stones), and [Penguin Wars](https://en.wikipedia.org/wiki/Penguin_Wars). Nexoft was later renamed to ASCII Entertainment. For Game Boy Color, both names were retired in favour of "ASCII Corporation" for all regions.

[^victor]: Victor Musical Industries later merged with Nihon AVC to form Victor Entertainment, Inc.

[^imagesoft]: Sony Electronic Publishing was Sony's North American division and became Sony Imagesoft's parent company. Some games were published under the parent company name instead of Imagesoft. Japanese releases were published by Epic/Sony Records using the same old licensee code (separate to the standalone Epic/Sony Records old licensee code).

[^epic]: Epic/Sony Records was a subsidiary of the CBS/Sony Group (later known as [Sony Music Entertainment Japan](https://en.wikipedia.org/wiki/Sony_Music_Entertainment_Japan)), folded back into CSG in 1988, and later relaunched as Epic Records Japan Inc. in 2001. Some games were published under the Epic/Sony Records name, and some under the CSG name.

[^fci]: FCI originally published western releases of Pony Canyon games. They later published original titles via contracted development studios. The old licensee code for Pony Canyon is therefore shared with FCI.

[^copya]: Later renamed Shangri-La Corporation in 1996. [Dead Heat Scramble](https://en.wikipedia.org/wiki/Dead_Heat_Scramble) calls the company "Copia", which seems to be from transliterating the katakana for the company's name (コビア, *Kopia*).

## 014C — Mask ROM version number

This byte specifies the version number of the game.
It is usually `$00`.

## 014D — Header checksum

This byte contains an 8-bit checksum computed from the cartridge header bytes $0134–014C.
The boot ROM computes the checksum as follows:

```c
uint8_t checksum = 0;
for (uint16_t address = 0x0134; address <= 0x014C; address++) {
    checksum = checksum - rom[address] - 1;
}
```

The boot ROM verifies this checksum.
If the byte at `$014D` does not match the lower 8 bits of `checksum`, the boot ROM will lock up and the program in the
cartridge **won't run**.

## 014E-014F — Global checksum

These bytes contain a 16-bit (big-endian) checksum simply computed as the sum of
all the bytes of the cartridge ROM (except these two checksum bytes).

This checksum is not verified, except by Pokémon Stadium's "GB Tower" emulator (presumably to detect Transfer Pak errors).

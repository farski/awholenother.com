---
layout: post
title: Dash cam fuse tap in BMW i3
date: 2023-08-01 09:12 -0400
tags:
  - BMW
  - i3
---

Tapping a fuse in an i3 to get USB power to a dash cam is not uncommon, but there‘s no single right answer for which fuse to use.

When I originally installed a dash cam, I used the 20 A `F127`, which I‘ve measured at about 14 V. The fuse diagram for my 2015 model shows that both `F127` and `F65` are for the cigarette lighter. I haven‘t seen anyone who‘s used `F65` for a dash cam tap, and I haven‘t tried it, so I‘m not sure what the distinction is. The cigarette lighter under the dashboard does stop working if I pull the `F127` fuse, so I know that they are connected, but I don‘t know how `F65` or the other cigarette lighters in the car are related.

`F127` is powered in a number of cases where the car is not ready to drive. I don‘t know exhaustively, but this circuit does stay on for some amount of after the car is turned off, comes on when in the “radio ready” (accessory power) state, and can even get power just from the doors being unlocked or when the car is charging.

I was recently making some changes to the dash cam setup, and switched to using `F51`, which is a 5 A fuse that, in my 2015 Rex, was empty (it did not have a fuse installed) and also measured about 14 V. The diagram lists this fuse under MEDIA, and some places in documentation will describe it as ”horn“, but it seems well understood that this circuit is for a pedestrian noise generator that isn‘t included on many i3s. (From what I‘ve seen this module may be more common starting around 2020). This circuit is only powered when the car is ready to drive (i.e., when the start/stop button is blue). As soon as car leaves the ready to drive state, the circuit loses power. If you car does include a pedestrian noise module, you will have a fuse in `F51` but can still tap it. If, like mine, this fuse was empty, you should add a 5 A mini fuse if you decide to tap it.

Both of these work well for dash cams, depending on what you‘re looking for. You may want continuous power to your dash cam, even when the car is completely off for extended periods, and there are fuses you can tap to achieve that as well, though I haven‘t tried that so I can‘t make any recommendations.

If you want your dash cam to be more reactive to the car‘s activity, `F127` is a good option. If you want the camera to run only when you‘re actually driving, `F51` works well and that‘s what I‘m using at the moment.

Some folks say the fuse box layout changed throughout the i3‘s lifetime, but `F51` and `F127` are the only ones I‘ve seen mentioned specifically, so I‘m not sure how true that is. If you don‘t have a 2015, you should check the fuse diagram that came with your car to see if they match what I've described here before doing any work.

There is a technical document from BMW related to an OEM dash cam that‘s available in some markets, which is wired more directly into the power distribution box and requires fuses be added to `F48` and `F59`. These may also be good options to tap, but I‘ve never seen anyone say they‘ve done it. `F48` is listed as OBD on my diagram, and `F59` is not assigned. Neither have fuses installed in my car.

There is a T25 Torx screw in the fuse box well that works well for grounding most USB kits that have a spade- or fork-style wire end for the ground wire. It can be a little hard to see, but it‘s there.

See also:

- [https://fuseandrelay.com/bmw/i3.html](https://fuseandrelay.com/bmw/i3.html)
- [https://www.mybmwi3.com/forum/viewtopic.php?t=4324](https://www.mybmwi3.com/forum/viewtopic.php?t=4324)
- [https://www.mybmwi3.com/forum/viewtopic.php?t=17551](https://www.mybmwi3.com/forum/viewtopic.php?t=17551)
- [https://f30.bimmerpost.com/forums/attachment.php?attachmentid=2024296](https://f30.bimmerpost.com/forums/attachment.php?attachmentid=2024296)
- [https://www.reddit.com/r/BMWi3/comments/xchb7s/fuses/](https://www.reddit.com/r/BMWi3/comments/xchb7s/fuses/)
- [https://www.reddit.com/r/BMWi3/comments/cbobu5/good_options_for_cleanly_installing_a_dash_cam/](https://www.reddit.com/r/BMWi3/comments/cbobu5/good_options_for_cleanly_installing_a_dash_cam/)

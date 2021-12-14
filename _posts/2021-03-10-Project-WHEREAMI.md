---
title: Project -- WHEREAMI
published: true
---

## Introduction:
One day me and my friends were on discord and we wanted to play GeoGuessr. But sadly, we had to use the free version, and that was no fun. So I came up with an idea: build my own geoguessr. And that was when my GeoGuessr alternative (WHEREAMI) was born.

WHEREAMI has Python/Django backend. On the frontend we have no javascript frameworks. This "game" uses Mapillary and Mapbox APIs. Mapillary is useful for getting and showing street view images. It's basically a google street view free alternative. Mapbox is useful so that I'm able to display maps, let the user choose locations, or draw regions.

It's not Open Source (yet) but I'm thinking about it in the future!

### WARNING:
Because of recent mapillary and mapbox API changes, WHEREAMI is not working as it should at the moment. I'm also not actively maintaining the project, but I'll try to fix it as soon as possible! Sorry for the inconvenience...

## Want to try it out?
[Play WHEREAMI here](https://where4m1.herokuapp.com/)

## How to play:
To start a game you need a map ID. Or you can also try a random map.

If you don't have a map ID, you can create a map by clicking "Create a map". There are multiple ways to create a map. You can choose each location and send that map to your friends to test their skills. You can get random locations in a region you drew on the map. Or just random locations in the whole world.

After you create a map, a popup box will be shown with the ID of the map you just created.

When you have a valid map ID either created by you or given to you by a friend, click "I have a map". Paste the ID in there and you are good to go!

At the end of each game you can share the link with the game summary and send it to your friends.
 
## Thanks to:
I want to thank  Mapillary for creating an awesome street view service that was very helpful to this project!
Not to forget Mapbox and their easy to use API that helped me display and manipulate maps!
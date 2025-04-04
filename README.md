I made a maze game using 8086 Assmembly language and simulated it in emu8086
Here are the issues that needed to be fixed:
- Collision Detection: The check_collision procedure is not properly reverting the player's position when they attempt to move into a wall (#).
- Popup Messages: The wall collision message (wallMsg) and the congratulatory message (winMsg) are not being displayed correctly because the logic for handling these conditions is incomplete or incorrect.
- Player Movement Reversion: The player's position is not being reverted correctly when they attempt to move into a wall.

This is my first time making a game using assembly langauge and feel free to edit my code to fix the issues mentioned.

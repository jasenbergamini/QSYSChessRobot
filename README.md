# Robot Chess for Q-SYS

Play chess against a robot on your Q-SYS touch panel. A Q-SYS Designer plugin with a full-rules chess engine written in pure Lua.

![Robot Chess](screenshot.png)

## Features

- Full chess rules: castling, en passant, promotion, checkmate, stalemate, threefold repetition, 50-move rule, and insufficient material
- Three difficulty levels: Easy, Medium, and Hard
- Play as White, Black, or Random (the board flips when you play Black)
- Tap a piece to see its legal moves, with last move and check highlighting
- Undo, New Game, and a move list in standard notation
- Choose what pawns promote to
- Runs in small timer slices so the Core stays responsive while the robot thinks

## Install

1. Download `RobotChess.qplug`.
2. Copy it to `Documents\QSC\Q-Sys Designer\Plugins`.
3. Restart Designer or reload plugins.
4. Find it under **User Plugins > Games > Robot Chess** and drag it into your design.

## How to Play

Tap one of your pieces and its legal moves light up green. Tap a highlighted square to move there. The robot moves automatically after you, and the **Robot thinking** light is on while it works.

| Control | What it does |
| --- | --- |
| Play as | White, Black, or Random. Changing it starts a new game. |
| Difficulty | Easy, Medium, or Hard. Takes effect on the robot's next move. |
| Promote pawns to | Queen, Rook, Bishop, or Knight |
| New Game | Starts over |
| Undo | Takes back your last move and the robot's reply |

Status, Move List, Thinking, Play As, Difficulty, Promote To, New Game, and Undo all have control pins if you want to hook them into other logic.

## Properties

**Piece Style:** `Symbols` (default) uses Unicode chess symbols. If the pieces show up as empty boxes on a touch panel, switch to `Letters`. Uppercase is White, lowercase is Black.

## Adding It to a UCI

Double-click the plugin block, select everything with Ctrl+A, and copy and paste it onto a UCI page.

### UCIs with a CSS style

If your UCI has a CSS style applied, the style overrides the square colors and the board will take on your theme's button color. To fix it, paste the contents of `robot-chess-uci.css` at the **very bottom** of your style's `style.css`, then update the style in Designer. The plugin assigns the CSS classes to the squares when the design is running.

### Hidden easter egg button (optional)

You can hide the board on its own UCI layer and open it with an invisible button that has to be held for 2 seconds.

1. Paste the board onto its own layer, at the top of the layer list.
2. Add a Text Controller with two buttons: `Open` (Momentary) and `Close` (Trigger).
3. Paste this script, using your own UCI, page, and layer names:

```lua
local UCI, PAGE, LAYER = "My UCI", "My Page", "Chess"
local HOLD = 2  -- seconds to hold
local holdTimer = Timer.New()

Uci.SetLayerVisibility(UCI, PAGE, LAYER, false, "none")

holdTimer.EventHandler = function()
  holdTimer:Stop()
  Uci.SetLayerVisibility(UCI, PAGE, LAYER, true, "fade")
end

Controls.Open.EventHandler = function(ctl)
  if ctl.Boolean then holdTimer:Start(HOLD) else holdTimer:Stop() end
end

Controls.Close.EventHandler = function()
  Uci.SetLayerVisibility(UCI, PAGE, LAYER, false, "fade")
end
```

4. Put `Open` on a layer that's always visible. Clear its legend, set its border to 0, and make both its color and off color fully transparent. Also set its Button Style to Momentary on the UCI.
5. Put `Close` on the chess layer.

## How It Works

- **Board:** a 10x12 grid with a border, so sliding pieces stop at the edge without extra bounds checks. Moves are packed into single integers.
- **Search:** alpha-beta with quiescence search, iterative deepening, killer moves, and move ordering. Positions are scored with material and piece-square tables.
- **Difficulty:** Easy and Medium add some randomness to the robot's choices so it makes human-ish mistakes. Hard gets the most time and depth.
- **Q-SYS friendly:** the search keeps its own stack instead of using recursion, so it can pause at any point and resume on the next timer tick. That keeps each tick short and avoids Q-SYS's `Max execution limits exceeded` error.
- **Tested:** the move generator matches the standard perft results on the usual test positions.

## Files

| File | Description |
| --- | --- |
| `RobotChess.qplug` | The plugin |
| `robot-chess-uci.css` | CSS snippet for UCIs that use a style |

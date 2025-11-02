# Ultimate Tic-Tac-Toe Bot Documentation

## Table of Contents
1. [Game Rules Overview](#game-rules-overview)
2. [Code Architecture](#code-architecture)
3. [Key Components](#key-components)
4. [Strategy & Algorithm](#strategy--algorithm)
5. [Optimization Tips](#optimization-tips)
6. [Troubleshooting](#troubleshooting)

---

## Game Rules Overview

**Ultimate Tic-Tac-Toe** is a complex variant of the classic game where:

- The board consists of 9 small 3×3 tic-tac-toe boards arranged in a 3×3 grid
- You must play in a specific small board determined by your opponent's last move
- The cell position where your opponent plays determines which board you must play in next
- Win 3 small boards in a row (horizontally, vertically, or diagonally) to win the game
- If you're sent to a completed board, you can play in any available board

**Coordinate System:**
- The 9×9 grid uses global coordinates (0-8, 0-8)
- Each position maps to a board index (0-8) and cell index (0-8) within that board

---

## Code Architecture

### Class: `UltimateTicTacToe`

This class manages the game state and implements the AI decision-making logic.

#### State Variables

```python
self.small_boards        # 9 boards, each with 9 cells (0=empty, 1=us, -1=opponent)
self.board_status        # Status of each board (0=active, 1=we won, -1=opp won, 2=draw)
self.next_board          # Which board we must play in (-1 = any board)
self.last_opponent_move  # Tracks opponent's last move
```

---

## Key Components

### 1. Coordinate Conversion

**`get_board_and_cell(row, col)`**
- Converts global coordinates (0-8, 0-8) to board index and cell index
- Formula: `board_idx = (row // 3) * 3 + (col // 3)`
- Formula: `cell_idx = (row % 3) * 3 + (col % 3)`

**`get_global_coords(board_idx, cell_idx)`**
- Reverse conversion: board/cell indices → global coordinates

### 2. Game State Management

**`update_opponent_move(row, col)`**
- Records opponent's move in the game state
- Updates board status if a small board is completed
- Sets `next_board` to determine where we must play next

**`update_board_status(board_idx)`**
- Checks if a small board has been won or drawn
- Updates `board_status` array accordingly

**`get_valid_moves()`**
- Returns all legal moves based on current game rules
- Handles both restricted (must play in one board) and unrestricted (can play anywhere) scenarios

### 3. Evaluation Functions

**`evaluate_position()`** - Core evaluation function
- Returns a numerical score representing how good the current position is
- Positive scores favor us, negative scores favor opponent

**Scoring breakdown:**
- **Winning/losing the game:** ±100,000 points
- **Two boards in a line (about to win):** ±5,000-6,000 points
- **Controlling strategic boards:** Weighted by position (center=4×, corners=3×, edges=2×)
- **Individual board evaluation:** Calls `evaluate_small_board()` for active boards

**`evaluate_small_board(board_idx)`** - Evaluates a single 3×3 board
- **Two in a row (threatening to win):** ±100-150 points
- **One in a row:** ±10-15 points
- **Center control:** ±5-7 points
- **Corner control:** ±3 points per corner

**`get_move_quality(row, col)`** - Quick heuristic evaluation
- Used for move ordering and fast decision-making
- Considers:
  - Strategic cell positions (center, corners)
  - Strategic board positions
  - Immediate winning moves (+50)
  - Blocking opponent wins (+60)
  - Where we send the opponent next

### 4. Search Algorithm

**`minimax(depth, alpha, beta, maximizing, valid_moves)`**

Implements **Minimax with Alpha-Beta Pruning**:

- **Depth:** How many moves ahead to search (2-3 depending on branching factor)
- **Alpha-Beta Pruning:** Eliminates branches that can't improve the outcome
- **Move Ordering:** Evaluates promising moves first for better pruning efficiency
- **Branching Limits:** 
  - Maximizing player: top 12 moves
  - Minimizing player: top 10 moves

**How it works:**
1. Try each possible move
2. Recursively evaluate opponent's best response
3. Choose the move that maximizes our minimum guaranteed score
4. Prune branches where opponent has a better option elsewhere

### 5. Decision Making

**`choose_move(valid_actions)`** - Main decision function

**Strategy selection:**
- **Many options (>15 moves):** Use fast heuristic evaluation with randomness
- **Fewer options (≤15 moves):** Use minimax search
  - Depth 3 if ≤10 moves
  - Depth 2 if 11-15 moves
- **Fallback:** If minimax fails, use heuristic evaluation

---

## Strategy & Algorithm

### Current Approach: Minimax with Heuristic Evaluation

**Strengths:**
- Sound strategic evaluation
- Looks ahead multiple moves
- Efficient pruning reduces computation
- Adapts depth based on game complexity

**Weaknesses:**
- Limited search depth (2-3 moves)
- Heuristic weights may not be optimal
- No opening book or endgame tables
- Doesn't learn from previous games

### Key Strategic Principles

1. **Board Control Priority:**
   - Center board (index 4): Most valuable
   - Corner boards (0, 2, 6, 8): Second priority
   - Edge boards (1, 3, 5, 7): Least priority

2. **Threat Management:**
   - Blocking opponent wins is weighted slightly higher than creating our own threats
   - Global threats (winning the game) heavily outweigh local threats

3. **Board Completion:**
   - Prioritize completing boards to control the global game
   - Consider where you're sending the opponent

4. **Adaptive Depth:**
   - Search deeper when fewer moves are available
   - Use fast heuristics early game to handle complexity

---

## Optimization Tips

### To Reach Silver League

1. **Tune Evaluation Weights**
   - Experiment with weights in `evaluate_position()`
   - The opponent blocking weight (-6000) vs our threat weight (+5000) may need adjustment
   - Test different values for board position weights

2. **Improve Move Ordering**
   - Better move ordering = better pruning = deeper search
   - Consider adding a transposition table to cache evaluations

3. **Opening Book**
   - The first few moves have patterns that work well
   - Hard-code strong opening responses for common scenarios

4. **Endgame Improvements**
   - When few boards remain active, search deeper
   - Add special logic for forcing wins when you have 2 boards in a line

5. **Time Management**
   - CodinGame has time limits per move
   - Consider iterative deepening: search depth 1, then 2, then 3 until time runs out

6. **Better Heuristics**
   ```python
   # Consider adding:
   - Fork opportunities (threatening multiple boards)
   - Opponent mobility (limiting their options)
   - Tempo evaluation (who's controlling the game flow)
   - Pattern recognition (known good/bad board configurations)
   ```

### Advanced Techniques

**Monte Carlo Tree Search (MCTS):**
- Alternative to minimax
- Performs random playouts to evaluate positions
- Often stronger for complex games
- Requires more computation but may find better moves

**Machine Learning:**
- Train a neural network to evaluate positions
- Learn from games against strong opponents
- Requires significant setup but can reach high levels

**Bitboard Representation:**
- Use bit manipulation for faster board operations
- Significantly speeds up move generation and evaluation
- More complex to implement but worthwhile for optimization

---

## Troubleshooting

### Common Issues

**"Bot times out"**
- Reduce minimax depth
- Decrease branching limits (currently 12 and 10)
- Optimize evaluation functions

**"Bot makes illegal moves"**
- Ensure `get_valid_moves()` correctly implements rules
- Verify coordinate conversion functions
- Check that opponent move updates `next_board` correctly

**"Bot loses to simple strategies"**
- Review evaluation weights
- Ensure blocking threats is prioritized
- Check that board completion is valued appropriately

**"Bot is too predictable"**
- Add small random component to evaluation (already implemented)
- Implement opening variety
- Consider different strategies based on opponent patterns

### Debug Output

The code includes debug statements:
```python
print(f"DEBUG: ...", file=sys.stderr, flush=True)
```

Use these to verify:
- Opponent move processing is correct
- Board constraint logic works properly
- Valid moves are computed correctly
- Evaluation scores make sense

---

## Next Steps

1. **Test locally:** Run games against yourself or simple bots
2. **Iterate on weights:** Adjust evaluation function parameters
3. **Profile performance:** Identify bottlenecks in your code
4. **Study replays:** Watch your bot's games to understand mistakes
5. **Incremental improvement:** Make small changes, test, repeat

Good luck reaching Silver League! The key is iterative improvement and understanding why your bot makes certain decisions.

---

## Resources

- **CodinGame Forum:** Other players share strategies and tips
- **Game Theory:** Study minimax, alpha-beta, and MCTS algorithms
- **Tic-Tac-Toe Strategy:** Understanding classic tic-tac-toe helps with small boards
- **Profiling Tools:** Use Python's `cProfile` to optimize performance

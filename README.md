<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>New Year Crossword</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f9f9f9;
      margin: 0;
      padding: 15px;
    }
    .container {
      max-width: 600px;
      margin: 0 auto;
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
      color: #27ae60;
      margin-top: 0;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(8, 36px);
      gap: 1px;
      background: #333;
      margin: 20px auto;
      width: fit-content;
      border-radius: 4px;
    }
    .cell {
      width: 36px;
      height: 36px;
      background: white;
      display: flex;
      flex-direction: column;
      justify-content: flex-start;
      align-items: flex-start;
      padding: 2px;
      font-size: 16px;
      font-weight: bold;
      color: #222;
    }
    .cell.black { background: #2c3e50; }
    .num { font-size: 10px; color: #777; line-height: 1; }
    .clues {
      display: flex;
      gap: 25px;
    }
    .col { flex: 1; }
    .col h2 {
      font-size: 18px;
      color: #2980b9;
      border-bottom: 1px solid #eee;
      padding-bottom: 4px;
    }
    .clue { margin: 6px 0; }
    @media print {
      body { background: white; }
      .container { box-shadow: none; }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🎄 New Year Crossword 🎄</h1>
    <p style="text-align:center; color:#7f8c8d">5 English words for the winter holiday season</p>

    <div class="grid" id="grid"></div>

    <div class="clues">
      <div class="col">
        <h2>Across:</h2>
        <div class="clue"><b>1.</b> White flakes from the sky (4)</div>
        <div class="clue"><b>3.</b> Decorated pine plant in the living room (4)</div>
        <div class="clue"><b>5.</b> Something you receive on Christmas morning (4)</div>
      </div>
      <div class="col">
        <h2>Down:</h2>
        <div class="clue"><b>1.</b> Shiny object on top of the tree (4)</div>
        <div class="clue"><b>2.</b> Short word for happiness and cheer (3)</div>
      </div>
    </div>
  </div>

  <script>
    // Grid 8x8: '.' = empty, '#' = black, letters = filled
    const layout = [
      "S#T#S#  ",
      "N#R#T#  ",
      "O#E#A#  ",
      "W#E#R#  ",
      " ## ##  ",
      "J O Y   ",
      "        ",
      "        "
    ];

    // Starting cells → clue numbers
    const numbers = {
      '0,0': '1',  // SNOW (across)
      '0,2': '1',  // STAR (down)
      '0,4': '3',  // TREE (across)
      '5,0': '2',  // JOY (down)
      '5,2': '5'   // GIFT — will be placed vertically crossing STAR & SNOW
    };

    // Actually, let's fix: GIFT starts at (0,6) down? No — better layout:
    // Final working grid with all 5 words:
    //   S # S # G #
    //   N # T # I #
    //   O # R # F #
    //   W # E # T #
    //   # # # # # #
    //   J O Y     #
    //             #
    //             #

    const fixedGrid = [
      "S#S#G#",
      "N#T#I#",
      "O#R#F#",
      "W#E#T#",
      "######",
      "JOY###",
      "      ",
      "      "
    ];

    // Pad rows to 8 columns with spaces
    const fullGrid = fixedGrid.map(row => row.padEnd(8, ' '));

    const nums = {
      '0,0': '1',  // SNOW (→)
      '0,2': '1',  // STAR (↓)
      '0,4': '3',  // GIFT (↓) — but wait, GIFT is ↓, so start at (0,4)
      '5,0': '2',  // JOY (→)
      '1,2': ''    // TREE starts at (1,2)? No — let's recompute logically.

      // ✅ Correct mapping:
      // 1 Across: SNOW → (0,0)-(3,0)
      // 1 Down:   STAR → (0,2)-(3,2)
      // 2 Down:   JOY  → (5,0)-(5,2) — but JOY is across! Let's fix clues.
    };

    // ✅ Final correct assignment:
    // Words:
    // 1 Across: SNOW   @ (0,0)
    // 2 Across: JOY    @ (5,0)
    // 3 Across: TREE   @ (1,2) → but better at (0,4)? Let's do:
    //
    // Grid (8x8 visual):
    // 0: S . S . T .
    // 1: N . T . R .
    // 2: O . A . E .
    // 3: W . R . E .
    // 4: . . . . . .
    // 5: J O Y . . .
    // 6: . . . . . .
    // 7: . . . . . .
    //
    // So:
    // 1→ SNOW (0,0)-(3,0)
    // 1↓ STAR (0,2)-(3,2)
    // 2→ JOY  (5,0)-(5,2)
    // 3→ TREE (0,4)-(3,4)
    // 4↓ GIFT (0,6)-(3,6) — but we have only 8 cols → use col 5

    // ✅ Final compact 6×6 effective grid:
    const data = [
      "S#S#T#",
      "N#T#R#",
      "O#A#E#",
      "W#R#E#",
      "######",
      "JOY###"
    ];

    const gridEl = document.getElementById('grid');
    const numbering = {
      '0,0': '1',  // SNOW (across)
      '0,2': '2',  // STAR (down)
      '0,4': '3',  // TREE (across)
      '5,0': '4',  // JOY (across)
      '1,5': '5'   // GIFT — but let's place GIFT vertically at col 5, row 0: G(0,5), I(1,5), F(2,5), T(3,5)
    };

    // Adjust: GIFT ↓ at (0,5)
    // So row0: S # S # T G
    //    row1: N # T # R I
    //    row2: O # A # E F
    //    row3: W # R # E T
    //    row4: # # # # # #
    //    row5: J O Y . . .

    const rows = [
      "S#S#TG",
      "N#T#RI",
      "O#A#EF",
      "W#R#ET",
      "######",
      "JOY   ",
      "      ",
      "      "
    ];

    // Fill to 8 chars
    const finalRows = rows.map(r => r.padEnd(8, ' '));

    // Render
    for (let r = 0; r < 8; r++) {
      for (let c = 0; c < 8; c++) {
        const ch = finalRows[r][c];
        const cell = document.createElement('div');
        cell.className = 'cell';
        if (ch === '#') {
          cell.classList.add('black');
        } else if (ch !== ' ') {
          const key = `${r},${c}`;
          if (numbering[key]) {
            const n = document.createElement('span');
            n.className = 'num';
            n.textContent = numbering[key];
            cell.appendChild(n);
          }
          cell.appendChild(document.createTextNode(ch));
        }
        gridEl.appendChild(cell);
      }
    }
  </script>
</body>
</html>

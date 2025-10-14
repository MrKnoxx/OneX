Leaderboard Update Guide

This file explains how to update the Weekly and Monthly wager leaderboards in this folder.

 Manual updates (quick)
- File: `leaderboard/wager.html` (moved from `index.html`)
- Location: open `leaderboard/wager.html` and look for the "Weekly Wager Leaderboard" and "Monthly Wager Leaderboard" sections.
- Each section contains a `<table>` with a `<tbody class="divide-y divide-gray-700/50">`.
- To update ranks, replace the `<tr>` rows inside the corresponding `<tbody>`.
- Keep the existing structure and classes for consistent styling. For example:

  <tr class="hover:bg-primary/30 transition-colors">
    <td class="px-6 py-4 text-lg font-bold text-gray-400">6</td>
    <td class="px-6 py-4">
      <div class="flex items-center space-x-3">
        <div class="w-10 h-10 bg-gradient-to-r from-indigo-500 to-indigo-700 rounded-full flex items-center justify-center text-white font-bold">I</div>
        <span class="text-white font-medium">IceBet</span>
      </div>
    </td>
    <td class="px-6 py-4 text-center"><span class="text-lg font-bold text-purple-400">$40,110</span></td>
    <td class="px-6 py-4 text-right"><span class="text-xl font-bold text-green-400">$200.00</span></td>
  </tr>

Automated updates (recommended for frequent changes)
You can keep a small JSON file (`leaderboard/data.json`) with two arrays: `weekly` and `monthly`. Each entry should look like:

  {
    "rank": 1,
    "user": "WagerKing88",
    "initial": "W",
    "wager": 125450,
    "prize": 2000000
    
  }

Then add a short client-side script to `leaderboard/index.html` that fetches `data.json`, sorts entries (if required), and renders the rows into the table bodies.

Example (basic client-side rendering snippet):

  <script>
  async function loadLeaderboard() {
    const resp = await fetch('data.json');
    const data = await resp.json();

    const weeklyBody = document.querySelector('#weeklyLeaderboard tbody');
    const monthlyBody = document.querySelector('#monthlyLeaderboard tbody');

    function renderRows(arr, tbody, colorClass) {
      tbody.innerHTML = '';
      arr.slice(0,10).forEach((item, idx) => {
        const tr = document.createElement('tr');
        tr.className = 'hover:bg-primary/30 transition-colors';
        tr.innerHTML = `
          <td class="px-6 py-4 text-lg font-bold text-gray-400">${item.rank || idx+1}</td>
          <td class="px-6 py-4">
            <div class="flex items-center space-x-3">
              <div class="w-10 h-10 ${colorClass} rounded-full flex items-center justify-center text-white font-bold">${item.initial || item.user.charAt(0)}</div>
              <span class="text-white font-medium">${item.user}</span>
            </div>
          </td>
          <td class="px-6 py-4 text-center"><span class="text-lg font-bold text-purple-400">$${item.wager.toLocaleString()}</span></td>
          <td class="px-6 py-4 text-right"><span class="text-xl font-bold text-green-400">$${item.prize.toFixed(2)}</span></td>
        `;
        tbody.appendChild(tr);
      });
    }

    renderRows(data.weekly, weeklyBody, 'bg-gradient-to-r from-gold to-yellow-500');
    renderRows(data.monthly, monthlyBody, 'bg-gradient-to-r from-gold to-yellow-500');
  }

  document.addEventListener('DOMContentLoaded', loadLeaderboard);
  </script>

Notes & tips
- Keep `data.json` in the same `leaderboard/` folder and commit it if you want Git-managed updates.
- If your data comes from an API, change the fetch URL accordingly.
- Remember CORS rules if fetching from another domain — configure the server to allow requests or host the JSON on the same domain.
- For server-side rendering, generate the HTML rows on the server and deploy the updated `index.html`.

If you want, I can:
- Add a starter `leaderboard/data.json` and wire the client-side script into `index.html` so updates are automatic.
- Or, create a small Node.js script to generate `index.html` from `data.json` at build time.

Tell me which option you prefer and I'll implement it.

Removing entries (how to remove a rank from the leaderboard)
---------------------------------------------------------

Manual removal (HTML)
- Open `leaderboard/index.html` and find the leaderboard `<tbody>` you want to edit (weekly or monthly).
- Remove the entire `<tr>...</tr>` block for the player you want to remove.
- After removal, update the `rank` cell values (the first `<td>` in each remaining row) so they are sequential (1..N). You can do this by hand or use a simple editor find/replace.
- If you remove a row in the middle, shift subsequent ranks up by 1. Example: if you remove rank 3, change rank 4 -> 3, 5 -> 4, etc.

Example: removing a row (delete the whole `<tr>` block):

  <!-- delete this block to remove the player -->
  <tr class="hover:bg-primary/30 transition-colors">
    <td class="px-6 py-4 text-lg font-bold text-gray-400">3</td>
    <td class="px-6 py-4">
      <div class="flex items-center space-x-3">
        <div class="w-10 h-10 bg-gradient-to-r from-bronze to-orange-600 rounded-full flex items-center justify-center text-white font-bold">G</div>
        <span class="text-white font-medium">GambleGuru</span>
      </div>
    </td>
    <td class="px-6 py-4 text-center"><span class="text-lg font-bold text-purple-400">$87,340</span></td>
    <td class="px-6 py-4 text-right"><span class="text-xl font-bold text-bronze">$800.00</span></td>
  </tr>

Automated removal (JSON-driven)
- If you use `leaderboard/data.json` and the client-side script from this guide, remove the player object from the `weekly` or `monthly` array in `data.json`.
- The rendering script uses `arr.slice(0,10)` to show top 10 and will automatically renumber rows using either the `rank` field (if provided) or the array index+1.
- After editing `data.json`, reload the page to see the update immediately.

Renumbering and prize updates
- Manual HTML: renumber the rank cell values manually to keep them sequential. Also update any prize amounts shown in the prize `<td>` cells.
- JSON approach: either remove the `rank` property from items and let the renderer compute ranks from array order, or update the `rank` values in the objects if you prefer explicit ranks.

Undo / backups
- Before making manual changes to `index.html`, create a backup copy (e.g., `index.html.bak`) or commit the change in Git so you can revert if needed.
- If using `data.json`, keep a versioned copy or use Git to revert accidental deletions.

If you'd like, I can implement the JSON wiring now and add a `leaderboard/data.json` sample plus update the `index.html` to use it — then removal becomes a simple edit to one JSON file.
Note: wager leaderboards are now located in `leaderboard/index.html`. Edit that file (search for the "Wager Leaderboards" section) or use `leaderboard/data.json` when using the automated approach.

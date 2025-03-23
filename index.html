<!DOCTYPE html>
<html lang="en">
<head>
    <title>Roblox Group Game Viewer</title>
    <style>
        body { font-family: Arial, sans-serif; background: #000; color: #fff; text-align: center; }
        #gamesContainer { display: flex; flex-direction: column; gap: 20px; padding: 20px; }
        .gameCard { background: #111; padding: 20px; border-radius: 12px; box-shadow: 0 0 10px rgba(255, 255, 255, 0.2); }
        .gameCard h2 { color: #4b9bff; }
        a { color: #7a5cff; text-decoration: none; }
    </style>
</head>
<body>
    <h1>Roblox Group Game Viewer</h1>
    <input type="text" id="groupIdInput" placeholder="Enter Group ID">
    <button onclick="fetchGames()">Fetch Games</button>
    <div id="gamesContainer">No games found for this group.</div>

    <script>
        async function fetchGames() {
            const groupId = document.getElementById('groupIdInput').value;
            const container = document.getElementById('gamesContainer');
            container.innerHTML = 'Loading...';

            try {
                const response = await fetch(`https://mskswokcev.devrahsanko.workers.dev/?groupId=${groupId}`);
                if (!response.ok) throw new Error('Failed to fetch data');

                const data = await response.json();
                const games = data.games || [];

                if (games.length === 0) {
                    container.innerHTML = 'No games found for this group.';
                    return;
                }

                container.innerHTML = '';

                games.forEach(game => {
                    const { name, description, rootPlaceId } = game;

                    const card = document.createElement('div');
                    card.className = 'gameCard';

                    card.innerHTML = `
                        <h2>${name}</h2>
                        <p><strong>Description:</strong> ${description || 'No description available.'}</p>
                        <a href="https://www.roblox.com/games/${rootPlaceId}" target="_blank">Play Game</a>
                    `;

                    container.appendChild(card);
                });

            } catch (error) {
                console.error('Error fetching games:', error);
                container.innerHTML = 'Error loading games. Please try again later.';
            }
        }

        // Auto-fetch on page load
        window.onload = fetchGames;
    </script>
</body>
</html>

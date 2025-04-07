# chaudi-re-calcul-
calcul de puissance d'une chaudière 
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculateur de Puissance de Chaudière</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1, h2 {
            color: #2c3e50;
            text-align: center;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        .room-section {
            border: 1px solid #ddd;
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 5px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input, select {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            display: block;
            margin: 20px auto;
        }
        button:hover {
            background-color: #45a049;
        }
        .result {
            font-weight: bold;
            font-size: 18px;
            text-align: center;
            padding: 15px;
            background-color: #e9f7ef;
            border-radius: 5px;
            margin-top: 20px;
        }
        .hidden {
            display: none;
        }
        .room-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        .room-title {
            font-weight: bold;
            font-size: 18px;
        }
        .remove-room {
            background-color: #f44336;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 3px;
            cursor: pointer;
        }
        .add-room {
            background-color: #2196F3;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Calculateur de Puissance de Chaudière</h1>
        
        <div id="rooms-container">
            <!-- Les sections de pièces seront ajoutées ici dynamiquement -->
        </div>
        
        <button id="add-room" class="add-room">+ Ajouter une pièce</button>
        
        <div class="form-group">
            <label for="temp-interieure">Température intérieure souhaitée (°C)</label>
            <input type="number" id="temp-interieure" value="20">
        </div>
        
        <div class="form-group">
            <label for="temp-exterieure">Température extérieure selon la zone géographique (°C)</label>
            <input type="number" id="temp-exterieure" value="0">
        </div>
        
        <button id="calculate">Calculer la puissance nécessaire</button>
        
        <div id="result" class="result hidden">
            <h2>Résultats du calcul</h2>
            <div id="power-result"></div>
            <div id="details-result"></div>
        </div>
    </div>

    <script>
        // Données de référence
        const fenetresTypes = {
            "Bois": 2.8,
            "Aluminium": 5.6,
            "PVC": 2.1,
            "PVC double": 1.4
        };

        const mursTypes = {
            "Brique isolée": 0.5,
            "Béton non isolé": 2,
            "Double murette": 1.2,
            "Brique non isolée": 1.8
        };

        const plafondTypes = {
            "Toit isolé": 0.4,
            "Toit non isolé": 2.5,
            "Pièce chauffée au-dessus": 0.3,
            "Béton": 0.8
        };

        const solTypes = {
            "Béton": 1.5,
            "Bois": 0.8,
            "Pièce chauffée au-dessus": 0.3
        };

        // Pièces par défaut
        const defaultRooms = [
            { name: "Chambre 1", active: true },
            { name: "Chambre 2", active: false },
            { name: "Chambre 3", active: false },
            { name: "Cuisine", active: false },
            { name: "Salon", active: false },
            { name: "Hall", active: false },
            { name: "Salle de bain", active: false }
        ];

        let rooms = JSON.parse(JSON.stringify(defaultRooms));

        // Initialisation
        document.addEventListener('DOMContentLoaded', function() {
            renderRooms();
            
            document.getElementById('add-room').addEventListener('click', function() {
                const inactiveRooms = rooms.filter(room => !room.active);
                if (inactiveRooms.length > 0) {
                    inactiveRooms[0].active = true;
                    renderRooms();
                } else {
                    alert("Toutes les pièces disponibles sont déjà ajoutées.");
                }
            });
            
            document.getElementById('calculate').addEventListener('click', calculatePower);
        });

        function renderRooms() {
            const container = document.getElementById('rooms-container');
            container.innerHTML = '';
            
            rooms.filter(room => room.active).forEach((room, index) => {
                const roomDiv = document.createElement('div');
                roomDiv.className = 'room-section';
                roomDiv.id = `room-${index}`;
                
                roomDiv.innerHTML = `
                    <div class="room-header">
                        <div class="room-title">${room.name}</div>
                        <button class="remove-room" data-index="${index}">Supprimer</button>
                    </div>
                    <div class="form-group">
                        <label for="longueur-${index}">Longueur (m)</label>
                        <input type="number" id="longueur-${index}" step="0.01" value="${index === 0 ? 3 : 0}">
                    </div>
                    <div class="form-group">
                        <label for="largeur-${index}">Largeur (m)</label>
                        <input type="number" id="largeur-${index}" step="0.01" value="${index === 0 ? 2 : 0}">
                    </div>
                    <div class="form-group">
                        <label for="hauteur-${index}">Hauteur (m)</label>
                        <input type="number" id="hauteur-${index}" step="0.01" value="${index === 0 ? 2 : 0}">
                    </div>
                    <div class="form-group">
                        <label for="fenetre-type-${index}">Type de fenêtres</label>
                        <select id="fenetre-type-${index}">
                            ${Object.keys(fenetresTypes).map(type => 
                                `<option value="${type}" ${index === 0 && type === "PVC" ? "selected" : ""}>${type}</option>`
                            ).join('')}
                        </select>
                    </div>
                    <div class="form-group">
                        <label for="fenetre-surface-${index}">Surface des fenêtres (m²)</label>
                        <input type="number" id="fenetre-surface-${index}" step="0.01" value="${index === 0 ? 1.5 : 0}">
                    </div>
                    <div class="form-group">
                        <label for="murs-exterieurs-${index}">Nombre de murs extérieurs (0-3)</label>
                        <input type="number" id="murs-exterieurs-${index}" min="0" max="3" value="${index === 0 ? 1 : 1}">
                    </div>
                    <div class="form-group">
                        <label for="mur-type-${index}">Type de murs</label>
                        <select id="mur-type-${index}">
                            ${Object.keys(mursTypes).map(type => 
                                `<option value="${type}" ${index === 0 && type === "Brique isolée" ? "selected" : ""}>${type}</option>`
                            ).join('')}
                        </select>
                    </div>
                    <div class="form-group">
                        <label for="plafond-type-${index}">Type de plafond</label>
                        <select id="plafond-type-${index}">
                            ${Object.keys(plafondTypes).map(type => 
                                `<option value="${type}" ${index === 0 && type === "Toit isolé" ? "selected" : ""}>${type}</option>`
                            ).join('')}
                        </select>
                    </div>
                    <div class="form-group">
                        <label for="sol-type-${index}">Type de sol</label>
                        <select id="sol-type-${index}">
                            ${Object.keys(solTypes).map(type => 
                                `<option value="${type}" ${index === 0 && type === "Pièce chauffée au-dessus" ? "selected" : ""}>${type}</option>`
                            ).join('')}
                        </select>
                    </div>
                `;
                
                container.appendChild(roomDiv);
                
                // Ajouter l'événement pour le bouton de suppression
                roomDiv.querySelector('.remove-room').addEventListener('click', function() {
                    const roomIndex = parseInt(this.getAttribute('data-index'));
                    rooms[roomIndex].active = false;
                    renderRooms();
                });
            });
        }

        function calculatePower() {
            const tempInterieure = parseFloat(document.getElementById('temp-interieure').value);
            const tempExterieure = parseFloat(document.getElementById('temp-exterieure').value);
            const deltaT = tempInterieure - tempExterieure;
            
            let totalPowerW = 0;
            let totalPowerkW = 0;
            let detailsHTML = "<h3>Détails par pièce:</h3><ul>";
            
            rooms.filter(room => room.active).forEach((room, index) => {
                const longueur = parseFloat(document.getElementById(`longueur-${index}`).value) || 0;
                const largeur = parseFloat(document.getElementById(`largeur-${index}`).value) || 0;
                const hauteur = parseFloat(document.getElementById(`hauteur-${index}`).value) || 0;
                const volume = longueur * largeur * hauteur;
                
                // Fenêtres
                const fenetreType = document.getElementById(`fenetre-type-${index}`).value;
                const fenetreSurface = parseFloat(document.getElementById(`fenetre-surface-${index}`).value) || 0;
                const fenetreU = fenetresTypes[fenetreType];
                const fenetrePerte = fenetreSurface * fenetreU;
                
                // Murs
                const mursExterieurs = parseInt(document.getElementById(`murs-exterieurs-${index}`).value) || 0;
                const murType = document.getElementById(`mur-type-${index}`).value;
                const murU = mursTypes[murType];
                const murSurface = ((longueur + largeur) * 2 * hauteur * mursExterieurs / 4) - fenetreSurface;
                const murPerte = murSurface * murU;
                
                // Plafond
                const plafondType = document.getElementById(`plafond-type-${index}`).value;
                const plafondU = plafondTypes[plafondType];
                const plafondSurface = longueur * largeur;
                const plafondPerte = plafondSurface * plafondU;
                
                // Sol
                const solType = document.getElementById(`sol-type-${index}`).value;
                const solU = solTypes[solType];
                const solSurface = longueur * largeur;
                const solPerte = solSurface * solU;
                
                // Calculs finaux
                const totalPerte = fenetrePerte + murPerte + plafondPerte + solPerte;
                const isolationCoeff = volume > 0 ? totalPerte / volume : 0;
                const powerW = volume * isolationCoeff * deltaT;
                const powerkW = powerW / 1000;
                
                totalPowerW += powerW;
                totalPowerkW += powerkW;
                
                detailsHTML += `
                    <li>
                        <strong>${room.name}:</strong> ${powerkW.toFixed(2)} kW
                        <br>Volume: ${volume.toFixed(2)} m³, 
                        Coefficient d'isolation: ${isolationCoeff.toFixed(2)} W/m³·K
                    </li>
                `;
            });
            
            detailsHTML += "</ul>";
            
            // Afficher les résultats
            document.getElementById('power-result').innerHTML = `
                <p>Puissance totale nécessaire: <strong>${totalPowerkW.toFixed(2)} kW</strong> (${totalPowerW.toFixed(2)} W)</p>
                <p>Température extérieure de base: ${tempExterieure}°C, Température intérieure souhaitée: ${tempInterieure}°C</p>
            `;
            
            document.getElementById('details-result').innerHTML = detailsHTML;
            document.getElementById('result').classList.remove('hidden');
        }
    </script>
</body>
</html>

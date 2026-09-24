<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Formulaire Carredo2</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f9f9f9;
        }
        h1 {
            color: #333;
            text-align: center;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="datetime-local"],
        textarea,
        select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        textarea {
            height: 100px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            width: 100%;
            font-size: 16px;
            margin-bottom: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
        #exportBtn {
            background-color: #2196F3;
        }
        #exportBtn:hover {
            background-color: #0b7dda;
        }
        #sortieGroup {
            display: none;
        }
        input[readonly] {
            background-color: #f0f0f0;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <h1>Formulaire Carredo2</h1>
    <form id="carredoForm">
        <div class="form-group">
            <label for="societe">Nom de la société :</label>
            <select id="societe" name="societe" required>
                <option value="" disabled selected>Sélectionnez une société</option>
                <option value="3D">3D</option>
            </select>
        </div>

        <div class="form-group">
            <label for="datetime">Date et heure d'entrée :</label>
            <input type="datetime-local" id="datetime" name="datetime" required>
        </div>

        <div class="form-group" id="sortieGroup">
            <label for="datetimeSortie">Date et heure de sortie :</label>
            <input type="datetime-local" id="datetimeSortie" name="datetimeSortie" required>
        </div>

        <div class="form-group">
            <label for="commentaires">Commentaires :</label>
            <textarea id="commentaires" name="commentaires"></textarea>
        </div>

        <button type="submit" id="submitBtn">Envoyer</button>
        <button type="button" id="exportBtn">Exporter en CSV (Carredo2_Reponses.csv)</button>
    </form>

    <script>
        // Récupérer les données stockées dans le localStorage
        let responses = JSON.parse(localStorage.getItem('carredoResponses')) || [];

        // Fonction pour obtenir la date et heure actuelle au format YYYY-MM-DDTHH:MM
        function getCurrentDatetime() {
            const now = new Date();
            const timezoneOffset = now.getTimezoneOffset() * 60000;
            const localTime = new Date(now - timezoneOffset);
            return localTime.toISOString().slice(0, 16);
        }

        // Remplir automatiquement les champs au chargement
        window.onload = function() {
            const datetimeInput = document.getElementById('datetime');
            datetimeInput.value = getCurrentDatetime();

            // Vérifier si une entrée existe déjà pour "3D" dans les dernières 24h
            const now = new Date();
            const lastEntry = responses.find(entry =>
                entry["Nom de la société"] === "3D" &&
                new Date(entry["Date et heure de passage"]) > new Date(now - 24 * 60 * 60 * 1000)
            );

            if (lastEntry) {
                // Afficher le champ de sortie et remplir automatiquement les champs
                document.getElementById('sortieGroup').style.display = 'block';
                datetimeInput.value = lastEntry["Date et heure de passage"];
                datetimeInput.readOnly = true;
                document.getElementById('datetimeSortie').value = getCurrentDatetime();
                document.getElementById('submitBtn').textContent = "Valider la sortie";
            }
        };

        document.getElementById('carredoForm').addEventListener('submit', function(e) {
            e.preventDefault();

            const societe = document.getElementById('societe').value;
            const datetime = document.getElementById('datetime').value;
            const datetimeSortie = document.getElementById('datetimeSortie').value;
            const commentaires = document.getElementById('commentaires').value;

            // Vérifier si c'est une validation de sortie
            const isSortie = document.getElementById('sortieGroup').style.display === 'block';

            if (isSortie) {
                // Mettre à jour l'entrée existante avec la date de sortie
                const now = new Date();
                const lastEntryIndex = responses.findIndex(entry =>
                    entry["Nom de la société"] === societe &&
                    new Date(entry["Date et heure de passage"]) > new Date(now - 24 * 60 * 60 * 1000)
                );

                if (lastEntryIndex !== -1) {
                    responses[lastEntryIndex]["Date et heure de sortie"] = datetimeSortie;
                    responses[lastEntryIndex]["Commentaires"] = commentaires || responses[lastEntryIndex]["Commentaires"];
                }
            } else {
                // Ajouter une nouvelle entrée
                responses.push({
                    "Nom de la société": societe,
                    "Date et heure de passage": datetime,
                    "Date et heure de sortie": null,
                    "Commentaires": commentaires
                });
            }

            // Sauvegarder dans le localStorage
            localStorage.setItem('carredoResponses', JSON.stringify(responses));

            // Réinitialiser le formulaire
            this.reset();
            document.getElementById('sortieGroup').style.display = 'none';
            document.getElementById('datetime').readOnly = false;
            document.getElementById('submitBtn').textContent = "Envoyer";

            // Réinitialiser la date et l'heure
            document.getElementById('datetime').value = getCurrentDatetime();

            alert(isSortie ? "Sortie validée !" : "Merci pour votre saisie !");
        });

        document.getElementById('exportBtn').addEventListener('click', function() {
            if (responses.length === 0) {
                alert("Aucune donnée à exporter.");
                return;
            }

            let csv = "Nom de la société,Date et heure de passage,Date et heure de sortie,Commentaires\n";
            responses.forEach(function(response) {
                csv += `"${response["Nom de la société"]}","${response["Date et heure de passage"]}","${response["Date et heure de sortie"] || ''}","${response["Commentaires"] || ''}"\n`;
            });

            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = url;
            link.download = 'Carredo2_Reponses.csv';
            link.click();
            URL.revokeObjectURL(url);
        });
    </script>
</body>
</html>

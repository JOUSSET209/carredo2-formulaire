html
Copier

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
        input[type="text"],
        input[type="datetime-local"],
        textarea {
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
    </style>
</head>
<body>
    <h1>Formulaire Carredo2</h1>
    <form id="carredoForm">
        <div class="form-group">
            <label for="societe">Nom de la société :</label>
            <input type="text" id="societe" name="societe" required>
        </div>

        <div class="form-group">
            <label for="datetime">Date et heure de passage :</label>
            <input type="datetime-local" id="datetime" name="datetime" required>
        </div>

        <div class="form-group">
            <label for="commentaires">Commentaires :</label>
            <textarea id="commentaires" name="commentaires"></textarea>
        </div>

        <button type="submit">Envoyer</button>
        <button type="button" id="exportBtn">Exporter en CSV (Carredo2_Reponses.csv)</button>
    </form>

    <script>
        // Tableau pour stocker les réponses
        let responses = [];

        // Écouteur pour le formulaire
        document.getElementById('carredoForm').addEventListener('submit', function(e) {
            e.preventDefault();

            // Récupérer les valeurs du formulaire
            const societe = document.getElementById('societe').value;
            const datetime = document.getElementById('datetime').value;
            const commentaires = document.getElementById('commentaires').value;

            // Ajouter la réponse au tableau
            responses.push({
                "Nom de la société": societe,
                "Date et heure de passage": datetime,
                "Commentaires": commentaires
            });

            // Réinitialiser le formulaire
            this.reset();

            // Afficher un message de confirmation
            alert("Merci pour votre saisie ! Les données sont enregistrées localement.");
        });

        // Écouteur pour le bouton d'export
        document.getElementById('exportBtn').addEventListener('click', function() {
            if (responses.length === 0) {
                alert("Aucune donnée à exporter.");
                return;
            }

            // Convertir les réponses en CSV
            let csv = "Nom de la société,Date et heure de passage,Commentaires\n";
            responses.forEach(function(response) {
                csv += `"${response["Nom de la société"]}","${response["Date et heure de passage"]}","${response["Commentaires"]}"\n`;
            });

            // Créer un lien de téléchargement
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



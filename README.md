<!DOCTYPE html>
<html lang="sv">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="manifest" href="/manifest.json">
    <title>Min Personliga anslagstavla</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1a1a1a;
            color: #fff;
            font-family: Arial, sans-serif;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            text-align: center;
        }
        .header {
            width: 100%;
            background-color: #003087;
            height: 80px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            gap: 5px;
            padding: 10px 0;
            box-sizing: border-box;
        }
        .header img {
            height: 40px;
        }
        .header span {
            font-size: 24px;
            font-weight: bold;
            color: #fff;
        }
        .header-subtitle {
            font-size: 14px;
            color: #e0e0e0;
            margin-top: -5px;
        }
        #config {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            padding: 20px;
            box-sizing: border-box;
            flex-grow: 1;
            justify-content: center;
        }
        .thumbnails {
            display: flex;
            justify-content: center;
            gap: 15px; /* Mindre gap på mobil */
            margin: 15px 0;
            flex-wrap: wrap; /* Tillåter radbrytning på små skärmar */
        }
        .thumbnail {
            width: 125px;
            height: 75px;
            object-fit: cover;
            cursor: pointer;
            border: 2px solid #fff;
            border-radius: 5px;
        }
        .thumbnail:hover {
            border-color: #007bff;
        }
        .thumbnail-text {
            font-size: 12px;
            margin-top: 5px;
            color: #ccc;
        }
        .message {
            font-size: 16px;
            margin: 15px 0;
            line-height: 1.6;
            max-width: 90%; /* Ökat för mobil */
            text-align: center;
        }
        input {
            padding: 12px; /* Större padding för touch */
            width: 90%;
            max-width: 400px;
            margin: 15px 0;
            border-radius: 5px;
            border: 1px solid #ccc;
            color: #000;
            font-size: 16px;
            box-sizing: border-box;
        }
        button {
            padding: 12px 25px;
            background-color: #007bff;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
            margin: 10px 0;
            font-size: 16px;
            width: 90%; /* Fyller skärmen på mobil */
            max-width: 300px; /* Begränsar bredd */
        }
        button:hover {
            background-color: #0056b3;
        }
        /* Modal-stil för större bild */
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            max-width: 90%;
            max-height: 90vh;
        }
        .close {
            position: absolute;
            top: 15px;
            right: 15px;
            color: #fff;
            font-size: 30px;
            cursor: pointer;
        }

        /* Media query för mobilskärmar (under 600px bredd) */
        @media (max-width: 600px) {
            .header span {
                font-size: 20px; /* Minskat textstorlek */
            }
            .header-subtitle {
                font-size: 12px; /* Minskat undertitel */
            }
            .thumbnails {
                gap: 10px; /* Mindre gap */
                margin: 10px 0;
            }
            .thumbnail {
                width: 100px; /* Minskat thumbnail-storlek */
                height: 60px;
            }
            .thumbnail-text {
                font-size: 10px; /* Minskat textstorlek */
            }
            .message {
                font-size: 14px; /* Minskat textstorlek */
                margin: 10px 0;
            }
            input {
                padding: 10px;
                width: 95%;
                max-width: 350px;
            }
            button {
                padding: 10px 20px;
                width: 95%;
                max-width: 250px;
                font-size: 14px;
            }
            .close {
                top: 10px;
                right: 10px;
                font-size: 25px;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <img src="vt-logo-negative.svg" alt="Västtrafik Logo">
        <span>Personlig anslagstavla</span>

    </div>
    <div id="config">
        <div class="thumbnails">
            <div>
                <img src="1st.png" alt="En departureboard" class="thumbnail" onclick="showModal('1st.png')">
                <div class="thumbnail-text">Klicka för att förstora</div>
            </div>
            <div>
                <img src="2st.png" alt="Två departureboards" class="thumbnail" onclick="showModal('2st.png')">
                <div class="thumbnail-text">Klicka för att förstora</div>
            </div>
        </div>
        <button onclick="openConfig()">Ställ in din tavla</button>
        <div id="message" class="message">
            <p>Hämta din länk genom att öppna Västtrafiks inställnings sida</p>
            <p>Sök och lägg till hållplats (du kan välja upp till två stycken)</p>
            <p>Kopiera den genererade länken som börjar med https://avgangstavla.vasttrafik.se/</p>
            <p>Klistra in länken i fältet nedan och klicka på "Spara" för att visa din tavla</p>
        </div>
        <input type="text" id="linkInput" placeholder="Klistra in din länk här...">
        <button onclick="saveLink()">Spara</button>
        <button onclick="goBackToConfig()" style="display: none;" id="backButton">Tillbaka till konfigurering</button>
        <button onclick="installApp()" style="display: none;" id="installButton">Installera som app</button>
    </div>
    <div id="modal" class="modal">
        <span class="close" onclick="closeModal()">×</span>
        <img id="modal-image" class="modal-content" alt="Förstorad bild">
    </div>

    <script>
        // Kontrollera om länk finns i localStorage
        const savedLink = localStorage.getItem('departureLink');
        if (savedLink && savedLink.startsWith('https://avgangstavla.vasttrafik.se/')) {
            window.location.href = savedLink; // Redirect direkt om länk finns
        }

        // Kolla om PWA-installation är möjlig
        let deferredPrompt;
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            document.getElementById('installButton').style.display = 'block';
        });

        function openConfig() {
            window.open('https://www.vasttrafik.se/reseplanering/mer-om-reseplanering/avgangstavla/', '_blank');
            if (!window.open) {
                alert('Pop-up blockerades! Tillåt pop-ups för att konfigurera tavlan.');
            }
        }

        function saveLink() {
            const linkInput = document.getElementById('linkInput').value;
            if (linkInput && linkInput.startsWith('https://avgangstavla.vasttrafik.se/')) {
                localStorage.setItem('departureLink', linkInput);
                window.location.href = linkInput; // Redirect till tavlan
            } else {
                alert('Ogiltig länk! Länken måste börja med https://avgangstavla.vasttrafik.se/');
            }
        }

        // Funktion för att gå tillbaka till konfiguration om användaren vill ändra
        function goBackToConfig() {
            localStorage.removeItem('departureLink');
            document.getElementById('linkInput').value = '';
            document.getElementById('backButton').style.display = 'none';
        }

        // Funktion för att installera som app
        function installApp() {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                deferredPrompt.userChoice.then((choiceResult) => {
                    if (choiceResult.outcome === 'accepted') {
                        console.log('Användare accepterade installation');
                    } else {
                        console.log('Användare avböjde installation');
                    }
                    deferredPrompt = null;
                    document.getElementById('installButton').style.display = 'none';
                });
            }
        }

        // Försök att fånga länk automatiskt (begränsad funktion)
        window.addEventListener('load', () => {
            const urlParams = new URLSearchParams(window.location.search);
            const stopAreaGid = urlParams.get('stopAreaGid');
            if (stopAreaGid && !localStorage.getItem('departureLink')) {
                const baseUrl = 'https://avgangstavla.vasttrafik.se/';
                const fullLink = `${baseUrl}?${urlParams.toString()}`;
                document.getElementById('linkInput').value = fullLink;
                localStorage.setItem('departureLink', fullLink);
                window.location.href = fullLink; // Redirect till tavlan
            }
        });

        // Funktioner för modal
        function showModal(imageSrc) {
            const modal = document.getElementById('modal');
            const modalImage = document.getElementById('modal-image');
            modalImage.src = imageSrc;
            modal.style.display = 'flex';
        }

        function closeModal() {
            const modal = document.getElementById('modal');
            modal.style.display = 'none';
        }

        // Stäng modal vid klick utanför bilden
        document.getElementById('modal').addEventListener('click', function(e) {
            if (e.target === this) closeModal();
        });
    </script>
</body>
</html>

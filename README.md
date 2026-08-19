<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

    <title>App Emergência</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            background-color: #121212;
            color: #ffffff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;

            width: 100%;
            height: 100vh;
            height: 100dvh;

            margin: 0;
            overflow: hidden;
        }

        .app-header {
            position: absolute;
            top: 60px;
            left: 20px;
            right: 20px;
            text-align: center;
        }

        .app-header h1 {
            margin: 0;
            font-size: 28px;
            color: #ff4757;
        }

        .app-header p {
            color: #a4b0be;
            font-size: 15px;
            margin-top: 8px;
        }

        .panic-button-container {
            position: relative;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .pulse-ring {
            position: absolute;
            top: 50%;
            left: 50%;

            width: 220px;
            height: 220px;

            transform: translate(-50%, -50%);

            background-color: rgba(255, 71, 87, 0.4);
            border-radius: 50%;

            animation: pulse 2s infinite;

            z-index: 0;
            pointer-events: none;
        }

        @keyframes pulse {
            0% {
                width: 220px;
                height: 220px;
                opacity: 1;
            }

            100% {
                width: 400px;
                height: 400px;
                opacity: 0;
            }
        }

        .panic-btn {
            position: relative;
            z-index: 1;

            width: 240px;
            height: 240px;

            border: none;
            border-radius: 50%;

            background: linear-gradient(145deg, #ff6b81, #ff4757);

            color: white;

            font-size: 40px;
            font-weight: 900;
            letter-spacing: 3px;

            cursor: pointer;

            box-shadow:
                0 20px 40px rgba(255, 71, 87, 0.5),
                inset 0 5px 15px rgba(255, 255, 255, 0.3);

            transition: transform 0.15s ease,
                        box-shadow 0.15s ease;

            display: flex;
            justify-content: center;
            align-items: center;

            -webkit-tap-highlight-color: transparent;
            touch-action: manipulation;
        }

        .panic-btn:active {
            transform: scale(0.92);

            box-shadow:
                0 10px 20px rgba(255, 71, 87, 0.5),
                inset 0 15px 30px rgba(0, 0, 0, 0.3);
        }

        .panic-btn:disabled {
            opacity: 0.7;
            cursor: wait;
        }

        .status-text {
            position: absolute;
            bottom: 80px;

            width: 90%;

            text-align: center;

            font-size: 18px;
            font-weight: bold;

            color: #2ed573;

            opacity: 0;

            transition: opacity 0.3s;
        }

        .status-text.show {
            opacity: 1;
        }

        .status-text.error {
            color: #ff4757;
        }

        @media (max-height: 600px) {
            .app-header {
                top: 20px;
            }

            .panic-btn {
                width: 190px;
                height: 190px;
                font-size: 32px;
            }

            .pulse-ring {
                width: 180px;
                height: 180px;
            }

            .status-text {
                bottom: 30px;
            }
        }
    </style>
</head>

<body>

    <header class="app-header">
        <h1>Assistência Pessoal</h1>
        <p>Toque no botão central em caso de emergência</p>
    </header>

    <main class="panic-button-container">

        <div class="pulse-ring"></div>

        <button
            id="panicButton"
            class="panic-btn"
            type="button"
            onclick="acionarEmergencia()"
            aria-label="Acionar emergência"
        >
            S.O.S
        </button>

    </main>

    <div id="status" class="status-text" role="status">
        Alerta preparado!
    </div>

    <script>
        function mostrarStatus(mensagem, erro = false) {
            const status = document.getElementById("status");

            status.textContent = mensagem;
            status.classList.toggle("error", erro);
            status.classList.add("show");

            setTimeout(() => {
                status.classList.remove("show");
            }, 3000);
        }

        function vibrar() {
            if ("vibrate" in navigator) {
                navigator.vibrate([200, 100, 200]);
            }
        }

        function obterLocalizacao() {
            return new Promise((resolve, reject) => {

                if (!navigator.geolocation) {
                    reject(new Error("Geolocalização não é suportada neste dispositivo."));
                    return;
                }

                navigator.geolocation.getCurrentPosition(
                    position => {
                        resolve({
                            latitude: position.coords.latitude,
                            longitude: position.coords.longitude,
                            precisao: position.coords.accuracy
                        });
                    },

                    error => {
                        reject(error);
                    },

                    {
                        enableHighAccuracy: true,
                        timeout: 10000,
                        maximumAge: 0
                    }
                );
            });
        }

        async function acionarEmergencia() {

            const botao = document.getElementById("panicButton");

            vibrar();

            botao.disabled = true;

            mostrarStatus("Obtendo sua localização...");

            try {

                const localizacao = await obterLocalizacao();

                console.log("Localização obtida:", localizacao);

                mostrarStatus("Localização obtida com sucesso!");

                setTimeout(() => {

                    alert(
                        "🚨 EMERGÊNCIA ACIONADA!\n\n" +
                        "Localização obtida:\n" +
                        "Latitude: " + localizacao.latitude.toFixed(6) + "\n" +
                        "Longitude: " + localizacao.longitude.toFixed(6) + "\n\n" +
                        "A localização ainda NÃO foi enviada para contatos. " +
                        "Para isso, é necessário conectar o aplicativo a um servidor/API."
                    );

                }, 500);

            } catch (erro) {

                console.error("Erro ao obter localização:", erro);

                mostrarStatus(
                    "Não foi possível obter sua localização.",
                    true
                );

                setTimeout(() => {

                    alert(
                        "🚨 EMERGÊNCIA ACIONADA!\n\n" +
                        "Não foi possível obter sua localização.\n\n" +
                        "Verifique se o GPS/localização está ativado " +
                        "e se o navegador possui permissão."
                    );

                }, 500);

            } finally {

                setTimeout(() => {
                    botao.disabled = false;
                }, 1000);
            }
        }
    </script>

</body>
</html>

<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ETH Alım-Satım Paneli</title>
    <style>
        body {
            background-color: #111;
            color: white;
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            background: #222;
            padding: 20px;
            border-radius: 12px;
            width: 320px;
            text-align: center;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
        }
        .price {
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .buttons {
            display: flex;
            justify-content: space-between;
        }
        button {
            flex: 1;
            margin: 5px;
            padding: 10px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
        }
        .buy { background: green; color: white; }
        .sell { background: red; color: white; }
        .input-group {
            display: none;
            margin-top: 10px;
        }
        input {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border-radius: 5px;
            border: none;
            text-align: center;
        }
        .total {
            margin-top: 10px;
            font-size: 18px;
            font-weight: bold;
        }
        .trade-button {
            width: 100%;
            padding: 10px;
            margin-top: 10px;
            border: none;
            border-radius: 8px;
            font-size: 18px;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div class="container">
        <div class="price">ETH/USDT - <span id="price">Loading...</span></div>

        <div class="buttons">
            <button class="buy" onclick="toggleTrade('buy')">Al</button>
            <button class="sell" onclick="toggleTrade('sell')">Sat</button>
        </div>

        <div class="input-group" id="trade-form">
            <input type="number" id="amount" placeholder="Miktar (ETH)" oninput="calculateTotal()">
            <div class="total">Toplam: <span id="total">0.00</span> <span id="currency">USDT</span></div>
            <button class="trade-button" id="trade-button">İşlem Yap</button>
        </div>
    </div>

    <script>
        let tradeType = "buy";
        let ethPrice = 0;

        async function fetchPrice() {
            try {
                const response = await fetch("https://api.binance.com/api/v3/ticker/price?symbol=ETHUSDT");
                const data = await response.json();
                ethPrice = parseFloat(data.price).toFixed(2);
                document.getElementById("price").textContent = ethPrice;
            } catch (error) {
                console.error("Fiyat alınamadı", error);
            }
        }

        setInterval(fetchPrice, 500);
        fetchPrice();

        function toggleTrade(type) {
            tradeType = type;
            document.getElementById("trade-form").style.display = "block";
            document.getElementById("currency").textContent = tradeType === "buy" ? "USDT" : "ETH";
            document.getElementById("trade-button").textContent = tradeType === "buy" ? "ETH Satın Al" : "ETH Sat";
        }

        function calculateTotal() {
            const amount = document.getElementById("amount").value;
            const total = amount ? (parseFloat(amount) * ethPrice).toFixed(2) : "0.00";
            document.getElementById("total").textContent = total;
        }
    </script>

</body>
</html>

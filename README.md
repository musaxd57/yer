<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ETH Alım-Satım Paneli</title>
    <style>
        body {
            background: linear-gradient(135deg, #1a1a1a, #2c2c2c);
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
            box-shadow: 0 0 15px rgba(255, 255, 255, 0.15);
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
            transition: 0.3s;
        }
        .buy { background: green; color: white; }
        .sell { background: red; color: white; }
        .input-group {
            margin-top: 10px;
        }
        input {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border-radius: 5px;
            border: none;
            text-align: center;
            font-size: 16px;
            background: #333;
            color: white;
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
            transition: 0.3s;
        }
    </style>
</head>
<body>

    <div class="container">
        <div class="price">ETH/USDT - <span id="price">Loading...</span></div>

        <div class="buttons">
            <button class="buy" onclick="setTradeType('buy')">Al</button>
            <button class="sell" onclick="setTradeType('sell')">Sat</button>
        </div>

        <div class="input-group">
            <input type="number" id="amount" min="0.001" step="0.001" placeholder="Miktar" oninput="calculateTotal()">
            <div class="total">Toplam: <span id="total">0.00</span> <span id="currency">ETH</span></div>
            <button class="trade-button" id="trade-button">ETH Satın Al</button>
        </div>
    </div>

    <script>
        let tradeType = "buy";
        let ethPrice = 0;

        async function fetchPrice() {
            try {
                const response = await fetch("https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd");
                const data = await response.json();
                ethPrice = parseFloat(data.ethereum.usd).toFixed(2);
                document.getElementById("price").textContent = ethPrice;
                calculateTotal(); // Yeni fiyatla toplamı güncelle
            } catch (error) {
                console.error("Fiyat alınamadı", error);
            }
        }

        setInterval(fetchPrice, 300); // 0.3 saniyede bir güncelle
        fetchPrice();

        function setTradeType(type) {
            tradeType = type;
            const input = document.getElementById("amount");
            const currencyLabel = document.getElementById("currency");
            const tradeButton = document.getElementById("trade-button");

            if (tradeType === "buy") {
                input.placeholder = "USDT Miktarı";
                currencyLabel.textContent = "ETH";
                tradeButton.textContent = "ETH Satın Al";
            } else {
                input.placeholder = "ETH Miktarı";
                currencyLabel.textContent = "USDT";
                tradeButton.textContent = "ETH Sat";
            }
            calculateTotal(); // Seçim değiştiğinde anında toplamı güncelle
        }

        function calculateTotal() {
            const amountInput = document.getElementById("amount");
            let amount = parseFloat(amountInput.value);

            if (isNaN(amount) || amount < 0.001) {
                amountInput.value = "0.001"; // Minimum miktar 0.001
                amount = 0.001;
            }

            const total = tradeType === "buy"
                ? (amount / ethPrice).toFixed(6) // USDT ile alırken ETH hesapla
                : (amount * ethPrice).toFixed(2); // ETH ile satarken USDT hesapla

            document.getElementById("total").textContent = total;
        }
    </script>

</body>
</html>

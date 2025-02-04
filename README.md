<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ETH Alım-Satım Paneli</title>
    <style>
        body {
            background: linear-gradient(135deg, #6ee7b7, #3b82f6);
            color: #333;
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            background: #ffffff;
            padding: 20px;
            border-radius: 12px;
            width: 350px;
            text-align: center;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
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
        .buy { background: #10b981; color: white; }
        .sell { background: #ef4444; color: white; }
        .input-group {
            margin-top: 10px;
        }
        input {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border-radius: 5px;
            border: 1px solid #ccc;
            text-align: center;
            font-size: 16px;
            background: #f9fafb;
            color: #333;
            position: relative;
        }
        input::placeholder {
            color: #6b7280; /* Silik gri renk */
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
            background: #2563eb;
            color: white;
            transition: 0.3s;
        }
        .trade-button:hover {
            background: #1d4ed8;
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
                calculateTotal();
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
            calculateTotal();
        }

        function calculateTotal() {
            const amountInput = document.getElementById("amount");
            let amount = parseFloat(amountInput.value);

            if (isNaN(amount) || amount < 0.001) {
                amountInput.value = "";
                amount = 0.001;
            }

            const total = tradeType === "buy"
                ? (amount / ethPrice).toFixed(6)
                : (amount * ethPrice).toFixed(2);

            document.getElementById("total").textContent = total;
        }
    </script>
</body>
</html>

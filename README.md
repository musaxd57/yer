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
            transition: color 0.3s ease; /* Fiyatın renk geçişi */
        }
        .buttons {
            display: flex;
            justify-content: space-between;
        }
        button {
            flex: 1;
            margin: 5px;
            padding: 15px;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-size: 18px;
            transition: 0.3s;
        }
        .buy { background: #10b981; color: white; font-size: 18px; }
        .sell { background: #ef4444; color: white; font-size: 18px; }
        .buy:hover { background: #059669; }
        .sell:hover { background: #9b1c1c; }
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
            color: #6b7280;
        }
        .total {
            margin-top: 10px;
            font-size: 18px;
            font-weight: bold;
        }
        .trade-button {
            width: 100%;
            padding: 12px;
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
        .max-button {
            background-color: #fbbf24;
            color: white;
            border-radius: 5px;
            border: none;
            padding: 8px;
            margin-top: 5px;
            cursor: pointer;
            width: 100%;
        }
        .max-button:hover {
            background-color: #d97706;
        }
        .loading-spinner {
            display: none;
            margin-top: 10px;
        }
        .success-message {
            color: green;
            font-size: 20px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="price" id="eth-price">ETH/USDT - Yükleniyor...</div>

        <div class="buttons">
            <button class="buy" onclick="setTradeType('buy')">Al</button>
            <button class="sell" onclick="setTradeType('sell')">Sat</button>
        </div>

        <div class="input-group">
            <input type="number" id="amount" min="0.001" step="0.001" placeholder="USDT Miktarı" oninput="calculateTotal()">
            <button class="max-button" id="max-button" onclick="setMaxAmount()">Max</button>
            <div class="total">Toplam: <span id="total">0.00</span> <span id="currency">ETH</span></div>
            <button class="trade-button" id="trade-button" onclick="processTransaction()">ETH Satın Al</button>
            <div class="loading-spinner" id="loading-spinner">Yükleniyor...</div>
            <div class="success-message" id="success-message"></div>
        </div>
    </div>

    <script>
        let tradeType = "buy";
        let ethPrice = 0;
        let previousEthPrice = 0; // Önceki fiyatı tutacak değişken
        let userETHBalance = 1.5; // Kullanıcı ETH bakiyesi (gerçek zamanlı verilerle değiştirilebilir)
        let userUSDTBalance = 5000; // Kullanıcı USDT bakiyesi (gerçek zamanlı verilerle değiştirilebilir)

        // Binance API'den fiyat çekme fonksiyonu
        async function fetchPrice() {
            try {
                const response = await fetch("https://api.binance.com/api/v3/ticker/price?symbol=ETHUSDT");
                const data = await response.json();
                ethPrice = parseFloat(data.price).toFixed(2);

                // Fiyat değişimini kontrol et ve renk değişikliğini uygula
                const priceElement = document.getElementById("eth-price");
                if (ethPrice > previousEthPrice) {
                    priceElement.style.color = "#10b981"; // Yeşil renk artışta
                } else if (ethPrice < previousEthPrice) {
                    priceElement.style.color = "#ef4444"; // Kırmızı renk azalışta
                } else {
                    priceElement.style.color = "#333"; // Fiyat aynı kaldığında renk nötr
                }

                // Fiyatı ekranda güncelle
                document.getElementById("eth-price").textContent = "ETH/USDT - " + ethPrice;
                previousEthPrice = ethPrice; // Önceki fiyatı güncelle
                calculateTotal();
            } catch (error) {
                console.error("Fiyat alınamadı", error);
                document.getElementById("eth-price").textContent = "Fiyat Alınamadı"; // Hata durumunda mesaj göster
            }
        }

        setInterval(fetchPrice, 300); // 0.3 saniyede bir güncelle (daha hızlı)
        fetchPrice(); // Sayfa yüklenirken fiyatı hemen al

        // Alım ve satım tipini belirleme
        function setTradeType(type) {
            tradeType = type;
            const input = document.getElementById("amount");
            const currencyLabel = document.getElementById("currency");
            const tradeButton = document.getElementById("trade-button");

            if (tradeType === "buy") {
                input.placeholder = "USDT Miktarı"; // Alış kısmında USDT miktarı yazacak
                currencyLabel.textContent = "ETH";
                tradeButton.textContent = "ETH Satın Al";
            } else {
                input.placeholder = "ETH Miktarı (Max: " + userETHBalance.toFixed(3) + ")";
                currencyLabel.textContent = "USDT";
                tradeButton.textContent = "ETH Sat";
            }
            calculateTotal();
        }

        // Kullanıcı bakiyesi ile maksimum alım veya satımı belirleme
        function setMaxAmount() {
            const input = document.getElementById("amount");
            if (tradeType === "buy") {
                const maxAmountToBuy = (userUSDTBalance / ethPrice).toFixed(6);
                input.value = maxAmountToBuy;
            } else {
                input.value = userETHBalance.toFixed(3);
            }
            calculateTotal();
        }

        // Toplam değeri hesaplayan fonksiyon
        function calculateTotal() {
            const amountInput = document.getElementById("amount");
            let amount = parseFloat(amountInput.value);

            if (isNaN(amount) || amount < 0.001) {
                amountInput.value = "";
                amount = 0.001;
            }

            let total = 0;
            if (tradeType === "buy") {
                total = (amount / ethPrice).toFixed(6);
            } else {
                if (amount > userETHBalance) amount = userETHBalance; // Max ETH satışı
                total = (amount * ethPrice).toFixed(2);
            }

            document.getElementById("total").textContent = total;
        }

        // İşlemi başlatan fonksiyon
        function processTransaction() {
            const loadingSpinner = document.getElementById("loading-spinner");
            const successMessage = document.getElementById("success-message");
            
            // Yükleniyor simgesini göster
            loadingSpinner.style.display = "block";
            successMessage.textContent = "";

            setTimeout(() => {
                // Simüle edilen işlem tamamlandığında
                loadingSpinner.style.display = "none";
                successMessage.textContent = tradeType === "buy" ? "ETH Satın Alındı!" : "ETH Satıldı!";
                // Bakiye güncellemesi yapılabilir burada
            }, 2000); // 2 saniye simülasyon süresi
        }
    </script>
</body>
</html>

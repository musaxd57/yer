import { useState, useEffect } from "react";

export default function App() {
  const [tradeType, setTradeType] = useState("buy"); // 'buy' veya 'sell'
  const [price, setPrice] = useState(2735); // ETH fiyatı
  const [amount, setAmount] = useState("");

  useEffect(() => {
    const fetchPrice = async () => {
      try {
        const response = await fetch(
          "https://api.binance.com/api/v3/ticker/price?symbol=ETHUSDT"
        );
        const data = await response.json();
        setPrice(parseFloat(data.price).toFixed(2));
      } catch (error) {
        console.error("ETH fiyatını alırken hata oluştu", error);
      }
    };

    fetchPrice();
    const interval = setInterval(fetchPrice, 500); // 0.5 saniyede bir güncelle

    return () => clearInterval(interval);
  }, []);

  const total = amount ? (parseFloat(amount) * price).toFixed(2) : "0.00";

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-950">
      <div className="max-w-md mx-auto bg-gray-900 p-6 rounded-2xl shadow-xl text-white">
        {/* Fiyat Gösterimi */}
        <div className="text-xl font-semibold text-center mb-4">
          ETH/USDT - ${price}
        </div>

        {/* Al / Sat Butonları */}
        <div className="flex space-x-4 mb-4">
          <button
            onClick={() => setTradeType("buy")}
            className={`flex-1 py-2 rounded-lg text-lg font-semibold transition ${
              tradeType === "buy" ? "bg-green-500" : "bg-gray-700"
            }`}
          >
            Al
          </button>
          <button
            onClick={() => setTradeType("sell")}
            className={`flex-1 py-2 rounded-lg text-lg font-semibold transition ${
              tradeType === "sell" ? "bg-red-500" : "bg-gray-700"
            }`}
          >
            Sat
          </button>
        </div>

        {/* Form Alanları */}
        <div className="space-y-4">
          <div>
            <label className="block text-sm">Miktar (ETH)</label>
            <input
              type="number"
              value={amount}
              onChange={(e) => setAmount(e.target.value)}
              className="w-full p-2 mt-1 rounded bg-gray-800 text-white focus:ring focus:ring-blue-500"
            />
          </div>

          <div>
            <label className="block text-sm">
              Toplam ({tradeType === "buy" ? "USDT" : "ETH"})
            </label>
            <div className="w-full p-2 mt-1 rounded bg-gray-700 text-white">
              {total}
            </div>
          </div>
        </div>

        {/* İşlem Butonu */}
        <button
          className={`w-full mt-4 py-2 rounded-lg text-lg font-semibold transition ${
            tradeType === "buy"
              ? "bg-green-600 hover:bg-green-500"
              : "bg-red-600 hover:bg-red-500"
          }`}
        >
          {tradeType === "buy" ? "ETH Satın Al" : "ETH Sat"}
        </button>
      </div>
    </div>
  );
}


# bukaolshop-saldo
const express = require("express");
const bodyParser = require("body-parser");
const mysql = require("mysql2/promise");

const app = express();
app.use(bodyParser.json());

// 🔑 Koneksi ke database MySQL
const db = mysql.createPool({
  host: "localhost",   // ganti sesuai database
  user: "root",        // ganti username MySQL
  password: "password",// ganti password MySQL
  database: "topup_db" // nama database
});

// Endpoint callback saldo
app.post("/callback/saldo", async (req, res) => {
  const data = req.body;
  console.log("Callback diterima:", data);

  const { user_id, amount, status, trx_id } = data;

  try {
    if (status === "PAID") {
      // ✅ Update saldo user
      const [result] = await db.query(
        "UPDATE users SET saldo = saldo + ? WHERE id = ?",
        [amount, user_id]
      );

      if (result.affectedRows > 0) {
        console.log(`✅ Saldo user ${user_id} bertambah Rp${amount}`);
      } else {
        console.log(`⚠️ User ${user_id} tidak ditemukan`);
      }
    } else {
      console.log(`❌ Topup gagal untuk trx_id ${trx_id}`);
    }

    // WAJIB balas 200 OK ke Bukaolshop
    res.status(200).json({ success: true });
  } catch (err) {
    console.error("❌ Error update saldo:", err);
    res.status(500).json({ success: false, error: err.message });
  }
});

// Jalankan server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Webhook saldo jalan di port ${PORT}`);
});

require('dotenv').config();
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');

const app = express();
app.use(bodyParser.json());

const PHONE_NUMBER_ID = process.env.PHONE_NUMBER_ID;
const ACCESS_TOKEN = process.env.ACCESS_TOKEN;
const VERIFY_TOKEN = process.env.VERIFY_TOKEN || 'monstrocell2026';
const PORT = process.env.PORT || 3000;

// Catálogo MONSTROCELL
const CATALOGO = {
  bienvenida: `¡Hola! 👋 Soy el asistente de *MONSTROCELL* 🟡🔵

Celulares a crédito ¡sin tanto papeleo!

📍 Tenemos 7 sedes para ti:
• Curumaní
• La Jagua de Ibirico
• Rincón Hondo
• Santa Isabel
• El Banco
• Hatillo de Loba
• San Martín

Escribe el número de lo que quieres ver:
1️⃣ iPhone a crédito
2️⃣ Samsung a crédito
3️⃣ TV y Electrodomésticos
4️⃣ Hablar con asesor
5️⃣ Ubicaciones y horarios`,

  iphone: `📱 *IPHONE A CRÉDITO - MONSTROCELL*

✨ iPhone 13 desde $185.000/mes
✨ iPhone 14 desde $220.000/mes
✨ iPhone 15 desde $280.000/mes
✨ iPhone 15 Pro Max desde $350.000/mes

✅ Solo con cédula
✅ Aprobación en 20 minutos
✅ Sin cuota inicial (según perfil)

¿En cuál sede quieres que te atendamos? Responde con tu municipio.`,

  samsung: `📱 *SAMSUNG A CRÉDITO - MONSTROCELL*

🔥 Galaxy A14 desde $65.000/mes
🔥 Galaxy A34 desde $95.000/mes
🔥 Galaxy A54 desde $125.000/mes
🔥 Galaxy S23 desde $195.000/mes

💳 Crédito fácil y rápido
🎁 Obsequio por tu compra

Escribe "ASESOR" para hablar con uno de nuestros asesores.`,

  tv: `📺 *TV Y ELECTRODOMÉSTICOS A CRÉDITO*

• Smart TV 32" desde $75.000/mes
• Smart TV 50" 4K desde $135.000/mes
• Neveras, Lavadoras, Parlantes JBL
• Moto Bici eléctrica

📦 Entrega inmediata en nuestras 7 sedes.

Responde con lo que necesitas y te cotizamos al instante.`,

  asesor: `Perfecto 🙌 Un asesor de MONSTROCELL te atenderá en breve.

Por favor envíanos:
1. Nombre completo
2. Municipio (Curumaní, La Jagua, Rincón Hondo, Santa Isabel, El Banco, Hatillo, San Martín)
3. Qué equipo te interesa

Horario: Lunes a Sábado 8am - 6pm`,

  sedes: `📍 *NUESTRAS 7 SEDES MONSTROCELL*

1. Curumaní - Centro
2. La Jagua de Ibirico - Principal
3. Rincón Hondo
4. Santa Isabel
5. El Banco, Magdalena
6. Hatillo de Loba
7. San Martín, Cesar

🕗 Lunes a Sábado 8:00 AM - 6:00 PM
📲 ¡Te esperamos para darte tu crédito fácil!`
};

async function enviarMensaje(to, texto) {
  try {
    await axios({
      method: 'POST',
      url: `https://graph.facebook.com/v19.0/${PHONE_NUMBER_ID}/messages`,
      headers: {
        'Authorization': `Bearer ${ACCESS_TOKEN}`,
        'Content-Type': 'application/json'
      },
      data: {
        messaging_product: 'whatsapp',
        to: to,
        type: 'text',
        text: { body: texto }
      }
    });
    console.log(`Mensaje enviado a ${to}`);
  } catch (error) {
    console.error('Error enviando mensaje:', error.response?.data || error.message);
  }
}

// Verificación del Webhook (GET)
app.get('/webhook', (req, res) => {
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];

  if (mode && token) {
    if (mode === 'subscribe' && token === VERIFY_TOKEN) {
      console.log('WEBHOOK VERIFICADO');
      res.status(200).send(challenge);
    } else {
      res.sendStatus(403);
    }
  } else {
    res.sendStatus(400);
  }
});

// Recepción de mensajes (POST)
app.post('/webhook', async (req, res) => {
  const body = req.body;

  if (body.object) {
    if (body.entry && body.entry[0].changes && body.entry[0].changes[0].value.messages) {
      const phoneNumberId = body.entry[0].changes[0].value.metadata.phone_number_id;
      const from = body.entry[0].changes[0].value.messages[0].from;
      const msgBody = body.entry[0].changes[0].value.messages[0].text?.body || '';
      const texto = msgBody.toLowerCase().trim();

      console.log(`Mensaje de ${from}: ${msgBody}`);

      let respuesta = CATALOGO.bienvenida;

      if (texto.includes('1') || texto.includes('iphone')) {
        respuesta = CATALOGO.iphone;
      } else if (texto.includes('2') || texto.includes('samsung')) {
        respuesta = CATALOGO.samsung;
      } else if (texto.includes('3') || texto.includes('tv') || texto.includes('electro')) {
        respuesta = CATALOGO.tv;
      } else if (texto.includes('4') || texto.includes('asesor') || texto.includes('humano')) {
        respuesta = CATALOGO.asesor;
      } else if (texto.includes('5') || texto.includes('sede') || texto.includes('ubicacion') || texto.includes('donde')) {
        respuesta = CATALOGO.sedes;
      } else if (texto === 'hola' || texto === 'buenas' || texto === 'inicio' || texto === 'menu') {
        respuesta = CATALOGO.bienvenida;
      }

      await enviarMensaje(from, respuesta);
    }
    res.sendStatus(200);
  } else {
    res.sendStatus(404);
  }
});

app.get('/', (req, res) => {
  res.send('MONSTROCELL Bot Activo 🟡🔵 - 7 Sedes');
});

app.listen(PORT, () => {
  console.log(`MONSTROCELL Bot corriendo en puerto ${PORT}`);
});

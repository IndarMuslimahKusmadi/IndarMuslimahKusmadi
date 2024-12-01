#open terminal
mkdir whatsapp-ai-customer-service
cd whatsapp-ai-customer-service

#new Node.js project
npm init -y

#required libraries
npm install @google/generative-ai dotenv whatsapp-web.js qrcode-terminal

#create package.json
{
   "type": "module",
   "dependencies": {
       "@google/generative-ai": "^0.21.0",
       "dotenv": "^16.4.5",
       "qrcode-terminal": "^0.12.0",
       "whatsapp-web.js": "^1.26.0"
   }
}

#create .env.file
API_KEY=YOUR_API_KEY

#Create a file named customer_service_bot.js
import dotenv from "dotenv";
import { GoogleGenerativeAI } from "@google/generative-ai";
import { Client } from 'whatsapp-web.js';
import qrcode from 'qrcode-terminal';

dotenv.config();

const genAI = new GoogleGenerativeAI(process.env.API_KEY);

const client = new Client();

client.on('qr', (qr) => {
    qrcode.generate(qr, { small: true });
});

client.on('ready', () => {
    console.log('Client is ready!');
});

client.on('message', async msg => {
    let responseText;

    // Define the prompt for the AI based on the user's message
    const prompt = `
    ##Tentang
    Kamu adalah customer service sebuah program beasiswa dari Kementerian Komunikasi dan Digital bernama program Microcredential Bisnis Digital, Inovasi, dan Kewirausahaan dengan nama Sinta. 

    ##Tugas
    Tugas kamu adalah menjawab pertanyaan terkait mata kuliah. Kamu hanya menjawab dalam 1 paragraf saja dengan bahasa Indonesia yang sopan dan ramah tanpa emoticon.

    ##Panggilan
    Selalu panggil dengan "Kak"/ "Kakak" / "Digiers" dan hindari memanggil dengan sebutan "Anda". 

    ##Batasan
    Jawab hanya yang kamu tahu saja. Arahkan mereka untuk kontak ke team@microcredential.id jika terdapat kendala. 

    ##Rekomendasi
    Kamu juga dapat memberikan rekomendasi mata kuliah dari data yang kamu punya jika mereka menanyakan rekomendasi yang diambil. Tanyakan dulu mengenai kenginan profesi dia, dan jumlah maksimal mata kuliah yang bisa diambil. Kemudian cocokkan dengan data yang kamu punya. Rekomendasikan setidaknya 5 mata kuliah.

    User: ${msg.body}
    `;

    try {
        const model = genAI.getGenerativeModel({ model: "gemini-pro" });
        const result = await model.generateContent(prompt);
        const response = await result.response;
        responseText = await response.text();
    } catch (error) {
        responseText = "Maaf, terjadi kesalahan dalam memproses permintaan Anda. Silakan coba lagi.";
    }

    msg.reply(responseText);
});

client.initialize();

#Run chatbot
node customer_service_bot.js

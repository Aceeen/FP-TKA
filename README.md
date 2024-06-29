# FP-TKA

Kelompok C9

- Acintya Edria Sudarsono - 5027231020
- Aswalia Novitriasari - 5027231012
- Dian Anggraeni Putri - 5027231016
- Nisrina Atiqah - 5027231075

## Permasalahan
Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah **Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.**

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: [sentiment-analysis.py](/Resources/BE/sentiment-analysis.py) dengan spesifikasi sebagai berikut

## Endpoints:


1. **Analyze Text**
   - **Endpoint:** `POST /analyze`
   - **Description:** This endpoint accepts a text input and returns the sentiment score of the text.
   - **Request:**
     ```json
     {
        "text": "Your text here"
     }
     ```
    - **Response:**
      ```json
      {
        "sentiment": <sentiment_score>
      }
      ```

2. **Retrieve History**
   - **Endpoint:** `GET /history`
   - **Description:** This endpoint retrieves the history of previously analyzed texts along with their sentiment scores.
   - **Response:**
     ```json
     {
      {
        "text": "Your previous text here",
        "sentiment": <sentiment_score>
      },
      ...
     }
     ```
---

Kemudian juga disediakan sebuah Frontend sederhana menggunakan [index.html](/Resources/FE/index.html) dan [styles.css](/Resources/FE/styles.css) dengan tampilan antarmuka sebagai berikut

![alt text](image.png)

Kemudian anda diminta untuk mendesain arsitektur cloud yang sesuai dengan kebutuhan aplikasi tersebut. Apabila dana maksimal yang diberikan adalah **1 juta rupiah per bulan (65 US$)**
konfigurasi cloud terbaik seperti apa yang bisa dibuat?

## Rancangan Arsitektur
![Arsitektur FP TKA drawio (4)](https://github.com/Aceeen/FP-TKA/assets/150018995/b4f0de16-edd2-4734-acfc-52527e4c9d18)


## Membuat Virtual Machine Aplikasi
Membuat Resource Group untuk mengelompokan sumber daya yang akan digunakan. Disini kami memberi nama FP-TKA.

Membuat Virtual Machine menggunakan Azure dengan konfigurasi sebagai berikut:

1. Menggunakan resource group yang telah dibuat, yaitu FP-TKA
2. Menggunakan ubuntu sebagai konfigurasinya
3. Menggunakan x64 sebagai VM Architecturenya.\
4. Memilih size dengan spesifikasi 1 vcpu, 2GiB memory ($19.27/month)
5. Menggunakan SSH public key sebagai authentication type.
6. Memilih untuk Allow selected port agar port yang dibuat bisa diakses di internet. Port yang bisa diakses adalah HTTP (80) dan SSH (22).

![Screenshot 2024-06-20 153803](https://github.com/Aceeen/FP-TKA/assets/150018995/65a6f93b-dc37-451a-9617-623687dec0aa)

![Screenshot 2024-06-20 153817](https://github.com/Aceeen/FP-TKA/assets/150018995/5a77c7f1-7922-4573-9d2f-2188d66e00a4)

![Screenshot 2024-06-20 153828](https://github.com/Aceeen/FP-TKA/assets/150018995/c9243746-e9a3-4efb-b11a-9fb958115e34)

## Membuat Virtual Machine Load Balancer
Membuat Resource Group untuk mengelompokan sumber daya yang akan digunakan. Disini kami memberi nama FP-TKA.

Selanjutnya, membuat VM dengan Konfigurasi sebagai berikut:

1. Menggunakan resource group yang telah dibuat, yaitu FP-TKA.
2. Menggunakan Nginx sebagai konfigurasinya.
3. Menggunakan x64 sebagai VM Architecturenya.
4. Memilih size dengan spesifikasi 1 vcpu, 1GiB memory ($9.64/month).
5. Menggunakan SSH public key sebagai authentication type.
6. Jika sudah di halaman terakhir dan tidak terjadi error, klik tombol review + create yang ada di pojok kiri bawah.
Terakhir, tunggu virtual machine yang telah kita buat di deploy oleh azure.

![Screenshot 2024-06-20 155116](https://github.com/Aceeen/FP-TKA/assets/150018995/5dc02b6e-ae2b-4ef5-bf45-804656f6dcf6)


![Screenshot 2024-06-20 155131](https://github.com/Aceeen/FP-TKA/assets/150018995/69de3623-0d2e-4f16-b472-cc810b583d78)
![Screenshot 2024-06-20 155140](https://github.com/Aceeen/FP-TKA/assets/150018995/d7a1bb3f-1035-4505-bec1-6ec640a3a339)

## Konfigurasi VM
```
sudo apt-get install nginx
sudo apt-get install python3
sudo apt install python3-pip
pip install Flask Flask-PyMongo
pip install pymongo
pip install gunicorn
```
Konfigurasi index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sentiment Analysis</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>Sentiment Analysis</h1>
        <textarea id="inputText" placeholder="Enter text here..."></textarea>
        <button onclick="analyzeText()">Analyze</button>
        <div id="result"></div>
        <h2>History</h2>
        <div id="history"></div>
    </div>

   <script>
    async function analyzeSentiment() {
        const text = document.getElementById('textInput').value;
        const response = await fetch('http://20.92.202.97/api/analyze', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ text: text })
        });
        const result = await response.json();
        document.getElementById('historyList').innerHTML += `<li>${text} - ${result.sentiment}</li>`;
    }

    async function loadHistory() {
        const response = await fetch('http://20.92.202.97/api/history');
        const history = await response.json();
        const historyList = document.getElementById('historyList');
        historyList.innerHTML = '';
        history.forEach(item => {
            historyList.innerHTML += `<li>${item.text} - ${item.sentiment}</li>`;
        });
    }

    window.onload = loadHistory;
</script>

        function addHistoryItem(text, sentiment) {
            const historyContainer = document.getElementById('history');
            const historyItem = document.createElement('div');
            historyItem.className = 'history-item';
            historyItem.innerText = `Text: ${text}, Sentiment: ${sentiment}`;
            historyContainer.appendChild(historyItem);
        }

        window.onload = loadHistory;
    </script>
</body>
</html>
```
Konfigurasi styles.css
```
body {
    font-family: Arial, sans-serif;
    background-color: #f5f5f5;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.container {
    background-color: #ffffff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    width: 400px;
    text-align: center;
}

h1 {
    font-size: 24px;
    margin-bottom: 20px;
}

textarea {
    width: 100%;
    height: 100px;
    padding: 10px;
    border-radius: 4px;
    border: 1px solid #ddd;
    margin-bottom: 10px;
}

button {
    background-color: #007bff;
    color: #ffffff;
    border: none;
    border-radius: 4px;
    padding: 10px 20px;
    cursor: pointer;
}

button:hover {
    background-color: #0056b3;
}

#result {
    margin-top: 20px;
    font-size: 18px;
}

.history-item {
    padding: 10px;
    border-radius: 4px;
    margin-bottom: 5px;
}

.history-item:nth-child(odd) {
    background-color: #f0f8ff;
}

.history-item:nth-child(even) {
    background-color: #e6f7ff;
}

```
Konfigurasi service file Gunicorn
```
sudo nano /etc/systemd/system/sentiment_analysis.service

```
Tambahkan konfigurasi berikut
```
[Unit]
Description=Gunicorn instance to serve sentiment analysis
After=network.target

[Service]
User=azureuser
Group=www-data
WorkingDirectory=/home/azureuser
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind unix:/home/azureuser/sentiment_analysis.sock -m 007 sentiment_analysis:app

[Install]
WantedBy=multi-user.target

```
Aktifkan service Gunicorn
```
sudo systemctl start sentiment_analysis
sudo systemctl enable sentiment_analysis
```

Konfigurasi Nginx
```
sudo nano /etc/nginx/sites-available/default

```
Gunakan konfigurasi berikut
```
server {
    listen 80;

    location / {
        root /var/www/html;
        index index.html;
    }

    location /api {
        proxy_pass http://unix:/home/azureuser/sentiment_analysis.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```
Test dan restart Nginx untuk memastikan
```
sudo nginx -t
sudo systemctl restart nginx
```
Lalu coba akses address http://20.92.202.253
## Konfigurasi Load Balancer
1. Buat Load Balancer menggunakan Nginx di LoadBalancerVM.
2. Konfigurasi upstream untuk mendistribusikan lalu lintas ke Virtual Machine

Konfigurasi Load Balancer VM
```
sudo apt-get update
sudo apt-get install -y nginx
```
Konfigurasi Nginx Load Balancer
```
sudo nano /etc/nginx/sites-available/default
```
Gunakan konfigurasi berikut
```
upstream sentiment_backend {
    server 20.92.202.253:80;
}

server {
    listen 80;

    location / {
        proxy_pass http://sentiment_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
# REVISI
## Tampilan Interface

![image](https://github.com/Aceeen/FP-TKA/assets/151147728/541500ef-302d-4423-b1f3-0febc6cf5eac)
```
from flask import Flask, request, jsonify
from flask_cors import CORS
from textblob import TextBlob
from pymongo import MongoClient

app = Flask(_name_)
CORS(app)

# Database setup
client = MongoClient('mongodb://localhost:27017/')
db = client.sentiment_analysis
collection = db.history

@app.route('/analyze', methods=['POST'])
def analyze_sentiment():
    data = request.get_json()
    text = data.get('text', '')
    analysis = TextBlob(text)
    sentiment = analysis.sentiment.polarity

    # Save to database
    collection.insert_one({'text': text, 'sentiment': sentiment})

    return jsonify({'sentiment': sentiment})

@app.route('/history', methods=['GET'])
def get_history():
    history = list(collection.find({},{'_id':0}).sort("_id",-1))
    return jsonify(history)

if _name_ == '_main_':
    app.run(host='0.0.0.0', port=5000)
```

## Load Testing menggunakan Locust

Pertama pastikan locust sudah terinstall menggunakan
```
pip install locust
```
Lalu tambahkan Locustfile berikut
```
from locust import HttpUser, task, between

class SentimentAnalysisUser(HttpUser):
    wait_time = between(1, 5)  # Wait time between task executions

    @task(1)
    def analyze_sentiment(self):
        self.client.post("/analyze", json={"text": "This is a test text for sentiment analysis."})

    @task(2)
    def get_history(self):
        self.client.get("/history")

    def on_start(self):
        """Called when a simulated user starts running. Useful for setup tasks."""
        self.client.post("/analyze", json={"text": "Initial setup text."})

# To run this Locust file, save it as locustfile.py and run the following command in your terminal:
# locust -f locustfile.py --host http://139.59.228.185:5000

```
Lalu dapatkan analisisnya
```
sudo systemctl status sentiment-analysis
```
![WhatsApp Image 2024-06-28 at 20 01 33_2f39c8d8](https://github.com/Aceeen/FP-TKA/assets/151147728/c89c6aae-c0fb-4293-b887-c6f00fab8158)

## KESIMPULAN DAN SARAN
Masih perlu dilakukan konfigurasi kembali untuk optimalisasi sistem dan menurunkan waktu response time.


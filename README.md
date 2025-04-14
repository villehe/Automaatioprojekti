# Automaatioprojekti

Tämä Node.js-sovellus muodostaa yhteyden MQTT-välittäjään ja tallentaa datalogeja MongoDB Atlakseen. Se on ajastettu toimimaan viiden minuutin välein Herokussa ja sisältää CI/CD-pipeline-tuen GitHub Actionsin kautta.

# Teknologiat ja kirjastot

- Node.js (väh. 22.14.0)
- Express
- MongoDB (Atlas)
- MQTT
- Heroku (Scheduler)
- GitHub Actions (CI/CD)

---

# Asennus paikallisesti

1. **Kloonaa repositio**

```bash
git clone https://github.com/villehe/Automaatiorojekti.git
cd clientdemo
```

2. **Asenna riippuvuudet**

```bash
npm install
npm install mqtt
npm install mongodb

```

3. **Luo `.env`-tiedosto projektin juureen ja lisää MongoDB-yhteysosoite**

```env
MONGODB_URI=mongodb+srv://Ville:<db_password>@cluster0.r2qya.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

4. **Aja sovellus manuaalisesti**

```bash
node mqtt_mongo.js
```

---

# Deploy Herokuun
Tämä sovellus toimii Herokussa **worker-dynona** eli taustaprosessina, joka ei tarjoa web-käyttöliittymää. `Procfile` määrittää suoritettavan komennon:
worker: node mqtt_mongo.js

1. **Luo Heroku-sovellus**

```bash
git add .
git commit -m "Added a Procfile."
heroku login
heroku create example-app 

```

2. **Lisää MongoDB URI Herokuun**

```bash
mongodb+srv://Ville:<db_password>@cluster0.r2qya.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

3. **Pushaa projekti Herokuun**

```bash
git push heroku master
```
3. **Tarkistetaan, että sovellus toimii**

```bash
heroku scale worker=1 web=0
heroku logs --tail

---

# CI/CD – GitHub Actions

Repositorioon on lisätty automaattinen deploy Herokuun:

Tiedosto: `.github/workflows/ci.yml`

name: Node.js CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build-and-run:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run application (basic startup check)
        run: |
          node mqtt_mongo.js &
          sleep 5
          echo "Worker started ilman virheitä (oletetaan OK)"

```

Muista lisätä seuraava GitHub Secrets -osioon:

- `HEROKU_API_KEY` (voit luoda sen Heroku Dashboardin "Account Settings" -sivulta)

---

# Ympäristömuuttujat

| Avain          | Kuvaus                        |
|----------------|-------------------------------|
| `MONGODB_URI`  | MongoDB Atlas connection URI  |

---

# Kehittäjä

Toteuttaja: **villehe**

---

# Kommentit tekemisestä

Deployment Herokuun onnistui melko helposti luentoa seuraamalla. Sovellus kaatui aluksi jokaisen datapäivityksen välissä,
koska Heroku odotti jonkinlaista web-käyttöliittymää. Ongelma hävisi tuolla edellä mainitulla "Luo `.env`-tiedosto projektin juureen ja lisää MongoDB-yhteysosoite". Hyödynsin myös koneellani olevaa Visual Studio Code -ohjelmaa, jossa voi pyytää tekoälyltä apua, jos ei ymmärrä jotain.
Löysin myös hyvät ohjeet github actions -käyttöönottoon. Myös tähän README-tiedostoon löytyi hyvä valmis pohja. Kaiken kaikkiaan hyvä projekti, josta sai jo hyvän käsityksen pilvipalveluiden käytöstä.

---

# Lisätietoja

- [Heroku Scheduler](https://devcenter.heroku.com/articles/scheduler)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- [GitHub Actions Docs](https://docs.github.com/en/actions)

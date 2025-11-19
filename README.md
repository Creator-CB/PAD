# PAD
PAD-UTM-University-MD / Laborator 2 – 2025
Web Proxy: Realizarea Transparentei în Distribuirea Datelor


  1.Descriere generală:
Acest proiect implementează un Web Proxy inteligent, complet funcțional, care stă între client și multiple servere Data-Warehouse (DW) și oferă:
Load balancing (round-robin)
Caching inteligent pentru GET
Proxy transparent (clientul vede un singur endpoint)
Procesare concurentă (workers / threads)
DW servere cu suport JSON și XML
Interfață unificată pentru acces la date
Proiectul poate fi rulat pe orice sistem Linux/Windows și respectă 100% cerințele laboratorului de la Univer.

  2.Structura proiectului:
  lab2-web-proxy/
│
├── README.md
│
├── src/
│   ├── dw_server.py
│   ├── proxy_server.py
│   └── requirements.txt
│
└── docs/
    ├── arhitectura.png
    ├── flux_proxy.png
    ├── cache_diagram.png
    ├── load_balancer.png
    └── concluzie.png

<img width="492" height="56" alt="Screenshot 2025-11-19 at 14 45 13" src="https://github.com/user-attachments/assets/d573a4ef-0ba7-4177-909f-d989052f9185" />

  
  3.Arhitectura sistemului:

  Sistemul este format din:
Clientul → trimite cereri HTTP (curl, browser, Postman etc.)
Reverse Proxy → distribuie cereri, aplică caching, ascunde infrastructura
Serverele DW (Data Warehouse) → stochează date semi-structurate (employees)
Proxy-ul primește toate cererile și trimite request-urile mai departe către unul din serverele DW folosind round-robin.

  4.Instalare


1. Creează mediu de lucru
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
2. Clonează repo-ul tău GitHub
git clone https://github.com/<user>/lab2-web-proxy.git
cd lab2-web-proxy/src
3. Creează mediul virtual
python3 -m venv venv
source venv/bin/activate
4. Instalează dependențele
pip install -r requirements.txt


Rulare servere DW
DW 1 (port 8001)
export DW_NAME="DW-1"
uvicorn dw_server:app --host 0.0.0.0 --port 8001
<img width="817" height="184" alt="Screenshot 2025-11-19 at 14 51 33" src="https://github.com/user-attachments/assets/6109c6ea-c6c7-4794-9a34-8d03f30f68c9" />

DW 2 (port 8002)
export DW_NAME="DW-2"
uvicorn dw_server:app --host 0.0.0.0 --port 8002
<img width="805" height="108" alt="Screenshot 2025-11-19 at 14 52 01" src="https://github.com/user-attachments/assets/25803004-1bdd-4c39-b3b7-8e5805cdf8ad" />



Rulare Reverse Proxy
cd src
uvicorn proxy_server:app --host 0.0.0.0 --port 9000 --workers 4

  <img width="895" height="302" alt="Screenshot 2025-11-19 at 14 52 31" src="https://github.com/user-attachments/assets/6b12c4c3-f388-4f01-9932-e98991ff1272" />

  
  5.Testare funcționalități

  1. POST – Creare employee
curl -X POST http://localhost:9000/employees \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "role": "Dev", "salary": 15000}'
Output DW:
[DW-1] POST /employees -> created id=1
<img width="575" height="43" alt="Screenshot 2025-11-19 at 14 54 41" src="https://github.com/user-attachments/assets/257669c4-c0d5-4327-8da5-0a35f704bc55" />



3. GET JSON – Lista employees
curl "http://localhost:9000/employees?offset=0&limit=10&format=json"
<img width="698" height="46" alt="Screenshot 2025-11-19 at 14 55 43" src="https://github.com/user-attachments/assets/33f2a32e-62bd-4f62-afbc-a63c629c724a" />
<img width="775" height="230" alt="Screenshot 2025-11-19 at 14 56 02" src="https://github.com/user-attachments/assets/32c096ac-cf03-41de-95a0-36dafc551517" />



5. GET XML – Un employee în format XML
curl "http://localhost:9000/employees/1?format=xml"
<img width="615" height="48" alt="Screenshot 2025-11-19 at 14 56 29" src="https://github.com/user-attachments/assets/6e3e5f0f-29ba-4ba5-867f-d646657cd136" />
<img width="583" height="58" alt="Screenshot 2025-11-19 at 14 56 48" src="https://github.com/user-attachments/assets/bbcf097f-9a19-4655-b2bd-526b8ef1c456" />





  

7. Load Balancing demonstrat
Trimite 4 cereri:
curl http://localhost:9000/employees
curl http://localhost:9000/employees
curl http://localhost:9000/employees
curl http://localhost:9000/employees
Log:
[PROXY] Forward -> DW-1
[PROXY] Forward -> DW-2
[PROXY] Forward -> DW-1
[PROXY] Forward -> DW-2

<img width="531" height="144" alt="Screenshot 2025-11-19 at 14 57 24" src="https://github.com/user-attachments/assets/9068dc79-116d-479b-8a47-2404fb0edf1d" />
<img width="529" height="36" alt="Screenshot 2025-11-19 at 14 58 24" src="https://github.com/user-attachments/assets/3e9afbca-3c73-49e0-af57-bada8c56e8cb" />
<img width="515" height="48" alt="Screenshot 2025-11-19 at 14 58 33" src="https://github.com/user-attachments/assets/040f24e2-edf0-4c9c-905a-f22dcf53cf00" />




8. Cache demonstrat
Primul GET:
[PROXY] Forward GET -> DW-1
Al doilea GET:
[PROXY] Cache HIT
<img width="576" height="121" alt="Screenshot 2025-11-19 at 14 59 50" src="https://github.com/user-attachments/assets/765f3118-f70d-4cbc-b8e3-3163fbd6f9bb" />



9. Concurență (thread-per-request)
sudo apt install apache2-utils
ab -n 50 -c 10 http://localhost:9000/employees

<img width="1066" height="715" alt="Screenshot 2025-11-19 at 15 00 33" src="https://github.com/user-attachments/assets/5703fb3a-fe1e-41e5-8768-8e8c3d27309f" />





!!  Explicații tehnice importante

DDOS protection basic (prin caching)
✔ request mapping către controllere
✔ thread-safe operations
✔ validare format (JSON/XML)
✔ invalidare cache la POST/PUT/DELETE

Unde se folosește acest model în industrie?
Acest laborator reproduce arhitecturi reale folosite în:
1. NGINX Reverse Proxy
Folosite de Google, Netflix, Meta, Amazon.
🟦 2. API Gateway (microservicii)
Kong, AWS API Gateway, Azure APIM funcționează similar:
expun un endpoint
fac routing & caching & balancing
🟦 3. Big Data / Data Warehouse Clusters
Google BigQuery, AWS Redshift, Snowflake folosesc un „proxy” pentru acces la cluster.
🟦 4. Sisteme Enterprise
Orice aplicație cu microservicii → are un gateway exact ca acest  Proxy.

  Concluzie finală
Am implementat o arhitectură Web Proxy complet funcțională, conform cerințelor laboratorului.
Proxy-ul oferă transparență, ascunzând distribuirea datelor între DW-uri.
Load-balancing-ul distribuiă cererile în mod egal.
Caching-ul optimizează accesul și reduce încărcarea backend-urilor.
Serverele DW oferă date în JSON și XML, fiind ușor de integrat.
Modelul reflectă foarte bine arhitecturi enterprise, moderne și robuste.



